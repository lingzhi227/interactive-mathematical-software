# Deep Code Analysis: LLM-SR and PySR

## Table of Contents
1. [LLM-SR: Scientific Equation Discovery via Programming with LLMs](#1-llm-sr)
2. [PySR: High-Performance Symbolic Regression](#2-pysr)
3. [Comparative Analysis](#3-comparative-analysis)
4. [Extensibility Assessment](#4-extensibility-assessment)

---

## 1. LLM-SR

**Repository**: https://github.com/deep-symbolic-mathematics/LLM-SR
**Paper**: ICLR 2025 Oral — "LLM-SR: Scientific Equation Discovery via Programming with Large Language Models" (arXiv:2404.18400)
**License**: Apache 2.0 (derived from DeepMind's FunSearch codebase)

### 1.1 Repository Structure

```
LLM-SR/
├── main.py                          # Entry point — loads data, config, launches pipeline
├── requirements.txt                 # 17 Python dependencies (PyTorch 2.0.1+cu118, transformers, etc.)
├── environment.yml                  # Conda environment definition
├── data/                            # Benchmark datasets (CSV format)
│   ├── oscillator1/train.csv        # Damped nonlinear oscillator (physics)
│   ├── oscillator2/train.csv        # Second oscillator variant
│   ├── bactgrow/train.csv           # E. coli bacterial growth (biology)
│   └── stressstrain/train.csv       # Stress-strain relationship (materials science)
├── specs/                           # Problem specification templates (prompt + eval code)
│   ├── specification_oscillator1_numpy.txt
│   ├── specification_oscillator2_numpy.txt
│   ├── specification_oscillator2_torch.txt
│   ├── specification_bactgrow_numpy.txt
│   └── specification_stressstrain_numpy.txt
├── llmsr/                           # Core framework (8 Python modules)
│   ├── pipeline.py                  # Main orchestration: wires sampler, evaluator, buffer
│   ├── sampler.py                   # LLM interface (local HF models + OpenAI API)
│   ├── evaluator.py                 # Sandboxed execution + fitness scoring
│   ├── evaluator_accelerate.py      # Optional Numba JIT acceleration
│   ├── buffer.py                    # Multi-island evolutionary experience buffer
│   ├── code_manipulation.py         # AST-based code parsing, function extraction
│   ├── config.py                    # Dataclass-based configuration
│   └── profile.py                   # TensorBoard logging + JSON sample tracking
└── llm_engine/
    └── engine.py                    # Flask server wrapping HuggingFace models for inference
```

**Total codebase size**: ~1,500 lines of Python across 10 source files. Compact and readable.

### 1.2 Architecture

LLM-SR implements an **evolutionary program synthesis** framework where an LLM generates candidate equation "program skeletons" and an island-based evolutionary algorithm selects and refines them.

#### High-Level Data Flow

```
┌─────────────┐     prompt      ┌─────────────┐     code sample     ┌─────────────┐
│  Experience  │ ──────────────> │   Sampler    │ ──────────────────> │  Evaluator   │
│   Buffer     │ <────────────── │  (LLM call)  │                    │  (Sandbox)   │
│  (Islands)   │   register      └─────────────┘                    └──────┬───────┘
│              │   program                                                  │
│              │ <─────────────────────────────────────────────────────────┘
└─────────────┘                          score + function
```

1. **ExperienceBuffer** selects an island, samples programs from clusters, constructs a versioned prompt
2. **Sampler** sends prompt to LLM (local or API), extracts function body from response
3. **Evaluator** compiles the generated code into a runnable program via AST manipulation, executes it in a sandboxed subprocess with timeout, returns a fitness score
4. **ExperienceBuffer** registers the program in the appropriate island/cluster
5. Loop repeats up to `max_sample_nums` (default 10,000)

#### Key Design Decisions
- **Equations as Python functions**: Equations are represented as Python function bodies (not symbolic trees). The LLM generates executable code, not mathematical expressions.
- **Program skeletons**: The LLM fills in a function body template. Parameters are optimized numerically (BFGS or Adam) within the evaluation function.
- **Multi-island evolution**: 10 islands maintain diverse populations. The weakest half are periodically reset and re-seeded from the strongest.
- **Cluster-based diversity**: Within each island, programs are grouped by their score signature (tuple of per-test scores). This prevents convergence to a single solution.

### 1.3 Key APIs and Interfaces

#### Entry Point: `main.py`

```python
python main.py --problem_name oscillator1 \
               --spec_path specs/specification_oscillator1_numpy.txt \
               --log_path ./logs/oscillator1 \
               [--use_api True --api_model gpt-3.5-turbo]  # or local LLM
               [--port 5000]                                # local engine port
```

Command-line arguments:
- `--problem_name`: Dataset name (maps to `data/{name}/train.csv`)
- `--spec_path`: Path to specification template file
- `--log_path`: TensorBoard + JSON log output directory
- `--use_api` / `--api_model`: Use OpenAI API instead of local model
- `--run_id`: Experiment identifier

#### Pipeline API: `pipeline.main()`

```python
def main(
    specification: str,           # Full specification template text
    inputs: Sequence[Any],        # Dataset dict: {'data': {'inputs': X, 'outputs': y}}
    config: config_lib.Config,    # Hyperparameters
    max_sample_nums: int | None,  # Max LLM queries (None = unlimited)
    class_config: config_lib.ClassConfig,  # LLM + Sandbox class injection
    **kwargs                      # log_dir, etc.
) -> None
```

#### Configuration: `config.py`

```python
@dataclasses.dataclass(frozen=True)
class Config:
    experience_buffer: ExperienceBufferConfig  # Island evolution params
    num_samplers: int = 1          # Parallel samplers (serial in practice)
    num_evaluators: int = 1        # Parallel evaluators
    samples_per_prompt: int = 4    # LLM samples per prompt (batch size)
    evaluate_timeout_seconds: int = 30  # Per-evaluation timeout
    use_api: bool = False          # OpenAI API vs local model
    api_model: str = "gpt-3.5-turbo"

@dataclasses.dataclass(frozen=True)
class ExperienceBufferConfig:
    functions_per_prompt: int = 2                  # Num previous solutions in prompt
    num_islands: int = 10                          # Island count
    reset_period: int = 4 * 60 * 60                # 4 hours between resets
    cluster_sampling_temperature_init: float = 0.1 # Softmax temperature for cluster selection
    cluster_sampling_temperature_period: int = 30_000  # Temperature annealing period
```

### 1.4 Core Algorithm Implementation

#### 1.4.1 Specification Templates (The Prompt Format)

A specification file (e.g., `specification_oscillator1_numpy.txt`) is a complete Python program with two decorated functions:

- `@equation.evolve` — the function whose body the LLM generates (the equation skeleton)
- `@evaluate.run` — the fitness evaluation function (fixed, not evolved)

Example structure (oscillator1):
```python
import numpy as np
from scipy.optimize import minimize

# 10 optimizable parameters
params = np.ones(10)

@evaluate.run
def evaluate(dataset):
    """Optimize parameters via BFGS, return negative MSE as score."""
    X, y = dataset['inputs'], dataset['outputs']
    def loss(params):
        x, v = X[:, 0], X[:, 1]
        pred = equation(x, v, params)
        return np.mean((pred - y) ** 2)
    result = minimize(loss, params, method='BFGS')
    loss_val = result.fun
    if np.isnan(loss_val) or np.isinf(loss_val):
        return None
    return -loss_val  # Higher is better

@equation.evolve
def equation(x, v, params):
    """Acceleration as function of position x, velocity v, and params."""
    dv = params[0]*x + params[1]*v + params[2]  # Initial linear model
    return dv
```

Key insight: The specification separates *structural search* (LLM evolves the equation form) from *parameter optimization* (SciPy BFGS optimizes numerical coefficients). This is a fundamental design principle — the LLM proposes the functional form, and classical optimization fits the constants.

#### 1.4.2 The Evolutionary Loop (`buffer.py`)

The experience buffer implements a **multi-island evolutionary algorithm with cluster-based diversity maintenance**:

**Island Model** (`ExperienceBuffer`):
- `num_islands = 10` independent sub-populations
- Each island maintains a dictionary of `Cluster` objects, keyed by score signature
- Periodic reset: every 4 hours, the weakest 5 islands are cleared and re-seeded with the best program from a surviving island
- `get_prompt()`: randomly selects an island, then samples `functions_per_prompt` (default 2) programs from clusters using softmax-temperature selection over cluster scores

**Cluster** objects group programs with identical score signatures (performance fingerprints):
- `sample_program()`: prefers shorter programs via softmax over negative code lengths
- This creates implicit parsimony pressure — among equally-performing programs, shorter ones are more likely to be selected as prompt exemplars

**Prompt Construction** (`Island._generate_prompt()`):
- Selected programs are sorted by score (worst to best)
- Each is renamed to `equation_v0`, `equation_v1`, etc., with docstrings saying "Improved version of equation_v{i-1}"
- A new empty function `equation_v{N}` is appended with docstring "Improved version of equation_v{N-1}"
- The LLM then completes this function body
- This versioning scheme tells the LLM these are progressively better solutions, guiding it to generate improvements

**Temperature Annealing**:
- Cluster sampling temperature decays linearly from `cluster_sampling_temperature_init` (0.1) to near 0 over `cluster_sampling_temperature_period` (30,000) samples
- This shifts from exploration (uniform sampling across clusters) to exploitation (focusing on the best clusters)
- Temperature resets each period, creating cyclical exploration-exploitation phases

#### 1.4.3 Sampling from the LLM (`sampler.py`)

**Abstract LLM Interface**:
```python
class LLM(ABC):
    def draw_samples(self, prompt: str) -> Collection[str]:
        """Return multiple predicted continuations of prompt."""
```

**LocalLLM Implementation** — supports two backends:

1. **Local HuggingFace models** via Flask server:
   - System prompt: "You are a helpful assistant tasked with discovering mathematical function structures for scientific systems. Complete the 'equation' function below, considering the physical meaning and relationships of inputs."
   - Batch inference: repeats the prompt `samples_per_prompt` times in a single forward pass
   - Calls `http://127.0.0.1:5000/completions` with JSON payload
   - Parameters: `do_sample=True`, configurable temperature/top_k/top_p

2. **OpenAI API**:
   - Uses `http.client.HTTPSConnection` directly (not the openai library)
   - `max_tokens=512`, model via `config.api_model`
   - Sequential sampling (one API call per sample, not batched)
   - Reads `API_KEY` from environment variable

**Response Processing** (`_extract_body()`):
- Finds the first `def` statement in the LLM response
- Extracts everything after it as the function body
- For local models: ensures 4-space indentation
- For API models: takes code as-is after the `def` line

#### 1.4.4 Evaluation (`evaluator.py`)

**Sandbox Execution** (`LocalSandbox.run()`):
- Uses `multiprocessing.Process` for isolation
- Timeout enforcement via `process.join(timeout=timeout_seconds)`
- Code is executed with `exec(program, all_globals_namespace)`, then the evaluation function is called
- Results communicated via `multiprocessing.Queue`
- Only `int` or `float` return values accepted (safety check)

**Program Assembly** (`_sample_to_program()`):
- `_trim_function_body()`: Uses AST parsing to extract valid function body, progressively removing lines that cause `SyntaxError`
- Renames any self-referential function calls (e.g., `equation_v2` -> `equation`)
- Inserts generated body into a deep copy of the template
- Returns compiled function + full runnable program string

**Ancestor Check** (`_calls_ancestor()`):
- Rejects programs that call previous versions (e.g., `equation_v1` calling `equation_v0`)
- Prevents trivial wrapper solutions

#### 1.4.5 LLM Engine (`llm_engine/engine.py`)

A standalone Flask server that wraps any HuggingFace causal LM:

```python
python engine.py --model_path mistralai/Mixtral-8x7B-Instruct-v0.1 \
                 --gpu_ids 0 1 2 3 --quantization --port 5000
```

Key features:
- **4-bit quantization** via BitsAndBytesConfig (optional)
- **Multi-GPU** via `device_map='auto'`
- **Batch inference**: stacks repeated prompts via `torch.vstack([inputs] * repeat_prompt)`
- **OOM recovery**: catches `torch.cuda.OutOfMemoryError`, clears cache, retries
- **Chat template**: uses `tokenizer.apply_chat_template()` for instruction-tuned models
- Generation params: `temperature=0.8`, `top_k=30`, `top_p=0.9`, `max_new_tokens=512`

### 1.5 Equation Representation

Equations in LLM-SR are **Python function bodies** — raw executable code. This is fundamentally different from traditional symbolic regression which uses expression trees.

Example evolved equation for the damped oscillator:
```python
def equation(x, v, params):
    dv = params[0] * np.sin(params[1] * x) + params[2] * v**3 + params[3] * np.exp(-params[4] * x**2)
    return dv
```

The representation has these properties:
- **Arbitrary complexity**: Can express any computable function, not limited to a fixed operator set
- **NumPy/PyTorch compatible**: Uses standard numerical libraries
- **Parameter separation**: Numerical constants are explicit `params[i]` references, optimized by BFGS/Adam
- **Human-readable**: The generated code is directly interpretable
- **No fixed grammar**: Unlike expression trees, there is no predefined grammar constraining the search space

The `code_manipulation.py` module provides AST-based tools for manipulating these code representations:
- `Function` dataclass: stores `name`, `args`, `body`, `return_type`, `docstring`, `score`
- `Program` dataclass: stores `preface` (imports/globals) + list of `Function` objects
- `text_to_program()` / `text_to_function()`: parse code strings into structured objects
- `rename_function_calls()`: tokenizer-based function call renaming
- `get_functions_called()`: extract called function names
- `yield_decorated()`: find functions with specific decorators

### 1.6 LLM Integration

**Supported LLMs**:
1. Any HuggingFace causal language model (via `llm_engine/engine.py`)
   - Tested with: `mistralai/Mixtral-8x7B-Instruct-v0.1`
   - Supports 4-bit/8-bit quantization
   - Multi-GPU with `device_map='auto'`
2. OpenAI API models
   - Tested with: `gpt-3.5-turbo`, `gpt-4o`
   - Direct HTTP calls (no SDK dependency)

**Prompt Template** (constructed dynamically):
```
You are a helpful assistant tasked with discovering mathematical function
structures for scientific systems. Complete the 'equation' function below,
considering the physical meaning and relationships of inputs.

[specification preface: imports, global variables]

def equation_v0(x, v, params):
    """[original docstring]"""
    [body of worst selected program]

def equation_v1(x, v, params):
    """Improved version of equation_v0."""
    [body of better selected program]

def equation_v2(x, v, params):
    """Improved version of equation_v1."""
    [LLM completes this...]
```

The LLM sees:
- The full problem context (imports, variable meanings, parameter setup)
- 2 progressively better previous solutions with scores implicitly encoded by ordering
- A new function header to complete, framed as an "improvement"

### 1.7 Dependencies

From `requirements.txt`:
- **Deep Learning**: torch==2.0.1+cu118, transformers==4.38.1, accelerate==0.27.2
- **Quantization**: bitsandbytes==0.42.0
- **Scientific Computing**: numpy==1.26.4, scipy==1.12.0, pandas==2.2.1, scikit-learn==1.4.1
- **Web/API**: flask==3.0.2, flask-cors==4.0.0, fastapi==0.110.0, uvicorn==0.27.1
- **Monitoring**: tensorboard==2.16.2, matplotlib==3.8.3
- **Model Hub**: huggingface-hub==0.20.3

### 1.8 Reproducibility

**Ease of setup**: Moderate
- Standard conda/pip installation
- For local LLM: requires GPU with sufficient VRAM (Mixtral-8x7B needs ~24GB with 4-bit quantization)
- For API: just needs an OpenAI API key

**Data availability**: All 4 benchmark datasets included in the repository as CSV files.

**Stochastic elements**: Results vary across runs due to:
- LLM sampling temperature (0.8)
- Random island/cluster selection
- Random evaluator assignment
- No random seed control in the main loop

**Missing from repo**:
- No pre-trained results or log files included
- No scripts for running all benchmarks
- No comparison baselines code

### 1.9 Code Quality

**Strengths**:
- Clean, well-structured codebase with clear separation of concerns
- Thorough docstrings on most classes and methods
- Good use of dataclasses for configuration
- AST-based code manipulation is robust
- Sandboxed evaluation prevents crashes from bad generated code

**Weaknesses**:
- No unit tests
- No type checking (mypy) despite type hints
- `exec()` for code execution is inherently risky (mitigated by subprocess isolation)
- Error handling in LLM calls uses bare `except Exception: continue` (infinite retry on any error)
- The `Sampler._global_samples_nums` class variable for cross-instance state is fragile
- No logging framework (uses print statements)
- Pipeline is sequential despite multi-sampler/evaluator design ("without parallelization only the first sampler will do any work")

### 1.10 Extensibility

**Adding new problems**: Easy
- Create a CSV dataset in `data/{problem_name}/train.csv`
- Write a specification template in `specs/` defining the equation function signature and evaluation logic
- Run with `--problem_name` and `--spec_path`

**Adding new LLMs**: Moderate
- Local models: Any HuggingFace model works via the engine
- Other APIs: Requires modifying `LocalLLM._draw_samples_api()` or subclassing `LLM`

**Changing the evolutionary algorithm**: Moderate
- Buffer parameters are configurable via `ExperienceBufferConfig`
- Algorithm logic is in `buffer.py` — can modify island reset strategy, cluster selection, prompt construction

**Changing equation representation**: Hard
- Deeply coupled to Python code generation paradigm
- Would require modifying specification templates, code_manipulation, and evaluator

---

## 2. PySR

**Repository**: https://github.com/MilesCranmer/PySR
**Paper**: arXiv:2305.01582 — "Interpretable Machine Learning for Science with PySR and SymbolicRegression.jl"
**License**: Apache 2.0

### 2.1 Repository Structure

```
PySR/
├── pysr/                            # Main Python package
│   ├── __init__.py                  # Public API exports (17 symbols)
│   ├── __main__.py                  # CLI entry point
│   ├── sr.py                        # PySRRegressor class (~2000+ lines, core logic)
│   ├── expression_specs.py          # Expression type specifications
│   ├── export_sympy.py              # SymPy conversion (50+ operator mappings)
│   ├── export_torch.py              # PyTorch module generation from equations
│   ├── export_jax.py                # JAX function generation
│   ├── export_numpy.py              # NumPy lambda generation
│   ├── export_latex.py              # LaTeX rendering
│   ├── export.py                    # Unified export orchestration
│   ├── feature_selection.py         # RandomForest-based feature selection
│   ├── denoising.py                 # Gaussian process denoising
│   ├── julia_helpers.py             # Julia <-> Python data conversion
│   ├── julia_import.py              # Julia runtime initialization
│   ├── julia_extensions.py          # Julia package management
│   ├── julia_registry_helpers.py    # Julia registry configuration
│   ├── utils.py                     # Type aliases and utilities
│   ├── logger_specs.py              # TensorBoard logging specs
│   ├── deprecated.py                # Backward-compatible function stubs
│   ├── sklearn_monkeypatch.py       # Sklearn compatibility patches
│   ├── param_groupings.yml          # Parameter grouping metadata
│   ├── juliapkg.json                # Julia dependency specification
│   ├── _cli/                        # Command-line interface
│   └── test/                        # Comprehensive test suite
│       ├── test_main.py             # Core functionality tests
│       ├── test_torch.py            # PyTorch export tests
│       ├── test_jax.py              # JAX export tests
│       ├── test_autodiff.py         # Autodifferentiation tests
│       ├── test_cli.py              # CLI tests
│       ├── test_startup.py          # Initialization tests
│       ├── test_dev.py              # Development tests
│       ├── test_slurm.py            # SLURM cluster tests
│       ├── test_nb.ipynb            # Notebook tests
│       └── params.py                # Test parameter configurations
├── Dockerfile                       # Docker build
├── Apptainer.def                    # Apptainer/Singularity for HPC clusters
├── setup.py / pyproject.toml        # Package configuration
└── docs/                            # Documentation site source
```

**Total codebase size**: ~5,000+ lines of Python. The Julia backend (SymbolicRegression.jl) adds ~10,000+ lines of Julia.

### 2.2 Architecture

PySR is a **Python wrapper** around **SymbolicRegression.jl**, a high-performance Julia library implementing multi-population evolutionary symbolic regression.

#### Architecture Layers

```
┌──────────────────────────────────────────────┐
│           User Python Code                    │
│  model = PySRRegressor(...)                   │
│  model.fit(X, y)                              │
│  model.predict(X_new)                         │
├──────────────────────────────────────────────┤
│       PySRRegressor (sr.py)                   │
│  - Scikit-learn compatible API                │
│  - Data validation & preprocessing            │
│  - Feature selection, denoising               │
│  - Parameter conversion to Julia types        │
├──────────────────────────────────────────────┤
│    Julia Bridge (julia_helpers.py)             │
│  - jl_array(), jl_dict(), jl_serialize()      │
│  - juliacall for Python<->Julia interop       │
├──────────────────────────────────────────────┤
│    SymbolicRegression.jl (Julia)              │
│  - Multi-population evolutionary search       │
│  - Expression tree representation             │
│  - Mutation, crossover, simplification        │
│  - Hall of Fame tracking                      │
│  - Distributed/threaded parallelism           │
└──────────────────────────────────────────────┘
```

#### Data Flow

1. User calls `model.fit(X, y)` with numpy arrays
2. `_validate_and_set_fit_params()`: validates shapes, handles DataFrames, manages units
3. `_pre_transform_training_data()`: optional feature selection (RandomForest) and denoising (GP)
4. `_run()`: converts all parameters to Julia types, constructs Julia Options struct, calls `SymbolicRegression.equation_search()`
5. Julia performs evolutionary search, returns hall of fame
6. Results parsed into `equations_` pandas DataFrame with multiple export formats
7. `model.predict()` evaluates the selected best equation on new data

### 2.3 Key APIs and Interfaces

#### Primary API: `PySRRegressor`

```python
from pysr import PySRRegressor

model = PySRRegressor(
    # Search parameters
    niterations=40,               # Number of evolutionary iterations
    populations=8,                # Number of parallel populations
    population_size=50,           # Individuals per population
    ncycles_per_iteration=500,    # Mutations per iteration per population

    # Operator configuration
    binary_operators=["+", "*", "-", "/"],
    unary_operators=["cos", "exp", "sin", "inv(x) = 1/x"],

    # Complexity control
    maxsize=20,                   # Max expression tree nodes
    maxdepth=10,                  # Max expression tree depth
    constraints={"/": (-1, 9)},   # Per-operator complexity limits
    nested_constraints={          # Nested operator restrictions
        "exp": {"exp": 0},        # No exp(exp(...))
    },

    # Loss configuration
    elementwise_loss="loss(prediction, target) = (prediction - target)^2",

    # Performance
    parallelism="multithreading", # or "serial", "multiprocessing"
    precision=64,                 # Float precision
    turbo=True,                   # Faster evaluation mode
    batching=True,                # Use data batching
    batch_size=50,

    # Output
    model_selection="best",       # or "accuracy", "score"
    output_directory="outputs",
)

model.fit(X, y)
print(model.equations_)           # DataFrame of discovered equations
best = model.get_best()           # Best equation by selection criterion
y_pred = model.predict(X_new)     # Predictions from best equation
```

#### Key Method Signatures

```python
# Core fitting
def fit(self, X, y, Xresampled=None, weights=None, variable_names=None,
        complexity_of_variables=None, X_units=None, y_units=None) -> self

# Prediction
def predict(self, X, index=None) -> ndarray

# Best equation access
def get_best(self, index=None) -> pd.Series | list[pd.Series]

# Export formats
model.sympy()      # SymPy expression
model.latex()       # LaTeX string
model.jax()         # JAX callable
model.pytorch()     # PyTorch nn.Module

# Persistence
model.save("model.pkl")
PySRRegressor.from_file("model.pkl")
```

#### Expression Specification System

PySR supports multiple expression types via the `expression_spec` parameter:

```python
# Standard single expression (default)
ExpressionSpec()

# Template-based multi-expression
TemplateExpressionSpec(
    combine="((; f, g), x1, x2) -> sin(f(x1, x2)) + g(x1)",
    num_features={"f": 2, "g": 1}
)

# Parametric expressions (deprecated, use TemplateExpressionSpec)
ParametricExpressionSpec(max_parameters=3)
```

### 2.4 Core Algorithm Implementation

PySR's search algorithm is implemented in Julia (SymbolicRegression.jl). The Python side manages configuration, data preprocessing, and result presentation.

#### 2.4.1 Evolutionary Search (Julia Backend)

The Julia backend implements a sophisticated multi-population evolutionary algorithm:

**Population Management**:
- `populations` independent populations evolve in parallel
- Each population has `population_size` individuals (expression trees)
- Periodic migration exchanges best individuals between populations
- `ncycles_per_iteration` mutation cycles per evolution iteration

**Mutation Operations** (configurable weights):
- `weight_mutate_constant`: Perturb numerical constants
- `weight_mutate_operator`: Change an operator node
- `weight_add_node`: Insert a new node into the tree
- `weight_delete_node`: Remove a node from the tree
- `weight_simplify`: Algebraic simplification
- `weight_randomize`: Generate completely random subtree
- `weight_insert_node`: Insert node at specific position
- `crossover_probability`: Combine subtrees from two parents

**Selection**:
- Tournament selection with configurable `tournament_selection_n` and `tournament_selection_p`
- Hall of Fame tracks Pareto front of accuracy vs. complexity
- Model selection strategies: "best" (Pareto optimal), "accuracy" (lowest loss), "score" (best loss/complexity tradeoff)

**Constant Optimization**:
- Built-in constant optimization via BFGS
- `optimizer_algorithm`: "BFGS" (default) or "NelderMead"
- `optimizer_nrestarts`: Number of optimization restarts
- `optimize_probability`: Probability of optimizing constants per mutation

#### 2.4.2 Julia Bridge (`julia_helpers.py`)

The bridge between Python and Julia uses `juliacall`:

```python
from juliacall import convert as jl_convert
from .julia_import import jl

# Data conversion
jl_array(numpy_array, dtype)      # numpy -> Julia Array
jl_dict(python_dict)              # dict -> Julia Dict
jl_named_tuple(dict)              # dict -> Julia NamedTuple

# Serialization (for model persistence)
jl_serialize(julia_obj) -> NDArray[np.uint8]  # Julia object -> bytes
jl_deserialize(bytes) -> julia_obj            # bytes -> Julia object

# Julia evaluation
jl.seval("using SymbolicRegression: plus, sub, mult, div, pow")
```

#### 2.4.3 Feature Selection (`feature_selection.py`)

```python
def run_feature_selection(X, y, select_k_features, random_state=None):
    """Use RandomForestRegressor to find k most important features."""
    clf = RandomForestRegressor(n_estimators=100, max_depth=3)
    clf.fit(X, y)
    selector = SelectFromModel(clf, max_features=select_k_features, prefit=True)
    return selector.get_support(indices=False)  # Boolean mask
```

#### 2.4.4 Denoising (`denoising.py`)

```python
def denoise(X, y, Xresampled=None, random_state=None):
    """Denoise dataset using Gaussian Process Regression."""
    gp_kernel = RBF(np.ones(X.shape[1])) + WhiteKernel(1e-1) + ConstantKernel()
    gpr = GaussianProcessRegressor(kernel=gp_kernel, n_restarts_optimizer=50)
    gpr.fit(X, y)
    return X, gpr.predict(X)  # or Xresampled, gpr.predict(Xresampled)
```

### 2.5 Equation Representation

PySR uses **expression trees** — the standard representation for symbolic regression:

#### Internal Representation (Julia)
In Julia, expressions are stored as typed tree structures with nodes for:
- Binary operators (`+`, `*`, `-`, `/`, `pow`, etc.)
- Unary operators (`cos`, `sin`, `exp`, `sqrt`, etc.)
- Variables (indexed by feature column)
- Constants (floating-point values, optimizable)

#### Export Formats (Python)

PySR provides 5 export formats, all derived from the internal Julia tree:

1. **SymPy** (`export_sympy.py`): Converts to `sympy.Expr` objects
   - 50+ operator mappings defined in `sympy_mappings` dict
   - Supports custom operators via `extra_sympy_mappings`
   - Used for symbolic manipulation, simplification, differentiation

2. **NumPy** (`export_numpy.py`): Generates callable lambda functions
   - Direct numerical evaluation on arrays
   - Used for `model.predict()`

3. **PyTorch** (`export_torch.py`): Generates `torch.nn.Module`
   - `SingleSymPyModule`: recursive tree of `_Node` modules
   - Float constants become `nn.Parameter` (trainable)
   - Rational/integer constants become buffers (fixed)
   - Full autograd support for fine-tuning discovered equations

4. **JAX** (`export_jax.py`): Generates JAX-compatible functions
   - JIT-compilable for accelerated evaluation

5. **LaTeX** (`export_latex.py`): Pretty-printed mathematical notation

#### Operator Mappings Example (`export_sympy.py`)

```python
sympy_mappings = {
    "div": lambda x, y: x / y,
    "sqrt_abs": lambda x: sympy.sqrt(abs(x)),
    "cbrt": lambda x: sympy.sign(x) * sympy.cbrt(abs(x)),
    "pow_abs": lambda x, y: abs(x) ** y,
    "relu": lambda x: sympy.Piecewise((0.0, x < 0), (x, True)),
    "cond": lambda x, y: sympy.Piecewise((y, x > 0), (0.0, True)),
    # ... 50+ total mappings
}
```

#### Results DataFrame

After `model.fit()`, `model.equations_` is a pandas DataFrame:

| complexity | loss | equation | sympy_format | lambda_format | score |
|------------|------|----------|-------------|---------------|-------|
| 1 | 5.23 | x0 | x0 | lambda X: X[:,0] | 0.0 |
| 3 | 1.02 | x0^2 | x0**2 | lambda X: X[:,0]**2 | 0.81 |
| 5 | 0.001 | 2.54*cos(x3)+x0^2 | 2.54*cos(x3)+x0**2 | ... | 3.42 |

### 2.6 LLM Integration

**PySR does not integrate LLMs.** It is a purely evolutionary/genetic programming system. The search is driven entirely by:
- Random mutation of expression trees
- Tournament selection based on fitness
- Crossover between individuals
- Algebraic simplification rules
- Constant optimization via BFGS

This is the fundamental difference from LLM-SR, which uses LLMs to generate candidate solutions rather than random mutations.

### 2.7 Dependencies

**Python**:
- numpy, pandas, scikit-learn (core)
- sympy (symbolic math)
- juliacall (Julia interop)
- Optional: torch, jax (for export)

**Julia** (via `juliapkg.json`):
- SymbolicRegression.jl (core search engine)
- Serialization (model persistence)
- PythonCall (Julia<->Python bridge)
- ClusterManagers (optional, for distributed computing)

### 2.8 Reproducibility

**Ease of setup**: Good
- `pip install pysr` handles everything
- Julia dependencies auto-install on first import
- Docker and Apptainer support for HPC environments
- Deterministic mode available (`deterministic=True`, requires `parallelism="serial"`)

**Stochastic control**:
- `random_state` parameter for reproducibility
- Deterministic mode enforces serial execution for exact reproduction

### 2.9 Code Quality

**Strengths**:
- Comprehensive test suite (8 test files covering core, torch, jax, autodiff, CLI, SLURM)
- Full scikit-learn API compliance (fit/predict/score)
- Extensive documentation with examples
- Type hints throughout
- Graceful deprecation handling
- Multiple export formats with cross-validation
- SLURM/HPC support out of the box

**Weaknesses**:
- Julia dependency adds complexity (installation, startup time)
- Large codebase across two languages (Python + Julia)
- Some monkey-patching for sklearn compatibility

### 2.10 Extensibility

**Adding new operators**: Easy
- Pass custom operators as strings in Julia syntax: `"inv(x) = 1/x"`
- Provide SymPy/Torch/JAX mappings via `extra_*_mappings` parameters

**Adding new loss functions**: Easy
- Custom Julia loss: `elementwise_loss="loss(pred, target) = abs(pred - target)^1.5"`
- Full objective functions via `loss_function` parameter

**Adding new expression types**: Moderate
- Subclass `AbstractExpressionSpec`
- Implement `julia_expression_spec()` and `create_exports()`
- `TemplateExpressionSpec` provides structured multi-expression support

**Adding dimensional analysis**: Built-in
- `X_units` and `y_units` parameters for physical unit constraints
- Search automatically respects dimensional consistency

**Distributed execution**: Built-in
- `parallelism="multiprocessing"` for multi-process
- `cluster_manager` parameter for SLURM/PBS clusters

---

## 3. Comparative Analysis

### 3.1 Fundamental Design Philosophies

| Aspect | LLM-SR | PySR |
|--------|--------|------|
| **Core idea** | LLM generates equation code; evolution selects | Random mutations on expression trees; tournament selection |
| **Search space** | Arbitrary Python functions | Fixed operator set expression trees |
| **Knowledge source** | LLM's pre-trained scientific knowledge | No domain knowledge; purely data-driven |
| **Equation representation** | Python function body (code string) | Expression tree (Julia Node type) |
| **Constant optimization** | Within evaluation (BFGS on params array) | Built-in BFGS on tree constants |
| **Diversity mechanism** | Multi-island + cluster signatures | Multi-population + migration |
| **Implementation language** | Pure Python | Python + Julia |

### 3.2 Strengths and Weaknesses

**LLM-SR Advantages**:
- Can leverage LLM's knowledge of physics, biology, materials science
- Can generate complex functional forms that would be extremely unlikely via random mutation
- Natural language docstrings guide the search (e.g., "Improved version of...")
- Flexible: any computable function, not limited to operator primitives
- Compact codebase (~1,500 lines)

**LLM-SR Disadvantages**:
- Requires LLM inference (expensive GPU or API costs)
- Non-deterministic (LLM sampling)
- No formal guarantees on search coverage
- Security concerns with `exec()` on generated code
- Sequential prompt-sample-evaluate loop is slow
- No test suite

**PySR Advantages**:
- High performance (Julia backend, multithreading)
- Rigorous Pareto front tracking (accuracy vs complexity)
- Deterministic mode for reproducibility
- Comprehensive export formats (SymPy, PyTorch, JAX, LaTeX)
- scikit-learn API compatibility
- Dimensional analysis support
- Production-ready with extensive testing
- No LLM dependency (runs offline, on CPU)

**PySR Disadvantages**:
- No domain knowledge injection (purely syntactic mutations)
- Limited to predefined operator set
- Julia dependency adds installation/startup complexity
- Cannot discover fundamentally novel functional forms outside operator vocabulary
- Struggles with high-dimensional problems

### 3.3 Complementarity

LLM-SR and PySR solve the same problem (symbolic regression) but from opposite directions:

- **PySR** is a bottom-up, exhaustive search: tries all combinations of operators and constants, selects the best. Strong for well-defined search spaces with known operators.
- **LLM-SR** is a top-down, knowledge-guided search: uses the LLM's scientific priors to propose plausible functional forms, then refines via evolution. Strong for problems where domain knowledge can narrow the search space.

A hybrid approach could combine both: use LLM-SR to propose initial functional forms (warm start), then use PySR-style mutations to refine them.

### 3.4 Performance Characteristics

| Metric | LLM-SR | PySR |
|--------|--------|------|
| **Startup time** | Seconds (Python) + model load | 30-60s (Julia JIT compilation) |
| **Per-iteration cost** | High (LLM inference) | Low (tree mutations) |
| **GPU requirement** | Yes (for local LLM) or API | No (CPU-based) |
| **Parallelism** | Designed but not implemented | Full multi-threading + distributed |
| **Typical runtime** | Hours (10K LLM samples) | Minutes to hours |
| **Memory** | High (LLM in VRAM) | Low (expression trees) |

---

## 4. Extensibility Assessment

### 4.1 Adapting LLM-SR for New Domains

**Steps**:
1. Prepare dataset as `data/{name}/train.csv` (features + target)
2. Write a specification template in `specs/`:
   - Define the equation function signature with domain-appropriate variable names
   - Write the evaluation function (fitness = negative loss after parameter optimization)
   - Provide initial placeholder equation
3. Run with appropriate LLM

**Difficulty**: Low for standard regression problems. Higher for problems requiring custom evaluation metrics, multi-objective optimization, or constrained search spaces.

**Key limitation**: The specification template format couples the problem definition to a specific code structure. For significantly different problem types (e.g., PDE discovery, multi-output systems), the template format would need extension.

### 4.2 Adapting PySR for New Domains

**Steps**:
1. Define custom operators if needed (Julia syntax strings)
2. Provide SymPy/PyTorch/JAX mappings for custom operators
3. Set dimensional constraints via `X_units` / `y_units`
4. Configure complexity constraints for the domain

**Difficulty**: Low. PySR's API is designed for easy domain adaptation. The `TemplateExpressionSpec` enables structured multi-expression discovery (e.g., ODEs, coupled systems).

### 4.3 Potential Hybrid Architectures

1. **LLM-guided PySR**: Use an LLM to suggest operator sets and complexity constraints before running PySR
2. **PySR-refined LLM-SR**: Use LLM-SR to discover functional forms, then refine constants/structure with PySR's evolutionary search
3. **LLM as mutation operator in PySR**: Replace random mutations with LLM-suggested modifications to expression trees
4. **Shared evaluation**: Both systems could use the same fitness evaluation framework

### 4.4 Integration Considerations

For a unified system combining LLM-based and evolutionary approaches:

- **Equation representation**: Need a common format. SymPy expressions are the natural choice (both can convert to/from SymPy)
- **Evaluation framework**: PySR's Julia-based evaluation is faster; LLM-SR's Python-based evaluation is more flexible
- **Search coordination**: LLM proposals could seed PySR populations, or PySR's hall of fame could inform LLM prompts
- **Cost management**: LLM calls are expensive; use them strategically (e.g., only when evolution stagnates)
