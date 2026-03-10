# LLM-Assisted Scientific Reasoning & Knowledge Tools: A Comprehensive Survey

**From Coding Assistants to Scientific Knowledge Transformers**

**Date**: 2026-03-10 | **Papers surveyed**: 82 | **Papers deep-read**: 12 | **Repos deeply analyzed**: 12 | **Total code analysis**: 3,117 lines

---

## Executive Summary

This survey examines how Large Language Models are being used to transform complex scientific knowledge — mathematical equations, physics theories, computational methods — into human-understandable formats including knowledge trees, natural language explanations, and executable code. We identify a 5-stage knowledge transformation pipeline spanning document extraction, symbolic mathematics, formal verification, code generation, and interactive presentation. Based on 82 papers from 2013-2026 (including 12 deep-read), we find:

1. **Code-as-reasoning is the dominant paradigm**: Equations represented as executable programs (LLM-SR, SymCode, CodePDE) enable both human understanding and machine verification. The PAL/PoT lineage from 2023 has matured into production-ready systems.

2. **Autoformalization is expanding beyond pure math**: Lean4 now covers physics (Lean4Physics, ICLR 2026), quantum computing (MerLean), and PDEs (PDE-Controller, ICML 2025). However, the formalization gap remains large (best: 39.5% on physics).

3. **Multi-agent architectures dominate scientific AI**: 8 of 12 deep-read systems use specialized agent teams. The generate-debug-refine loop with external verification is universal.

4. **Nonlinear knowledge interfaces outperform linear chat**: RCTs show +11% retention (Learn Your Way) and significant cognitive load reduction (Mindalogue, CHI 2025) from tree/graph/canvas interfaces.

5. **The critical gap is integration**: No system spans all three knowledge formats (natural language ↔ LaTeX math ↔ numerical code). Deep code analysis of 12 repos reveals four viable integration paths, with MCP-based Science CLI as the highest-feasibility option.

---

## 1. Introduction

### 1.1 Motivation

Scientists interact with knowledge in three formats:
- **Natural language**: Textbooks, papers, lectures
- **Mathematical notation**: Equations in LaTeX, symbolic expressions
- **Numerical code**: Python/Julia implementations, simulation scripts

The barrier to scientific understanding often lies in the transitions between these formats. A physicist reads the Dirac equation in LaTeX but must mentally translate it to understand the physical implications, then write code to simulate it. LLMs offer the possibility of automating these transitions, making scientific knowledge more accessible.

### 1.2 Scope

This survey covers research at the intersection of:
- LLM-assisted mathematical reasoning and equation understanding
- Scientific knowledge representation (knowledge graphs, ontologies)
- Autoformalization (natural language → formally verified code)
- Code generation for scientific computing
- Interactive learning interfaces for STEM
- IDE/CLI tool extensions for scientific workflows

### 1.3 Research Questions

1. How can LLMs transform mathematical equations into human-understandable knowledge representations?
2. What is the state of automated translation between natural language, mathematical notation, and numerical code?
3. How can coding assistant paradigms (Codex, Claude Code) be extended for scientific reasoning?
4. What verification methods ensure correctness of LLM-generated scientific code?
5. What gaps remain for building integrated scientific knowledge management systems?

---

## 2. The Knowledge Transformation Pipeline

### 2.1 Five Stages

We identify five stages in transforming scientific knowledge from raw form to human-understandable interactive representations:

**Stage 1: Document Extraction** (LaTeX → structured data)
- Nougat [@blecher2023nougat] (ECCV 2024): Vision Transformer for PDF→LaTeX/markup
- Mathpix: Commercial API with ~99% accuracy on printed equations
- TeXpert [@kale2025texpert] (ACL 2025): Benchmark showing LLMs struggle with complex LaTeX

**Stage 2: Symbolic Mathematics** (LaTeX → executable symbolic code)
- LLM-SR [@shojaee2024llmsr] (ICLR 2025 Oral): Equations as executable Python programs with evolutionary search. Achieves 100-1000x lower NMSE than PySR/DSR with 1000x fewer iterations.
- SymCode [@bagherinezhad2025symcode] (EACL 2026): Math→SymPy verified code with self-debugging. +13.6pp accuracy, 60-77% token reduction.
- QuaSAR [@ranaldi2025quasar] (ACL 2025): Quasi-symbolic CoT without full formalization. +8% on adversarial benchmarks.
- ASyMOB [@shalyt2025asymob]: LaTeX→SymPy parsing pipeline with LLM fallback (18.5% of cases).

**Stage 3: Formal Verification** (symbolic → machine-verified)
- PDE-Controller [@soroco2025pdecontroller] (ICML 2025): NL→Signal Temporal Logic for PDEs. >64% formalization accuracy, >82% code generation.
- MerLean [@ren2026merlean]: Agentic LaTeX→Lean4 for quantum computation. 2,050 declarations from 114 statements across 3 papers.
- Lean4Physics [@li2025lean4physics] (ICLR 2026): First physics benchmark in Lean4. PhysLib provides +11.75% improvement.
- PhysLean/HepLean [@toobysmith2025physlean] (Comp.Phys.Comm. 2025): Maxwell, QHO, Wick's theorem in Lean4.
- Vericoding Benchmark [@bursuc2025vericoding]: 12,504 specs. Dafny improved 68%→96% in one year.

**Stage 4: Code Generation** (formal/symbolic → numerical solver)
- CodePDE [@li2025codepde] (TMLR): PDE solving as code generation. Expert-competitive with 4 debug rounds. Bug-free rate: 41%→84%.
- AutoNumerics [@du2026autonumerics]: Multi-agent pipeline for 24 PDE problems with coarse-to-fine execution.
- MCP-SIM [@park2026mcpsim] (npj AI 2026): 100% success on 12 physics simulation tasks via Plan-Act-Reflect-Revise.
- DREAMS [@wang2025dreams]: Multi-agent DFT simulation with <1% error vs. human experts.

**Stage 5: Human-Understandable Presentation** (code → interactive knowledge)
- Learn Your Way [@google2025learnyourway]: Multi-representation AI textbook. RCT: +11% retention, all components >0.90 pedagogical rating.
- Mindalogue [@zhang2024mindalogue] (CHI 2025): Nonlinear "nodes+canvas" interface. Outperforms linear chat on 4/5 dimensions.
- AutoMathKG [@bian2025automathkg]: 13,388-entity mathematical knowledge graph with semantic search.
- Concept Map Generation [@zhai2025conceptmaps]: Systematic review of 28 studies on LLM concept maps.

### 2.2 Key Finding: No System Spans All Five Stages

| System | Stage 1 | Stage 2 | Stage 3 | Stage 4 | Stage 5 |
|--------|---------|---------|---------|---------|---------|
| MerLean | ✓ | — | ✓✓ | — | ✓ (back-translation) |
| MCP-SIM | — | — | — | ✓✓ | ✓✓ |
| CodePDE | — | — | — | ✓✓ | — |
| LLM-SR | — | ✓✓ | — | — | — |
| Learn Your Way | — | — | — | — | ✓✓ |

The closest to a full pipeline is MCP-SIM (Stages 4-5: simulation + explanation) and MerLean (Stages 1, 3, 5: extraction, verification, back-translation).

---

## 3. Thematic Analysis

### 3.1 Code as the Universal Bridge

The foundational insight from PAL [@gao2023pal] and Program of Thoughts [@chen2023pot] — that code is a more reliable reasoning medium than natural language for mathematical tasks — has matured into a full paradigm:

- **Discovery**: LLM-SR discovers scientific equations as Python programs
- **Verification**: SymCode converts math reasoning to auditable SymPy scripts
- **Implementation**: CodePDE generates PDE solvers competitive with expert code
- **Simulation**: MCP-SIM generates FEniCS simulations from natural language

The key technical innovation is **skeleton/parameter separation** (LLM-SR): LLMs handle equation structure while numerical optimizers handle coefficients. This division of labor exploits LLMs' strength (structure, patterns) while compensating for their weakness (precise numbers).

### 3.2 Autoformalization: From Math to Physics

The autoformalization field has undergone a domain expansion:

**2020-2023**: Pure mathematics (Szegedy, GPT-f, DSP, Wu+ NeurIPS 2022)
**2024-2025**: Applied mathematics (PDE-Controller, ICML 2025)
**2025-2026**: Physics (Lean4Physics, PhysLean, MerLean for quantum computation)

Critical finding from Lean4Physics: expert math provers (DeepSeek-Prover-V2: 14.5%) dramatically underperform general LLMs (Gemini-2.5-Pro: 39.5%) on physics tasks. **Physics formalization requires physical reasoning, not just mathematical skill.** Domain-specific infrastructure (PhysLib: +11.75%) matters more than model capability.

### 3.3 Multi-Agent Scientific Agents

The dominant architectural pattern for complex scientific tasks:

```
                    ┌─────────────────┐
                    │  Central Planner │ (LLM)
                    └────────┬────────┘
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
        ┌──────────┐  ┌──────────┐  ┌──────────┐
        │ Structure │  │  Solver  │  │  Critic  │
        │ Generator │  │ Generator│  │/Debugger │
        └──────────┘  └──────────┘  └──────────┘
```

Systems following this pattern: DREAMS (DFT), MCP-SIM (physics simulation), AgenticSciML (SciML), AutoNumerics (PDE), MerLean (formalization), ChemGraph (chemistry), Lang-PINN (PINNs), ATHENA (numerical algorithms).

Performance evidence:
- AgenticSciML: 4 orders of magnitude error reduction through agent collaboration
- MCP-SIM: 100% success vs. 50% for single-agent (12 physics tasks)
- CodePDE: Bug-free rate 41%→84% with self-debugging
- MerLean: 30-attempt compile-fix loop for Lean4 proofs

### 3.4 Interactive Knowledge Representations

Three validated approaches for making scientific knowledge navigable:

**1. Multi-representation generation** (Learn Your Way):
- 5 formats from 1 source (text, slides, audio, mind maps, quizzes)
- Personalized by grade level and interests
- RCT evidence: +9pp immediate, +11pp retention

**2. Nonlinear node+canvas interaction** (Mindalogue):
- 4-layer hierarchical mind map with Explain/Examples/Explore per node
- Outperforms linear LLM chat on convenience, controllability, cognitive load, reliability
- Statistically significant cognitive load reduction (p=0.033)

**3. Knowledge graphs with semantic search** (AutoMathKG):
- Definition/Theorem/Problem ontology with 9 tactic-labeled edge types
- SBERT-based vector database for natural language queries
- Auto-expanding from new sources (ProofWiki, arXiv, textbooks)

### 3.5 Scientific Coding Benchmarks: The Reality Check

| Benchmark | Domain | Best Score | Realistic? |
|-----------|--------|-----------|------------|
| SciCode [@tian2024scicode] | 16 subfields | 58.9% (2026) | Yes — curated by scientists |
| LLM-SRBench [@shojaee2025llmsrbench] | Equation discovery | 31.5% | Yes — anti-memorization design |
| UGPhysics [@xu2025ugphysics] | Undergrad physics | 49.8% | Yes — data-leakage screened |
| PHYBench [@qiu2025phybench] | Physics problems | 36.9% | Yes — original problems |
| Lean4Physics [@li2025lean4physics] | Physics formalization | 39.5% | Yes — hand-crafted |
| ASyMOB [@shalyt2025asymob] | Symbolic math | 70.3% drop on perturbations | Reveals memorization |

**Bottom line**: LLMs achieve impressive demo-level performance but struggle on realistic scientific tasks. The gap between SciCode's initial 4.6% (2024) and current 58.9% (2026) shows rapid improvement, but research-grade scientific coding remains largely unsolved.

### 3.6 Infrastructure: MCP as the Scientific Integration Layer

The Model Context Protocol is emerging as the standard for connecting LLM agents to scientific computing infrastructure:

- **LBNL/NERSC MCP servers** [@pan2025mcpscience]: Globus Transfer/Compute/Search, facility status APIs, Octopus event fabric, Garden model hosting
- **MATLAB MCP Server**: Official MathWorks integration with hardware access
- **Lean LSP MCP**: Lean4 language server for theorem proving agents (used by MerLean)
- **Rhea** (in MCP for Science): RAG-based dynamic tool discovery for scaling to thousands of scientific tools

Key architectural lesson: "Build thin MCP adapters for broad services rather than creating new ones" — reuse existing cyberinfrastructure rather than building parallel systems.

---

## 4. Cross-Domain Analysis

### 4.1 The Math↔Physics Transfer Problem

Multiple independent studies converge on the same finding:

1. **Mathematical reasoning does not transfer to physics** [@wu2025knowledge_reasoning]: SFT on math doesn't improve physics; RL is needed for cross-domain transfer
2. **Expert math provers fail on physics** (Lean4Physics): DeepSeek-Prover 14.5% vs. Gemini 39.5%
3. **Physics errors ≠ math errors** (PhysReason): Flawed reasoning and wrong principle application, not calculation errors
4. **Best physics score is 49.8%** (UGPhysics): Even o1-mini achieves only half on undergraduate physics

**Implication**: The user's vision of a Dirac equation→DFT pipeline cannot rely on mathematical manipulation alone. Physical understanding — symmetries, conservation laws, dimensional analysis, causal reasoning — must be explicitly encoded, either through domain-specific training, formal libraries (PhysLib), or multi-agent collaboration with physics-specific agents (DREAMS).

### 4.2 The Three Knowledge Formats

| Format | Best Tools | Maturity | Key Challenge |
|--------|-----------|----------|---------------|
| Natural Language → Math | SymCode, PDE-Controller | ★★★ | Ambiguity in NL specifications |
| Math → Formal Proof | MerLean, Lean4Physics | ★★ | Domain-specific infrastructure |
| Formal → Code | CodePDE, AutoNumerics | ★★★ | Self-debugging is essential |
| Code → Explanation | MCP-SIM, Learn Your Way | ★★★★ | Best-developed stage |
| All three simultaneously | None | ★ | The critical gap |

### 4.3 Verification Spectrum

The field spans from no verification to full formal proofs:

| Level | Method | Example | Trust Level |
|-------|--------|---------|-------------|
| 0 | None | Learn Your Way (content accuracy via rubric) | Low |
| 1 | Runtime assertions | SymCode (LLM-generated test cases) | Medium |
| 2 | Convergence testing | CodePDE (nRMSE on reference solutions) | Medium-High |
| 3 | Solver verification | PDE-Controller (Gurobi MILP) | High |
| 4 | Formal proof | MerLean/Lean4Physics (Lean4 type checker) | Very High |

**Trend**: Rapid progress at Level 4 (Dafny: 68%→96% in one year). Scientific verification likely to be practical within 2 years.

---

## 5. Open-Source Ecosystem & Cross-Repository Architecture

Based on deep source code analysis of 12 repositories (3,117 lines of analysis notes), we identify the practical landscape and key architectural patterns.

### 5.1 Repository Landscape

| Category | Repo | Stars | Language | Production-Ready? |
|----------|------|-------|----------|--------------------|
| **Equation Discovery** | PySR | 3.4K | Python/Julia | Yes — scikit-learn API, 5 export formats |
| | LLM-SR | 215 | Python | Prototype — no tests, no Docker |
| **Autoformalization** | PhysLean | 507 | Lean4 | Yes — 16 physics domains, dim. analysis as types |
| | MerLean | N/A | Claude+MCP | Not released — described in paper only |
| | lean-lsp-mcp | 312 | TypeScript | Yes — 21 MCP tools, REPL fast path |
| **Scientific Agents** | MCP-SIM | New | Python | Prototype — no tests; does NOT use MCP protocol |
| | DREAMS | New | Python | Prototype — LangGraph+Claude 3.7, 30 DFT tools |
| | Science MCPs | New | Python | Yes — 6 FastMCP servers for HPC |
| **Infrastructure** | Nougat | 9.9K | Python | Yes — 350M params, 76.5% math F1 |
| | Jupyter AI | 4.1K | Python/TS | Yes — 12+ LLM providers, `%%ai` magic |
| | SymPy (LaTeX) | 14.5K | Python | Yes — but ~80% parsing coverage, no physics notation |
| | MATLAB MCP | 224 | Go | Yes — 9 tools, session management |

**Clear divide**: Production tools (PySR, SymPy, Nougat, Jupyter AI, PhysLean) have tests, Docker, CI/CD, and stable APIs. Research prototypes (LLM-SR, MCP-SIM, DREAMS) have novel architectures but lack software engineering fundamentals.

### 5.2 Five Cross-Repository Architectural Patterns

**Pattern 1 — Equation Representation**: Four incompatible representations exist (Python functions, expression trees, symbolic objects, Lean4 propositions). No system converts between all four. PySR's 5-format export is the closest to a bridge.

**Pattern 2 — LLM Integration Spectrum**: Ranges from LLMs as "suggestion engines" in search loops (LLM-SR: minimal autonomy) to fully autonomous agents with tool access (MerLean: 30-iteration compile-fix). The trend is toward greater autonomy grounded by external verification.

**Pattern 3 — Self-Correction Loops**: Universal across all successful systems. Generate→Execute→Check→Fix→Repeat. LLM-SR uses 2,500 evolutionary generations; MerLean uses 30 compile attempts; CodePDE uses 4 debug rounds. No production system uses one-shot generation.

**Pattern 4 — State Management**: DREAMS' CANVAS pattern (persistent shared state backed by pickle, readable by all agents) enables plan-and-replan workflows and deserves wider adoption. Most other systems use session-level or stateless approaches.

**Pattern 5 — LaTeX Pipeline Bottleneck**: SymPy's ANTLR/Lark parser covers ~80% of standard math but fails on physics notation (Dirac, tensors, field theory). ASyMOB falls back to LLM for 18.5% of cases. SymPy Issue #26128 proposes LLM-based parsing but is blocked by training data.

### 5.3 Four Viable Integration Paths

Based on composability analysis of the codebase:

| Integration | Pipeline | Feasibility |
|------------|----------|-------------|
| **Equation Discovery→Verification** | LLM-SR → PySR (export to SymPy) → PhysLean (formal verify) | Medium |
| **Document→Simulation** | Nougat (PDF→LaTeX) → SymPy+LLM (parse) → CodePDE/MCP-SIM (solve) → Jupyter AI (present) | High |
| **Interactive Learning** | AutoMathKG (KG backend) → Mindalogue UI → Jupyter AI `%%ai -f math` → SymPy lambdify | Medium |
| **Science CLI via MCP** | Claude Code → lean-lsp-mcp + MATLAB MCP + Science MCPs + custom science KG MCP | High |

**Most impactful integration**: Science CLI via MCP (Integration 4) — all MCP infrastructure exists, and the custom "science knowledge MCP server" (equation library, derivation history, verification hooks) is the highest-leverage missing piece.

---

## 6. Gap Analysis and Future Directions

### 6.1 Critical Gaps

**Gap 1: No Unified Three-Format Bridge**
No system maintains semantic consistency across natural language ↔ LaTeX math ↔ numerical code simultaneously with round-trip capability. MerLean handles two formats (LaTeX↔Lean4); MCP-SIM handles three stages but not three formats.

**Gap 2: No Interactive Derivation Trees**
No system provides step-by-step navigable derivation trees where each node has NL explanation + LaTeX equation + runnable code. AutoMathKG provides the graph structure; Mindalogue provides the interaction paradigm; but no one combines them with executable math.

**Gap 3: No "Science Mode" for Coding CLIs**
Despite MCP's emergence as integration standard, no coding CLI offers domain-specific scientific features (equation libraries, derivation tracking, dimensional analysis, simulation launching).

**Gap 4: Cross-Domain Transfer Remains Unsolved**
Mathematical reasoning doesn't transfer to physics. Domain-specific infrastructure (PhysLib) helps more than better models. Fundamental advances in causal/physical reasoning may be needed.

**Gap 5: Scientific Code Verification**
Vericoding is rapidly improving for general code but untested on scientific computing. Lightweight domain-specific verification (unit checking, conservation laws, symmetry preservation) is needed.

### 6.2 Research Roadmap

**Near-term (1-2 years)**:
- MerLean + CodePDE integration: Formal spec → verified numerical solver
- MCP servers for major simulation codes (VASP, Gaussian, LAMMPS, FEniCS)
- Expand PhysLib to cover classical mechanics, electromagnetism, quantum mechanics
- Add explanation generation to MCP-SIM and CodePDE

**Medium-term (2-4 years)**:
- Three-format bridge system maintaining NL ↔ LaTeX ↔ Code consistency
- Interactive derivation tree interface integrated with Jupyter/VS Code
- Domain-specific coding CLI with equation libraries and verification hooks
- Vericoding for scientific computing (dimensional analysis, conservation laws)

**Long-term (4+ years)**:
- Autonomous scientific textbook generation from papers with verified derivations and runnable simulations
- Cross-domain reasoning systems with genuine physical understanding
- Living, self-updating knowledge structures covering undergraduate physics
- Community-maintained formal physics libraries at Mathlib scale

### 6.3 Recommended Architecture for the Vision

```
┌─────────────────────────────────────────────────────────────┐
│                    User Interface Layer                       │
│  ┌──────────┐  ┌──────────────┐  ┌────────────────────┐    │
│  │ CLI/IDE  │  │ Derivation   │  │ Interactive        │    │
│  │ Extension│  │ Tree Explorer│  │ Notebook (Jupyter) │    │
│  └────┬─────┘  └──────┬───────┘  └─────────┬──────────┘    │
│       └────────────────┼────────────────────┘               │
│                        ▼                                     │
│  ┌─────────────────────────────────────────┐                │
│  │    Knowledge Management Core             │                │
│  │  ┌─────────┐ ┌──────┐ ┌──────────────┐ │                │
│  │  │ NL ←──→ │ │LaTeX │ │ Code (Python)│ │                │
│  │  │ Engine  │ │Engine│ │ Engine       │ │                │
│  │  └─────────┘ └──────┘ └──────────────┘ │                │
│  │         Three-Format Bridge              │                │
│  └──────────────────┬──────────────────────┘                │
│                     ▼                                        │
│  ┌─────────────────────────────────────────┐                │
│  │    Knowledge Graph Backend               │                │
│  │  (AutoMathKG-style with physics ext.)    │                │
│  └──────────────────┬──────────────────────┘                │
│                     ▼                                        │
│  ┌──────────┐ ┌──────────┐ ┌───────────┐ ┌──────────────┐ │
│  │ Lean4    │ │ SymPy    │ │ FEniCS/   │ │ HPC via MCP  │ │
│  │ Verifier │ │ Symbolic │ │ Simulation│ │ (NERSC etc.) │ │
│  └──────────┘ └──────────┘ └───────────┘ └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## 7. Conclusion

The field of LLM-assisted scientific reasoning has advanced rapidly from foundational chain-of-thought prompting (2022) to systems that can formalize entire research papers into verified Lean4 code (MerLean, 2026) and generate 100%-successful physics simulations from natural language (MCP-SIM, 2026). The key paradigm — **code as the universal bridge between mathematical notation and computation** — is well-established and productive.

Our deep code analysis of 12 repositories reveals both the promise and the friction:

- **The self-correction loop is universal**: Every successful system (LLM-SR, MerLean, MCP-SIM, CodePDE, DREAMS) uses Generate→Execute→Check→Fix→Repeat. One-shot generation is insufficient for scientific computing.
- **Four incompatible equation representations** (Python functions, expression trees, symbolic objects, Lean4 propositions) fragment the ecosystem. PySR's 5-format export is the closest to a bridge, but no system spans all four.
- **The LaTeX parser is the weakest link**: SymPy handles ~80% of standard math but fails on physics notation (Dirac, tensors, field theory). LLM-augmented parsing is the right direction but lacks training data.
- **Production vs. prototype divide**: Mature tools (PySR, SymPy, Nougat, PhysLean) are ready for integration; research prototypes (LLM-SR, MCP-SIM, DREAMS) need engineering investment.
- **MCP is the integration standard**: lean-lsp-mcp (21 tools), MATLAB MCP (9 tools), and LBNL Science MCPs all converge on Model Context Protocol. A custom "science knowledge MCP server" — providing equation libraries, derivation histories, and verification hooks — is the highest-leverage contribution.

The vision of a unified system spanning all three knowledge formats (natural language ↔ mathematical notation ↔ numerical code) with interactive navigation (knowledge trees, derivation graphs) remains unrealized. The critical gaps are:

1. **Integration across the pipeline** — each stage works in isolation; four viable integration paths are identified but none yet implemented
2. **Cross-domain transfer** — math reasoning doesn't give you physics understanding; domain-specific infrastructure (PhysLib: +11.75%) matters more than model capability
3. **Verification** — rapidly improving (Dafny: 68%→96% in one year) but not yet applied to scientific computing
4. **Scale** — current knowledge structures cover thousands of entities; comprehensive physics requires millions

The most promising near-term path: build a Science CLI via MCP connecting existing production tools (lean-lsp-mcp for verification, PySR/SymPy for symbolic math, Nougat for document extraction, Jupyter AI for presentation) into a unified pipeline accessible from Codex/Claude Code, with DREAMS' CANVAS pattern for persistent shared state across agents.

The components exist. The integration is the frontier.

---

## References

See `references.bib` for full BibTeX entries. Key citations:

- [@wei2022cot] Chain-of-Thought Prompting, NeurIPS 2022
- [@gao2023pal] PAL: Program-aided Language Models, ICML 2023
- [@chen2023pot] Program of Thoughts, TMLR 2023
- [@shojaee2024llmsr] LLM-SR, ICLR 2025 Oral
- [@shojaee2025llmsrbench] LLM-SRBench, ICML 2025 Oral
- [@bagherinezhad2025symcode] SymCode, EACL 2026
- [@li2025codepde] CodePDE, TMLR
- [@soroco2025pdecontroller] PDE-Controller, ICML 2025
- [@ren2026merlean] MerLean, arXiv 2602.16554
- [@li2025lean4physics] Lean4Physics, ICLR 2026
- [@park2026mcpsim] MCP-SIM, npj AI 2026
- [@google2025learnyourway] Learn Your Way, arXiv 2509.13348
- [@zhang2024mindalogue] Mindalogue, CHI 2025
- [@bian2025automathkg] AutoMathKG, arXiv 2505.13406
- [@pan2025mcpscience] MCP for Science, arXiv 2508.18489
- [@bursuc2025vericoding] Vericoding Benchmark, arXiv 2509.22908
- [@tian2024scicode] SciCode, NeurIPS 2024
- [@romeraparedes2024funsearch] FunSearch, Nature 2024
- [@novikov2025alphaevolve] AlphaEvolve, arXiv 2506.13131
- [@wang2025dreams] DREAMS, arXiv 2507.14267
- [@toobysmith2025physlean] PhysLean, Comp.Phys.Comm. 2025
- [@blecher2023nougat] Nougat, ECCV 2024
- [@wu2025knowledge_reasoning] Knowledge or Reasoning?, arXiv 2506.02126
- [@xu2025ugphysics] UGPhysics, ICML 2025
