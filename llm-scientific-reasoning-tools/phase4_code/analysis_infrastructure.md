# Infrastructure Analysis: Tools for LLM-Scientific Reasoning Pipelines

**Date**: 2026-03-10
**Scope**: Deep code analysis of five repositories that form the infrastructure backbone for integrating LLMs with scientific document processing, symbolic mathematics, computational environments, and knowledge graphs.

---

## Table of Contents

1. [Nougat: PDF-to-LaTeX OCR](#1-nougat-pdf-to-latex-ocr)
2. [Jupyter AI: LLM Extension for JupyterLab](#2-jupyter-ai-llm-extension-for-jupyterlab)
3. [SymPy LaTeX Parser: Symbolic Math from LaTeX](#3-sympy-latex-parser-symbolic-math-from-latex)
4. [MATLAB MCP Core Server: AI Agent Control of MATLAB](#4-matlab-mcp-core-server-ai-agent-control-of-matlab)
5. [InfraNodus VSCode Extension: Knowledge Graph Construction](#5-infranodus-vscode-extension-knowledge-graph-construction)
6. [Cross-Repository Integration Patterns](#6-cross-repository-integration-patterns)
7. [Gaps and Opportunities](#7-gaps-and-opportunities)

---

## 1. Nougat: PDF-to-LaTeX OCR

**Repository**: https://github.com/facebookresearch/nougat
**Paper**: arXiv:2308.13418 (Blecher et al., 2023)
**License**: CC-BY-NC-4.0

### 1.1 Architecture

Nougat is a Visual Transformer model built on the Donut (Document Understanding Transformer) architecture. It takes rasterized PDF pages as input and autoregressively generates Mathpix Markdown (.mmd) output containing LaTeX equations, tables, and structured text.

#### Encoder: Swin Transformer

| Parameter | Value |
|-----------|-------|
| Base architecture | Swin Transformer (hierarchical) |
| Input resolution | 896 x 672 pixels |
| Rasterization DPI | 96 |
| Patch size | 4 pixels |
| Window size | 7 |
| Layer depths | [2, 2, 14, 2] across 4 stages |
| Embedding dimension | 128 |
| Attention heads per stage | [4, 8, 16, 32] |
| Pretrained from | `swin_base_patch4_window12_384` |

The encoder preprocesses images by:
1. Cropping whitespace margins via OpenCV binary thresholding and contour detection
2. Optional rotation to align the longer axis with the canvas (`align_long_axis=True`)
3. Thumbnail resizing with centered or randomized padding
4. Positional bias interpolation for non-standard window sizes

#### Decoder: mBART

| Parameter | Value |
|-----------|-------|
| Base architecture | mBART (Multilingual BART) |
| Decoder layers | 10 (base) / 4 (small) |
| Hidden dimension | 1024 |
| Max position embeddings | 4096 tokens (base) / 3584 (small) |
| Cross-attention | Enabled (encoder-decoder) |
| Pretrained from | `facebook/mbart-large-50` |
| Special tokens | `<s>`, `</s>`, `<pad>`, `<unk>` + custom additions |

The decoder dynamically resizes position embeddings via interpolation when the sequence length differs from the pretrained 1024 tokens.

#### Inference Control

- **StoppingCriteriaScores**: Monitors logit variance across a 200-token sliding window. Generation halts when variance drops below 0.015, indicating a repetitive loop.
- **RunningVarTorch**: Efficient rolling variance computation for real-time loop detection.

### 1.2 Model Sizes

| Variant | Parameters | Decoder Layers | Max Tokens |
|---------|-----------|----------------|------------|
| nougat-base (0.1.0-base) | 350M | 10 | 4096 |
| nougat-small (0.1.0-small) | 250M | 4 | 3584 |

### 1.3 Training Data

- **Total dataset**: 8,204,754 pages from three sources:
  - arXiv: 7,511,745 pages (91.5%)
  - PubMed Central: 536,319 pages
  - Industry Documents Library: 446,777 pages
- **Acceptance rate**: ~47% of pages passed quality thresholds
- **Training config**: AdamW optimizer, initial LR 5e-5 reduced to 7.5e-6, batch size 192, 3 epochs, LR decay factor 0.9996 every 15 updates

### 1.4 Performance Metrics

#### Overall (Nougat Base, 350M)

| Metric | Value |
|--------|-------|
| Edit distance (all) | 0.071 |
| BLEU (all) | 89.1 |
| METEOR (all) | 93.0 |
| Precision | 93.5% |
| Recall | 92.8% |
| F1 | 93.1% |

#### By Modality

| Modality | Edit Distance | BLEU | F1 |
|----------|---------------|------|-----|
| Plain text | 0.058 | 91.2 | 95.7% |
| **Math/Equations** | **0.128** | **56.9** | **76.5%** |
| Tables | 0.211 | 69.7 | 78.0% |

#### Small Model (250M)

| Metric | Value |
|--------|-------|
| Overall F1 | 92.9% |
| Math F1 | 76.9% |
| Plain text F1 | 95.7% |

#### Baseline Comparisons

| System | Overall F1 | Math F1 | Edit Distance |
|--------|-----------|---------|---------------|
| **Nougat Base** | **93.1%** | **76.5%** | **0.071** |
| GROBID | 73.0% | 9.7% | 0.312 |
| PDF embedded text | 79.2% | N/A | 0.255 |

**Key finding**: Nougat achieves 76.5% F1 on mathematical equations compared to GROBID's 9.7%. This is the primary reason Nougat is significant for scientific reasoning pipelines -- it can extract equations that are otherwise completely inaccessible from PDFs.

### 1.5 Inference Speed

- **Mean generation time**: 19.5 seconds per batch (6 pages in parallel)
- **GPU**: NVIDIA A10G (24GB VRAM)
- **Average tokens per page**: ~1,400
- **Per-page time**: ~3.25 seconds
- **Comparison**: GROBID processes 10.6 PDFs/second (much faster but vastly lower quality on math)

### 1.6 Post-Processing Pipeline

Nougat includes a sophisticated post-processing system (`postprocessing.py`):

1. **Equation cleanup**: Converts equation tags from `(1.2) \[...\]` to `\[... \tag{1.2}\]`; normalizes `\bm{}` to `\mathbf{}`; fixes `\mbox{\boldmath$...$}` patterns
2. **Repetition detection**: Finds the longest repeating substring at end of output and truncates to single occurrence (min segment length: 30 chars)
3. **Hallucination removal**: Removes section titles >100 characters, orphaned numeric headers, incomplete footnote markup
4. **Table validation**: Rejects tables with >15 `\begin{tabular}`, >60 `\multicolumn`, or >400 `&` (likely hallucinated)
5. **Reference deduplication**: Fuzzy matching (similarity >0.9) to detect duplicate/hallucinated reference sections
6. **Markdown normalization**: URL formatting, escaped period fixes, list restructuring

### 1.7 Usage API

**CLI**:
```bash
pip install nougat-ocr
nougat path/to/paper.pdf -o output_directory
# Output: output_directory/paper.mmd
```

**REST API**:
```bash
pip install "nougat-ocr[api]"
nougat_api
# POST http://127.0.0.1:8503/predict/ with start/stop page params
```

**Python (via HuggingFace Transformers)**:
```python
from transformers import NougatProcessor, VisionEncoderDecoderModel
processor = NougatProcessor.from_pretrained("facebook/nougat-base")
model = VisionEncoderDecoderModel.from_pretrained("facebook/nougat-base")
```

### 1.8 Limitations

- English scientific documents only (Chinese, Russian, Japanese not supported)
- 1.5% of test pages show repetitive generation loops
- 32% higher failure rate on out-of-domain documents
- Math F1 of 76.5% means ~1 in 4 equations has errors
- CPU inference triggers false positives in failure detection (use `--no-skipping`)
- Maximum 4096 tokens per page limits very dense pages

---

## 2. Jupyter AI: LLM Extension for JupyterLab

**Repository**: https://github.com/jupyterlab/jupyter-ai
**Documentation**: https://jupyter-ai.readthedocs.io
**Status**: Incubation project under JupyterLab organization

### 2.1 Architecture

Jupyter AI provides two primary interfaces for LLM integration into computational notebooks:

1. **`%%ai` Magic Command**: A cell magic that works across any IPython kernel (JupyterLab, Notebook, Colab, Kaggle, VSCode)
2. **JupyterLab Chat Extension**: A native sidebar UI for conversational AI within JupyterLab 4+

The system is split into two packages:
- `jupyter-ai`: Full package with JupyterLab extension + magics
- `jupyter-ai-magics`: Lightweight magic-only package for non-JupyterLab environments

**Provider Architecture**: Uses LangChain as the abstraction layer, with LiteLLM providing a unified interface to hundreds of models. Providers are modular -- installed as separate `langchain-*` packages.

### 2.2 The `%%ai` Magic Command

#### Syntax
```python
%load_ext jupyter_ai_magic_commands

%%ai <provider>/<model-id> [-f format]
<prompt text starting on second line>
```

#### Output Formats (`-f` flag)

| Format | Description | Use Case |
|--------|-------------|----------|
| `markdown` | Default rendering | General text responses |
| `code` | Executable Python cell | Code generation |
| `math` | IPython Math (LaTeX) | Equation rendering |
| `html` | HTML/SVG rendering | Diagrams, formatted content |
| `json` | Structured data | API-like responses |
| `text` | Plain text | Raw output |
| `image` | Image rendering | Text-to-image models |

#### Variable Interpolation

The magic command supports injecting notebook context:
- `In[n]` -- references code from cell n
- `Out[n]` -- references output from cell n
- `Err[n]` -- references error messages from cell n

This enables prompts like: "Explain the error in `{Err[5]}` and fix the code in `{In[5]}`"

#### Configuration

```python
# Set default model
%config AiMagics.initial_language_model = "anthropic:claude-v1.2"

# Set chat history depth (default: 2 exchanges)
%config AiMagics.max_history = 4

# Create aliases for frequently-used models
%ai alias claude anthropic:claude-v1.2
%%ai claude
What is the Fourier transform of a Gaussian?
```

### 2.3 Supported LLM Providers

| Provider | Package | Example Model ID |
|----------|---------|------------------|
| Anthropic | `langchain-anthropic` | `anthropic:claude-v1.2` |
| OpenAI | `langchain-openai` | `openai:gpt-4` |
| AWS Bedrock | `langchain-aws` | `bedrock/anthropic.claude-3-5-haiku-*` |
| Google Gemini | `langchain-google-genai` | `google:gemini-pro` |
| Hugging Face | `langchain-huggingface` | `huggingface_hub:*` |
| Cohere | `langchain-cohere` | `cohere:command` |
| MistralAI | `langchain-mistralai` | `mistral:*` |
| NVIDIA NGC | `langchain-nvidia-ai-endpoints` | `nvidia:*` |
| AWS SageMaker | (via boto3) | `sagemaker:*` |
| Ollama (local) | `langchain-ollama` | `ollama:llama2` |
| vLLM (local) | (built-in) | `vllm:*` |
| GPT4All (local) | (built-in) | `gpt4all:*` |
| OpenRouter | `langchain-openrouter` | `openrouter:*` |

### 2.4 Chat Interface Features

- **Persistent chat**: All conversations stored as `.chat` files (Jupyter Server documents)
- **Context memory**: Remembers last 2 exchanges (configurable via `max_history`)
- **File attachment**: `@file` mention for document-specific queries
- **Selection inclusion**: `Include selection` checkbox to send highlighted code
- **Code cell error fixing**: Drag-and-drop error cells into chat
- **Notebook generation**: `/generate` command creates complete notebooks from prompts

### 2.5 How It Handles Code + Equations

For scientific reasoning, the key workflow is:

1. **Equation rendering**: `%%ai model -f math` renders LLM output as typeset LaTeX equations in the notebook
2. **Code generation**: `%%ai model -f code` produces executable Python that can be run in subsequent cells
3. **Context chaining**: Use `In[n]`/`Out[n]` interpolation to feed computation results back into LLM prompts
4. **Error remediation**: Chat interface can diagnose and fix errors in scientific code

**Limitation**: The system treats equations as display output (`-f math`) -- there is no built-in mechanism to parse LLM-generated LaTeX into executable SymPy code. This is a gap that a scientific reasoning pipeline would need to bridge.

### 2.6 Extensibility

**Custom providers**: Install any `langchain-*` package that implements the LangChain LLM/ChatModel interface.

**Server-level configuration**:
```bash
jupyter lab \
  --AiExtension.initial_language_model="anthropic:claude-3-opus" \
  --AiExtension.blocked_providers='["openai"]' \
  --AiExtension.allowed_providers='["anthropic", "ollama"]' \
  --AiExtension.model_parameters='{"anthropic:*": {"max_tokens": 4096}}'
```

**Personas API** (v3+): The `jupyter_ai.personas` entry point group allows adding custom AI personas beyond the default Jupyternaut. Custom personas can implement specialized message processing logic while accessing the configured chat model.

**Persistent configuration**: Aliases and settings can be set in `ipython_config.py`:
```python
c.AiMagics.aliases = {"sci": "anthropic:claude-3-opus"}
```

### 2.7 System Requirements

- Python 3.9-3.12
- JupyterLab 4+ or Jupyter Notebook 7.5+
- At least one model provider configured with API key
- Credentials via `getpass` (secure) or `%env` (carries leak risk)

---

## 3. SymPy LaTeX Parser: Symbolic Math from LaTeX

**Repository**: https://github.com/sympy/sympy
**Module**: `sympy/parsing/latex/`
**Key issue**: #26128 (LLM-based LaTeX parsing)

### 3.1 The `parse_latex()` API

```python
from sympy.parsing.latex import parse_latex

# Basic usage
expr = parse_latex(r"\frac{d}{dx} x^2")
# Returns: Derivative(x**2, x)

# With Lark backend
expr = parse_latex(r"\int_0^1 x^2 dx", backend="lark")

# Strict mode (ANTLR only)
expr = parse_latex(r"\frac{1}{2}", strict=True, backend="antlr")
```

**Function signature**: `parse_latex(s: str, strict: bool = False, backend: str = "antlr") -> sympy.Expr`

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `s` | str | required | LaTeX string (raw strings recommended) |
| `backend` | str | `"antlr"` | Parser backend: `"antlr"` or `"lark"` |
| `strict` | bool | `False` | ANTLR-only: raise on invalid LaTeX vs. attempt recovery |

### 3.2 Two Parser Backends

#### ANTLR Backend (Default)

- Module: `sympy.parsing.latex._parse_latex_antlr`
- Uses ANTLR4 grammar for parsing
- Supports `strict` mode (raises exceptions on invalid LaTeX)
- Graceful error recovery when `strict=False`
- More mature, longer history in SymPy

#### Lark Backend (Newer)

- Module: `sympy.parsing.latex.lark`
- Uses Lark parsing library with Earley parser (handles ambiguous grammars)
- Grammar defined in `grammar/latex.lark`
- Two-stage pipeline: `LarkLaTeXParser` (parse) -> `TransformToSymPyExpr` (transform)
- Explicitly handles ambiguity (`ambiguity="explicit"`)
- Retains all tokens (`keep_all_tokens=True`)
- Experimental -- API may change
- No `strict` mode

**Lark Parser Configuration**:

| Setting | Value |
|---------|-------|
| Parser algorithm | Earley |
| Root rule | `latex_string` |
| Lexer | Auto-selected |
| Ambiguity | Explicit (preserves all parses) |
| Token retention | All tokens kept |

### 3.3 Supported LaTeX Constructs

Based on exhaustive analysis of the Lark grammar (`latex.lark`) and the `TransformToSymPyExpr` transformer:

#### Arithmetic and Relations

| Category | Supported Syntax |
|----------|-----------------|
| Addition/Subtraction | `+`, `-` (binary and unary) |
| Multiplication | `*`, `\times`, `\cdot`, implicit |
| Division | `/`, `\div`, `\frac{}{}`, `\dfrac{}{}` |
| Relations | `=`, `\neq`, `<`, `>`, `\leq`, `\geq` |
| Superscript/Subscript | `^`, `_` |
| Factorial | `!` |
| Absolute value | `|...|` |

#### Functions

| Category | Functions |
|----------|-----------|
| Trigonometric | `\sin`, `\cos`, `\tan`, `\csc`, `\sec`, `\cot` (with power variants) |
| Inverse trig | `\arcsin`, `\arccos`, `\arctan`, `\arccsc`, `\arcsec`, `\arccot` |
| Hyperbolic | `\sinh`, `\cosh`, `\tanh`, `\arsinh`, `\arcosh`, `\artanh` |
| Logarithmic | `\log`, `\ln`, `\lg` (with base via subscript), `\exp` |
| Special | `\sqrt`, `\det`, `\trace`, `\adjugate`, `\min`, `\max` |
| Grouping | `()`, `[]`, `\{...\}`, `\left...\right` variants |
| Floor/Ceiling | `\lfloor...\rfloor`, `\lceil...\rceil` |

#### Calculus

| Construct | LaTeX Syntax | SymPy Result |
|-----------|-------------|--------------|
| Integrals | `\int_a^b f(x) dx` | `Integral(f(x), (x, a, b))` |
| Derivatives | `\frac{d}{dx} f(x)` | `Derivative(f(x), x)` |
| Limits | `\lim_{x \to a}` | `Limit(expr, x, a)` |
| Summation | `\sum_{i=0}^{n}` | `Sum(expr, (i, 0, n))` |
| Products | `\prod_{i=1}^{n}` | `Product(expr, (i, 1, n))` |

#### Linear Algebra

| Construct | Environments |
|-----------|-------------|
| Matrices | `matrix`, `pmatrix`, `bmatrix`, `vmatrix`, `array` |
| Operations | Determinant, trace, adjugate, transpose, conjugate |
| Matrix arithmetic | `MatAdd`, `MatMul` (distinguished from scalar ops) |

#### Quantum Mechanics

| Construct | LaTeX | SymPy |
|-----------|-------|-------|
| Ket | `\langle \psi \rangle` | `Ket('psi')` |
| Bra | `\langle \phi |` | `Bra('phi')` |
| Inner product | `\langle \phi | \psi \rangle` | `InnerProduct(Bra('phi'), Ket('psi'))` |
| Outer product | `| \psi \rangle \langle \phi |` | `OuterProduct(Ket('psi'), Bra('phi'))` |

#### Other

- Binomial coefficients (`\binom{n}{k}`)
- Greek letters (with `\var` prefix handling)
- Multi-letter symbols with primes
- Imaginary unit `i`
- Infinity `\infty`
- Degree-to-radian conversion

### 3.4 Pre-Parse Validation

Before dispatching to either backend, `parse_latex()` runs:

1. **`check_matrix_delimiters()`**: Detects mismatched, excess, or missing matrix delimiters with specific index reporting and contextual snippets
2. **`check_cases_env()`**: Rejects `\begin{cases}...\end{cases}` environments (not yet supported) with a descriptive error

Both raise `LaTeXParsingError` with detailed diagnostic messages.

### 3.5 Known Limitations

1. **`\begin{cases}` environment**: Not supported, raises `LaTeXParsingError`
2. **Matrix division**: Cannot divide by matrices (ambiguity: left vs. right multiplication by inverse)
3. **Unconstrained summations**: `\sum` without explicit bounds not supported
4. **Partial integral bounds**: Must provide both bounds or neither
5. **Prime notation on non-matrices**: Triggers parsing errors
6. **Determinant/trace on scalars**: Requires matrix input
7. **Nested environments**: Complex nesting can produce ambiguous parses
8. **Custom macros**: `\newcommand` and user-defined macros not supported
9. **Text-mode commands**: `\text{}`, `\mathrm{}` have limited support

### 3.6 Issue #26128: LLM-Based LaTeX Parsing

**Status**: Open (as of Nov 2025, 19 comments)

**Core Proposal**: Use LLMs to parse LaTeX into SymPy expressions, motivated by the observation that "an LLM is going to be much better at parsing LaTeX because it's seen a ton of LaTeX in its training data" (Aaron Meurer, SymPy maintainer).

#### Key Discussion Points

1. **Local vs. Cloud LLMs**: Community vote was ~9:3 in favor of local LLMs, though maintainer sylee957 advocated cloud-based for the exploration stage

2. **Dataset Problem**: "We don't have a good dataset to train on" -- synthetic data from SymPy's own `latex()` function would be incomplete; arXiv extraction requires manual validation

3. **TeX Engine Integration**: Proposed using LuaTeX's Lua hooks to access intermediate representations, normalizing diverse LaTeX inputs before LLM processing

4. **Comparison of 18+ existing parsers**: The discussion cataloged TeX Live, MiKTeX, XeTeX, LuaTeX, MathJax, KaTeX, Pandoc, LaTeXML, pylatexenc, latex2sympy, and more

5. **Complementary vision**: Aaron Meurer proposed a standalone "LaTeX math-mode parser library for Python" that parses to AST/CST -- useful for formatting, linting, and canonicalization independent of LLMs

6. **Reliability concern**: LLMs may produce plausible but mathematically incorrect symbolic expressions. Evaluation methodology is unresolved.

#### Implications for Scientific Reasoning

This issue reveals a fundamental tension: grammar-based parsers (ANTLR/Lark) are deterministic but brittle on non-standard LaTeX, while LLMs are flexible but non-deterministic. A hybrid approach -- using LLMs to normalize LaTeX before deterministic parsing -- may be the most promising path.

---

## 4. MATLAB MCP Core Server: AI Agent Control of MATLAB

**Repository**: https://github.com/matlab/matlab-mcp-core-server
**Language**: Go
**Architecture**: Model Context Protocol (MCP) server with stdio transport
**Requirement**: MATLAB R2020b or later

### 4.1 Architecture

The server is implemented in Go with a clean hexagonal architecture:

```
cmd/matlab-mcp-core-server/main.go      -- Entry point
internal/adaptors/mcp/tools/             -- MCP tool definitions
internal/adaptors/application/           -- Application-level orchestration
internal/adaptors/sdk/                   -- SDK abstraction layer
internal/usecases/                       -- Business logic
```

Two operational modes:
- **Single-session**: One MATLAB instance, tools operate on it directly
- **Multi-session**: Multiple MATLAB instances, managed via session IDs

Communication is via stdio (standard input/output), following the MCP specification.

### 4.2 Exposed MCP Tools

#### Single-Session Tools (5 tools)

**1. `detect_matlab_toolboxes`**
- Input: None
- Output: MATLAB version + installed toolboxes with versions
- Purpose: Let the AI agent understand what MATLAB capabilities are available
- Annotation: Read-only, no side effects

**2. `check_matlab_code`**
- Input: `script_path` (string, required) -- absolute path to .m file
- Output: Array of `code_issues` objects with `description`, `line`, `start_column`, `end_column`, `severity`, `fixable`
- Purpose: Static analysis (linting) without execution
- Detects: Coding style issues, potential errors, deprecated functions, performance concerns, best practice violations
- Annotation: Read-only, no side effects

**3. `evaluate_matlab_code`**
- Input: `code` (string, required), `project_path` (string, optional)
- Output: Command window output from execution
- Purpose: Execute arbitrary MATLAB code strings
- Side effects: Can modify MATLAB workspace, create files, etc.

**4. `run_matlab_file`**
- Input: `script_path` (string, required) -- absolute path to .m file
- Output: Command window output or success confirmation
- Purpose: Execute a MATLAB script file, auto-sets working directory to script location
- Side effects: Full execution with all MATLAB capabilities

**5. `run_matlab_test_file`**
- Input: `script_path` (string, required) -- absolute path to test .m file
- Output: Comprehensive test results following MATLAB testing framework
- Purpose: Run MATLAB unit tests with full result reporting

#### Multi-Session Tools (4 additional tools)

**6. `start_matlab_session`**
- Input: `matlab_root` (string) -- MATLAB installation path
- Output: `session_id` (integer), `ver_output`, `add_ons_output`
- Purpose: Start a new MATLAB instance and return its session ID

**7. `stop_matlab_session`**
- Input: `session_id` (integer)
- Output: Confirmation message
- Purpose: Terminate a specific MATLAB session

**8. `list_available_matlabs`**
- Input: None
- Output: Array of `{version, matlabRoot}` objects
- Purpose: Discover all MATLAB installations on the host

**9. `evaluate_matlab_code` (multi-session variant)**
- Input: `code` (string), `session_id` (integer), `project_path` (optional)
- Output: Command window output
- Purpose: Execute code in a specific session

### 4.3 Resources (Documentation for AI Agents)

The server exposes two markdown resources that guide AI code generation:

1. **`matlab_coding_guidelines`**: Naming conventions, formatting rules, performance optimization patterns
2. **`plain_text_live_code_guidelines`**: Rules for plain text live scripts (MATLAB R2025a+)

### 4.4 Configuration

| Argument | Purpose |
|----------|---------|
| `--matlab-root` | MATLAB installation path |
| `--initialize-matlab-on-startup` | Start MATLAB immediately (vs. lazy init) |
| `--initial-working-folder` | Default working directory |
| `--matlab-display-mode` | `desktop` (show GUI) or `nodesktop` (headless) |
| `--disable-telemetry` | Opt out of anonymized usage data |

**Integration with AI clients**:
- **Claude Code**: `claude mcp add matlab-mcp-core-server`
- **Claude Desktop**: `.mcpb` bundle installation
- **VS Code**: Workspace-level `mcp.json` configuration

### 4.5 Implications for Scientific Reasoning

The MATLAB MCP server enables AI agents to:
1. **Execute numerical computations**: Solve ODEs, run simulations, perform matrix operations
2. **Validate results**: Run MATLAB unit tests on generated solutions
3. **Static analysis**: Check generated MATLAB code for errors before execution
4. **Multi-version testing**: Start sessions with different MATLAB versions
5. **Toolbox discovery**: Determine if Symbolic Math Toolbox, Optimization Toolbox, etc. are available

**Key gap**: No tool for reading MATLAB workspace variables back into the AI context. The agent sees only command window text output, not structured data.

---

## 5. InfraNodus VSCode Extension: Knowledge Graph Construction

**Repository**: https://github.com/infranodus/infranodus-vscode-extension
**Status**: Alpha
**Architecture**: VSCode extension + InfraNodus cloud API

### 5.1 Knowledge Graph Construction

InfraNodus builds semantic networks from text by:
1. **Topic extraction**: AI-powered identification of primary concepts and themes
2. **Relational mapping**: Determining how concepts interconnect via co-occurrence and semantic similarity
3. **Network generation**: Creating nodes (concepts) and edges (relationships) in a graph structure

The extension operates on:
- Individual files
- Entire directories
- Obsidian vault structures

### 5.2 Visualization Capabilities

The graph renders in VSCode's Secondary Sidebar as an interactive network with:
- **Concept nodes**: Sized by frequency/importance
- **Relationship edges**: Weighted by co-occurrence strength
- **Search highlighting**: Query terms highlight relevant subgraph
- **Gap analysis**: Identifies disconnected or underexplored concept areas
- **Differential analysis**: Compares content across files/folders to show semantic variations

### 5.3 LLM Integration

InfraNodus bridges knowledge graphs and LLMs through:
1. **AI button**: Generates prompts containing relevant graph segments for external AI tools
2. **Clipboard export**: Selected graph data formatted for AI copilot consumption
3. **Compressed graph logging**: "InfraNodus Log" formats graph data for efficient LLM context injection
4. **Compatible with**: Cursor AI, Windsurf AI, GitHub Copilot, and other copilot systems

### 5.4 Key Features

| Feature | Description |
|---------|-------------|
| File/Folder analysis | Works on individual files or entire directories |
| Search interface | Query-based exploration of graph content |
| Gap analysis | Identifies disconnected or underexplored concept areas |
| AI export | Pastes formatted graph data into external AI tools |
| Log viewer | Displays compressed graph representation |
| Obsidian integration | Visualize vault knowledge structure |

### 5.5 Commands

- `InfraNodus Graph: Set API Key` -- Configure API credentials
- `InfraNodus Graph: Open InfraNodus Log` -- View compressed graph data
- `InfraNodus Graph: Paste (Selected) Graph in a File` -- Export graph to file
- Context menu integration for right-click analysis of files/folders

### 5.6 Architecture

```
VSCode Extension (UI Layer)
    |
    v
Secondary Sidebar Webview
    |
    v
InfraNodus API Client
    |
    v
InfraNodus Cloud Service (Graph Generation + AI Topic Modeling)
    |
    v
Output: Interactive Graph + Compressed Log Data
```

### 5.7 Limitations

- **Alpha stage**: Feature incomplete, optimization ongoing
- **Cloud-dependent**: Requires InfraNodus account + API key (no offline mode)
- **No local processing**: All graph construction happens server-side
- **Text-optimized**: Code-specific analysis not detailed
- **No structured output**: Graph data is exported as formatted text, not as a queryable graph database

### 5.8 Implications for Scientific Reasoning

InfraNodus could serve as a:
1. **Literature mapping tool**: Build concept graphs from extracted paper text
2. **Gap identification**: Find unexplored connections between scientific concepts
3. **Context preparation**: Format knowledge graph summaries as LLM prompts for reasoning tasks

**Key gap**: The cloud-only architecture and lack of structured graph export (e.g., JSON, GraphML) limit integration into automated pipelines.

---

## 6. Cross-Repository Integration Patterns

### 6.1 PDF-to-Symbolic-Math Pipeline

```
Scientific PDF
    |
    v  [Nougat]
LaTeX Markdown (.mmd)
    |
    v  [SymPy parse_latex()]
Symbolic Expression (sympy.Expr)
    |
    v  [Jupyter AI %%ai -f code]
Executable Python/MATLAB Code
    |
    v  [MATLAB MCP / Python execution]
Numerical Results
```

**Bottleneck analysis**:
- Nougat: 76.5% F1 on equations -- ~1 in 4 equations will have errors
- SymPy `parse_latex()`: Brittle on non-standard LaTeX, no `\begin{cases}` support
- Combined error rate is multiplicative: effectively ~60% of equations survive the full pipeline correctly

### 6.2 LLM-Augmented Parsing Pipeline (Proposed)

```
Scientific PDF
    |
    v  [Nougat]
LaTeX Markdown (.mmd) -- may contain errors
    |
    v  [LLM normalization/correction]
Cleaned LaTeX -- standardized notation
    |
    v  [SymPy parse_latex()]
Symbolic Expression
    |
    v  [LLM verification via Jupyter AI]
Verified symbolic result
```

This addresses the SymPy #26128 discussion: use LLMs to normalize LaTeX (fix Nougat errors, standardize notation) before deterministic parsing.

### 6.3 Knowledge-Grounded Reasoning Pipeline

```
Paper corpus
    |
    v  [InfraNodus]
Knowledge graph (concept relationships)
    |
    v  [LLM prompt construction]
Contextual prompt with relevant concepts
    |
    v  [Jupyter AI %%ai]
Reasoning output (text + equations)
    |
    v  [SymPy parse_latex()]
Symbolic verification
```

### 6.4 Computational Validation Pipeline

```
Mathematical derivation (LaTeX)
    |
    v  [SymPy parse_latex()]
Symbolic expression
    |
    v  [SymPy simplify/solve]
Symbolic result
    |
    v  [MATLAB MCP evaluate_matlab_code]
Numerical validation
    |
    v  [MATLAB MCP run_matlab_test_file]
Test verification
```

---

## 7. Gaps and Opportunities

### 7.1 Critical Missing Capabilities

| Gap | Impact | Potential Solution |
|-----|--------|-------------------|
| Nougat equation error rate (23.5%) | Cascading errors in downstream parsing | Fine-tune on domain-specific papers; LLM post-correction |
| SymPy `\begin{cases}` not supported | Cannot parse piecewise functions from papers | Contribute to SymPy Lark backend |
| No LaTeX-to-code direct translation | Extra pipeline step through symbolic intermediate | LLM direct translation with SymPy verification |
| MATLAB MCP lacks workspace read-back | Cannot inspect computed variables | Extend with `get_matlab_variable` tool |
| InfraNodus cloud-only | Cannot integrate into offline HPC pipelines | Build local alternative using NetworkX + sentence-transformers |
| No standard equation provenance tracking | Cannot trace which paper/page an equation came from | Build metadata pipeline linking Nougat page output to SymPy expressions |

### 7.2 Integration Opportunities

1. **Nougat + SymPy + LLM error correction**: Build a pipeline where Nougat extracts LaTeX, an LLM normalizes/corrects it, and SymPy parses to symbolic form. Verification by round-tripping: `latex(parse_latex(nougat_output))` should be semantically equivalent.

2. **Jupyter AI as orchestration layer**: Use Jupyter AI's `%%ai` magic with variable interpolation to chain: extract equations -> parse -> solve -> plot, all within a single notebook.

3. **MATLAB MCP for numerical ground truth**: Use MATLAB's Symbolic Math Toolbox as an independent verification of SymPy's symbolic results, and MATLAB's numerical solvers for validation.

4. **Multi-backend SymPy parsing**: Try ANTLR first (strict=True), fall back to Lark, fall back to LLM-based parsing. Each layer handles increasingly non-standard LaTeX.

5. **Knowledge graph-guided retrieval**: Use InfraNodus-style concept graphs to identify which papers/equations are relevant for a given scientific question, then extract and parse only those.

### 7.3 Maturity Assessment

| Tool | Maturity | Production-Ready? | Key Risk |
|------|----------|-------------------|----------|
| Nougat | Stable (v0.1.0) | Yes, with error tolerance | 23.5% equation error rate |
| Jupyter AI | Incubation | Yes, for interactive use | Provider API cost/reliability |
| SymPy LaTeX Parser | Stable (ANTLR), Experimental (Lark) | Yes (ANTLR), No (Lark) | Limited construct support |
| MATLAB MCP | Released (v1.x) | Yes | Requires MATLAB license |
| InfraNodus VSCode | Alpha | No | Cloud dependency, no offline |

### 7.4 Performance Characteristics

| Tool | Latency | Throughput | Resource Requirements |
|------|---------|------------|----------------------|
| Nougat (base) | ~3.25s/page | 6 pages parallel | 24GB GPU (A10G) |
| Nougat (small) | ~2s/page (est.) | 6 pages parallel | 16GB GPU (est.) |
| SymPy parse_latex() | <10ms | 1000s/sec | CPU only |
| Jupyter AI %%ai | 1-30s (API dependent) | 1 request at a time | Network + API key |
| MATLAB MCP | ~1s startup + execution time | 1 session per tool call | MATLAB license + install |
| InfraNodus | 2-5s (API dependent) | 1 request at a time | Network + subscription |

---

## Appendix A: Key File Paths in Each Repository

### Nougat
- `nougat/model.py` -- SwinEncoder + BARTDecoder architecture
- `nougat/postprocessing.py` -- Equation cleanup, repetition detection, hallucination removal
- `nougat/dataset/rasterize.py` -- PDF-to-image pipeline (pypdfium2, 96 DPI)

### Jupyter AI
- `packages/jupyter-ai-magics/` -- Magic command implementation
- `packages/jupyter-ai/` -- JupyterLab extension
- ReadTheDocs: `jupyter-ai.readthedocs.io`

### SymPy LaTeX Parser
- `sympy/parsing/latex/__init__.py` -- `parse_latex()` function with pre-parse validation
- `sympy/parsing/latex/_parse_latex_antlr.py` -- ANTLR backend
- `sympy/parsing/latex/lark/latex_parser.py` -- `LarkLaTeXParser` class
- `sympy/parsing/latex/lark/transformer.py` -- `TransformToSymPyExpr` (full construct handling)
- `sympy/parsing/latex/lark/grammar/latex.lark` -- Complete Lark grammar
- Issue #26128 -- LLM-based LaTeX parsing discussion

### MATLAB MCP Core Server
- `cmd/matlab-mcp-core-server/main.go` -- Entry point
- `internal/adaptors/mcp/tools/singlesession/` -- 5 single-session tool definitions
- `internal/adaptors/mcp/tools/multisession/` -- 4 multi-session tool definitions
- `cmd/matlab-mcp-core-server/assets/instructions.txt` -- AI coding guidelines

### InfraNodus VSCode Extension
- Root `README.md` -- Primary documentation (alpha project)

---

## Appendix B: Version Information

| Tool | Version Analyzed | Last Updated |
|------|-----------------|--------------|
| Nougat | 0.1.0 | 2023-08-25 (paper) |
| Jupyter AI | v3.x (latest) | 2026 (active development) |
| SymPy LaTeX Parser | 1.13+ (master) | 2026 (active development) |
| MATLAB MCP | v1.x | 2026 (active development) |
| InfraNodus VSCode | Alpha | 2025 (active development) |
