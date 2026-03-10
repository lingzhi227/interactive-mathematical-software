# Phase 5: Synthesis — LLM-Assisted Scientific Reasoning & Knowledge Tools

**Date**: 2026-03-10 | **Based on**: 82 papers (Phase 2), 12 deep-read papers (Phase 3), 12 repos deeply analyzed (Phase 4)

---

## 1. Taxonomy of Approaches

### 1.1 The Knowledge Transformation Pipeline

The research landscape maps onto a 5-stage pipeline for transforming scientific knowledge:

```
[Stage 1]          [Stage 2]           [Stage 3]          [Stage 4]           [Stage 5]
Document/Theory → Symbolic Math → Formal Verification → Executable Code → Human-Understandable
  (LaTeX, NL)      (SymPy, STL)     (Lean4, Dafny)      (Python, Julia)    (Trees, Graphs, NL)
     ↑                 ↑                  ↑                   ↑                    ↑
   Nougat           LLM-SR            MerLean             CodePDE            Learn Your Way
   Mathpix          SymCode         PDE-Controller        AutoNumerics        Mindalogue
   TeXpert          QuaSAR          Lean4Physics          MCP-SIM            AutoMathKG
                    ASyMOB          PhysLean              DREAMS             Concept Maps
```

**Key insight**: No single system spans all 5 stages. The user's vision of a unified pipeline (Dirac equation → LaTeX derivation → formal verification → numerical solver → interactive textbook) would require integrating tools from each stage.

### 1.2 Three Paradigms for Scientific Knowledge Transformation

| Paradigm | Approach | Representative Systems | Strengths | Weaknesses |
|----------|----------|----------------------|-----------|------------|
| **Code-as-Reasoning** | Equations as executable programs | LLM-SR, SymCode, PAL, PoT, CodePDE | Verifiable via execution, composable | Limited to what code can express |
| **Formal Verification** | Mathematical proofs as type-checked code | MerLean, Lean4Physics, PDE-Controller, Vericoding | Mathematical guarantees | Brittle, domain-specific infrastructure needed |
| **Knowledge Graphs** | Structured ontologies with semantic search | AutoMathKG, SciAgents, MindMap, InfraNodus | Navigable, cross-referencing | No execution or verification capability |

**The missing paradigm**: None combine all three. A complete system would represent knowledge as both executable code AND formally verified statements AND navigable knowledge graphs.

### 1.3 Architectural Patterns

| Pattern | Frequency | Examples |
|---------|-----------|---------|
| Multi-agent with central planner | 8 papers | DREAMS, MCP-SIM, AgenticSciML, AutoNumerics, MerLean, ChemGraph, Lang-PINN, ATHENA |
| Evolutionary search + LLM | 4 papers | LLM-SR, FunSearch, AlphaEvolve, PiT-PO |
| Generate-debug-refine loop | 6 papers | CodePDE, SymCode, MerLean, PDE-SHARP, AutoNumerics, MCP-SIM |
| Knowledge graph + LLM | 4 papers | AutoMathKG, SciAgents, MindMap, LinkQ |
| Multi-representation generation | 3 papers | Learn Your Way, Mindalogue, Concept Maps |

**Dominant pattern**: Multi-agent + generate-debug-refine. Nearly all successful systems use iterative refinement with external verification (compiler, solver, simulator).

---

## 2. Comparative Analysis Across 12 Deep-Read Papers

### 2.1 Technical Capability Matrix

| Paper | NL→Math | Math→Code | Code→Sim | Sim→Explain | Formal Verify | Knowledge Structure |
|-------|---------|-----------|----------|-------------|---------------|-------------------|
| LLM-SR | ✓ | ✓✓ | — | — | — | — |
| SymCode | — | ✓✓ | — | — | ✓ (runtime) | — |
| CodePDE | ✓ | ✓✓ | ✓✓ | — | ✓ (convergence) | — |
| PDE-Controller | ✓✓ | ✓ | ✓ | — | ✓ (Gurobi) | — |
| MerLean | ✓✓ | — | — | ✓ (LaTeX back-translation) | ✓✓ (Lean4) | — |
| Lean4Physics | — | — | — | — | ✓✓ (Lean4) | ✓ (PhysLib) |
| Learn Your Way | — | — | — | ✓✓ | — | ✓✓ (mind maps) |
| Mindalogue | — | — | — | ✓✓ | — | ✓✓ (node+canvas) |
| AutoMathKG | — | — | — | — | — | ✓✓ (KG) |
| MCP-SIM | ✓✓ | ✓ | ✓✓ | ✓✓ | ✓ (convergence) | — |
| MCP for Science | — | — | ✓ | — | — | — |
| Vericoding | — | ✓ | — | — | ✓✓ (formal) | — |

**No paper covers all columns.** The closest to spanning the pipeline are MCP-SIM (NL→code→sim→explain) and MerLean (NL→formal→back-translate).

### 2.2 Performance Benchmarks

| Task | Best System | Score | Human Baseline | Gap |
|------|------------|-------|----------------|-----|
| Research-grade scientific coding | Gemini 3.1 Pro | 58.9% (SciCode) | ~100% expert | Large |
| Scientific equation discovery | Best system | 31.5% (LLM-SRBench) | ~100% expert | Very large |
| Undergraduate physics | o1-mini | 49.8% (UGPhysics) | ~80% student | Large |
| Physics reasoning | DeepSeek-R1 | <60% (PhysReason) | ~80% expert | Large |
| PDE solver generation | Claude 3.5 | Expert-competitive (CodePDE) | Expert | Closing |
| Autoformalization (math) | Claude Opus 4 | 67/312 correct (IndiMathBench) | N/A | Large |
| Autoformalization (physics) | Gemini-2.5-Pro | 39.5% (Lean4Physics) | N/A | Very large |
| Verified code synthesis | Model union | 96% (Dafny), 27% (Lean) | ~100% | Closing (Dafny) |
| AI textbook learning gains | Learn Your Way | +11% retention | Traditional textbook | Significant improvement |

### 2.3 Key Technical Insights from Deep Reading

1. **Skeleton/parameter separation** (LLM-SR): LLMs handle equation structure, numerical optimizers handle coefficients. This division of labor is fundamental — LLMs are good at structure, bad at precise numbers.

2. **Code-as-verification** (SymCode): Converting math reasoning to SymPy code shifts errors from opaque logical fallacies to debuggable programming errors. 60-77% token reduction as a bonus.

3. **Self-debugging is essential** (CodePDE): Bug-free rate improves from 41% to 84% with 4 rounds of debugging. No production system should use one-shot generation.

4. **Infrastructure > models** (Lean4Physics): PhysLib provides +11.75% improvement — more than switching between frontier models. Building domain-specific libraries matters more than waiting for better LLMs.

5. **Bidirectional translation** (MerLean): Lean→LaTeX "autoinformalization" is as important as LaTeX→Lean formalization. Scientists need to verify in their native notation.

6. **Plan-Act-Reflect-Revise** (MCP-SIM): Iterative control cycle emulating expert reasoning achieves 100% success where one-shot generation achieves 50%.

7. **Multi-representation learning works** (Learn Your Way): 5 formats from 1 source, with +11% retention in RCT. Mind maps as hierarchical knowledge navigation are validated.

8. **Nonlinear > linear** (Mindalogue): Nodes+canvas interface outperforms linear chat on 4/5 dimensions. Knowledge trees are not just nice — they're measurably better.

---

## 3. Cross-Repository Code Analysis (Phase 4 Deep Dive)

Based on deep source code analysis of 12 repositories (3,117 lines of analysis notes), we identify five cross-cutting architectural patterns and four viable integration paths.

### 3.1 Five Architectural Patterns

**Pattern 1: Equation Representation Spectrum**

| System | Representation | Composability | Verifiability |
|--------|---------------|---------------|---------------|
| LLM-SR | Python function bodies | Flexible — any math | Hard to analyze symbolically |
| PySR | Expression trees (Julia) | Exportable to 5 formats (SymPy, LaTeX, JAX, PyTorch, NumPy) | Fixed operator vocabulary |
| SymPy | Symbolic expression objects | Full CAS capabilities | Limited LaTeX parsing (~80%) |
| PhysLean | Lean4 propositions with types | Formally verified + dimensional analysis as types | Requires Lean expertise |

**Critical gap**: No system seamlessly converts between all four representations. A unified representation layer would enable: LLM-SR discovers → PySR verifies with expression tree → SymPy manipulates → PhysLean formally verifies.

**Pattern 2: LLM Integration Spectrum**

| System | LLM Role | Autonomy |
|--------|----------|----------|
| LLM-SR | "Suggestion engine" in search loop | Minimal — LLM proposes, optimizer evaluates |
| MCP-SIM | 6 specialized agents | Medium — agents follow linear pipeline |
| DREAMS | 3 hierarchical agents (LangGraph) | High — supervisor inspects and replans |
| MerLean | Full reasoning agent (Claude + MCP) | Maximum — 30-iteration autonomous compile-fix |

**Key contrast**: LLM-SR treats LLMs as fast hypothesis generators within an evolutionary loop. MerLean/DREAMS treat LLMs as autonomous agents with tool access. Jupyter AI is purely user-driven. The trend is toward greater autonomy with tool-grounded verification.

**Pattern 3: Self-Correction Loops (Universal)**

| System | Verification Oracle | Max Iterations | Improvement |
|--------|-------------------|----------------|-------------|
| LLM-SR | Fitness on held-out data | 2,500 generations | 100-1000x NMSE reduction |
| MerLean | Lean4 type checker | 30 compile attempts | Formalizes 114/114 statements |
| MCP-SIM | FEniCS runtime | <5 iterations | 50%→100% success rate |
| DREAMS | DFT convergence | Plan-replan cycles | <1% error vs. experts |
| CodePDE | nRMSE + convergence | 4 debug rounds | 41%→84% bug-free |

**Universal pattern**: Generate → Execute → Check → Fix → Repeat. No production system uses one-shot generation.

**Pattern 4: State Management in Multi-Agent Systems**

| System | Mechanism | Key Design |
|--------|-----------|------------|
| MCP-SIM | Shared memory dict | Session-level, linear pipeline |
| DREAMS | CANVAS (pickle-backed) | Persistent across tasks, supervisor can inspect/replan |
| MerLean | MCP tool context | Per-statement, stateless between declarations |
| Science MCPs | Stateless (tool-level) | Per-call, no session memory |

**DREAMS' CANVAS pattern deserves adoption**: A persistent shared state object that all agents read/write, backed by pickle serialization, enabling plan-and-replan workflows.

**Pattern 5: The LaTeX Pipeline Bottleneck**

```
PDF ──Nougat──→ LaTeX ──SymPy parser──→ Symbolic ──codegen──→ NumPy/Code
     (76.5%)       (ANTLR/Lark)         (SymPy)       (lambdify)
                   Limitations:
                   - No \begin{cases}
                   - No custom macros
                   - 18.5% fallback to LLM (ASyMOB)
                   - No physics-aware parsing (Dirac, tensor, field theory)
```

SymPy Issue #26128 proposes LLM-based parsing but is blocked by lack of training data. Nougat + LLM-augmented SymPy + PhysLean dimensional checking could form a robust pipeline.

### 3.2 Implementation Quality Divide

| Category | Production-Ready | Research Prototypes |
|----------|-----------------|-------------------|
| **Tools** | PySR (3.4K★), Nougat (9.9K★), SymPy (14.5K★), Jupyter AI (4.1K★), PhysLean (507★), lean-lsp-mcp (312★), MATLAB MCP (224★) | LLM-SR (215★), MCP-SIM, DREAMS |
| **Indicators** | Tests ✓, Docker ✓, CI/CD ✓, stable API | No tests, no Docker, no dependency management |
| **Implication** | Ready to integrate via MCP | Need engineering investment before integration |

**Notable finding**: MCP-SIM does NOT actually use the Model Context Protocol despite its name — it uses LangChain agents with a linear pipeline. Only lean-lsp-mcp, MATLAB MCP, and the LBNL Science MCPs implement actual MCP servers.

### 3.3 Four Viable Integration Paths

**Integration 1: Equation Discovery → Verification** (Feasibility: Medium)
```
LLM-SR (discover equation as Python) → PySR (export to SymPy expression)
  → PhysLean (formally verify with dimensional analysis)
```
PySR already exports to SymPy; MerLean handles SymPy→Lean4 translation.

**Integration 2: Document → Simulation** (Feasibility: High)
```
Nougat (PDF → LaTeX) → SymPy parser + LLM fallback (LaTeX → symbolic)
  → CodePDE/MCP-SIM (symbolic → solver) → Jupyter AI (present in notebook)
```
Each component exists independently; the glue code is straightforward.

**Integration 3: Interactive Learning Pipeline** (Feasibility: Medium)
```
AutoMathKG (knowledge graph backend) → Mindalogue-style UI (nonlinear)
  → Jupyter AI %%ai -f math (inline equations) → SymPy lambdify (run code)
```
UI integration is the main challenge.

**Integration 4: Science CLI via MCP** (Feasibility: High)
```
Claude Code / Codex CLI
  → lean-lsp-mcp (21 tools for Lean4 verification)
  → MATLAB MCP (9 tools for simulation)
  → Science MCPs (Globus/NERSC/ALCF access)
  → Custom "science knowledge" MCP (equation library, derivation history)
```
MCP is the right integration layer. The custom science knowledge MCP is the missing piece.

---

## 4. Cross-Domain Analysis

### 4.1 Math vs. Physics: The Transfer Problem

A critical finding across multiple papers:

- **UGPhysics**: Best LLM achieves 49.8% on undergrad physics (vs. much higher on math)
- **Lean4Physics**: Expert math provers (DeepSeek-Prover-V2: 14.5%) dramatically underperform general LLMs (Gemini: 39.5%) on physics tasks
- **Knowledge or Reasoning** (Wu+ 2025): Mathematical reasoning does NOT naturally transfer to physics via SFT
- **PhysReason**: Physics errors are primarily flawed reasoning and wrong principle application, not calculation errors

**Implication for the user's vision**: The Dirac equation example requires not just mathematical manipulation but physical intuition (spinor fields, gauge symmetry, relativistic invariance). A system bridging math↔physics must incorporate domain-specific knowledge, not just mathematical skill.

### 4.2 The Three Knowledge Formats

The user identified three knowledge formats: natural language, mathematical notation (LaTeX), and numerical code. Our survey reveals these map to concrete systems:

| Format | Representation | Tools | Strengths | Weaknesses |
|--------|---------------|-------|-----------|------------|
| **Natural Language** | Text, explanations, tutorials | Learn Your Way, Mindalogue, MCP-SIM reports | Accessible, personalized | Ambiguous, not executable |
| **Mathematical Notation** | LaTeX, Lean4, SymPy symbolic | MerLean, Lean4Physics, SymCode, AutoMathKG | Precise, verifiable | High learning curve, static |
| **Numerical Code** | Python, Julia, FEniCS | CodePDE, DREAMS, AutoNumerics, MCP-SIM | Executable, testable | Implementation details obscure theory |

**The gap**: No system seamlessly connects all three. MerLean goes LaTeX→Lean4→LaTeX (two formats). MCP-SIM goes NL→Code→Explanation (two formats). LLM-SR goes Data→Code (one format). The three-format bridge is the key unsolved problem.

### 4.3 The Verification Spectrum

```
No verification ←————————————————————————————————→ Full formal verification
     |              |               |                    |
   LearnLM     SymCode         CodePDE/MCP-SIM       MerLean/Vericoding
  (no check)  (runtime assert)  (convergence test)   (Lean4 type check)
```

Current systems cluster at the extremes — either no verification (educational tools) or full formal verification (autoformalization). The middle ground (lightweight verification for scientific code) is underexplored.

---

## 5. Mapping to the User's Vision

### 5.1 The User's Proposed System

The user envisions:
1. GPT-level LLM transforms complex equations into knowledge trees, graphs, natural language
2. A Codex CLI extension transforms knowledge into code format
3. Three knowledge formats: natural language ↔ LaTeX math ↔ numerical code
4. Interactive learning of math/physics via IDE tools
5. Example: Dirac equation → derivation → materials simulation (DFT)

### 5.2 What Exists Today (from our survey)

| Component | Available Today | Gaps |
|-----------|----------------|------|
| Equation→knowledge tree | AutoMathKG (math only), Learn Your Way (mind maps) | No physics equations, no derivation trees |
| Equation→natural language | MCP-SIM (explains simulations), Learn Your Way (personalizes text) | Not for arbitrary equations |
| LaTeX↔code translation | SymCode (LaTeX→SymPy), LLM-SR (equations as programs) | No round-trip, no derivation preservation |
| Codex CLI for science | MCP for Science (infrastructure), Jupyter AI (notebooks) | No unified "science mode" for coding CLIs |
| Interactive learning | Mindalogue (nonlinear), Learn Your Way (multi-format) | Not integrated with code execution |
| Dirac equation→DFT | DREAMS (DFT agent), Aitomia (quantum chem) | Manual, not from theory derivation |
| Formal verification | MerLean (quantum), Lean4Physics (physics) | Not integrated with code gen or learning |
| Knowledge structure management | AutoMathKG, SciAgents (KG+agents) | Not cross-domain math↔physics |

### 5.3 Architecture for the Vision

Based on our survey, the user's vision would require integrating:

```
Layer 1: Knowledge Extraction
  ├── Nougat/Mathpix: Document → LaTeX
  ├── AutoMathKG-style: LaTeX → Knowledge Graph
  └── LLM: LaTeX → Natural Language Explanation

Layer 2: Knowledge Formalization
  ├── SymCode: LaTeX → SymPy symbolic
  ├── MerLean-style: LaTeX → Lean4 formal
  └── PDE-Controller-style: NL → Formal spec

Layer 3: Knowledge Operationalization
  ├── CodePDE/AutoNumerics: Formal spec → Numerical solver
  ├── DREAMS-style: Theory → DFT calculation
  └── MCP-SIM-style: NL → Simulation + Explanation

Layer 4: Knowledge Presentation
  ├── Learn Your Way: Multi-representation generation
  ├── Mindalogue: Nonlinear interactive exploration
  └── Knowledge Tree/Graph visualization

Layer 5: Infrastructure
  ├── MCP servers: Connect to HPC/simulation tools
  ├── Jupyter/IDE integration: Notebook + code environment
  └── Vericoding: Formal verification of generated code
```

---

## 6. Maturity Assessment

| Capability | Maturity (1-5) | Evidence |
|-----------|---------------|---------|
| LLM math reasoning | ⬤⬤⬤⬤○ (4) | CoT, SymCode, Minerva — strong and improving |
| Equation→code | ⬤⬤⬤○○ (3) | LLM-SR, CodePDE — works but 31.5% accuracy ceiling |
| Autoformalization (math) | ⬤⬤⬤○○ (3) | MerLean formalizes full papers, but 27% Lean accuracy |
| Autoformalization (physics) | ⬤⬤○○○ (2) | Lean4Physics best=39.5%, infrastructure nascent |
| Knowledge graphs for science | ⬤⬤⬤○○ (3) | AutoMathKG, SciAgents — functional but small scale |
| AI textbooks | ⬤⬤⬤⬤○ (4) | Learn Your Way RCT shows +11% retention |
| Interactive knowledge exploration | ⬤⬤⬤○○ (3) | Mindalogue validated at CHI, but limited scale |
| Scientific coding agents | ⬤⬤⬤○○ (3) | DREAMS, MCP-SIM — 100% success but narrow domains |
| MCP for science | ⬤⬤○○○ (2) | LBNL proof of concept, MATLAB server — early stage |
| Verified scientific code | ⬤⬤○○○ (2) | Dafny 96% but scientific domains untested |
| Cross-domain math↔physics | ⬤○○○○ (1) | Transfer doesn't work; active research gap |
| Unified pipeline (all 5 stages) | ⬤○○○○ (1) | Does not exist yet |

---

## 7. Research Frontiers and Opportunities

### 7.1 Near-Term (1-2 years)
- **Integrate MerLean + CodePDE**: Formal specification → verified numerical solver pipeline
- **Add explanation layer to MCP-SIM**: Already generates simulation reports; extend to knowledge trees
- **Build physics PhysLib extensions**: More Lean4 libraries for electromagnetism, QM, statistical mechanics
- **MCP servers for scientific tools**: Extend LBNL's work to more simulation codes (VASP, Gaussian, LAMMPS)

### 7.2 Medium-Term (2-4 years)
- **Unified three-format bridge**: System that maintains consistency across NL ↔ LaTeX ↔ Code
- **Interactive derivation trees**: Hierarchical, navigable representations of multi-step derivations (e.g., from Dirac equation to band structure)
- **Domain-specific coding CLI**: Extension of Codex/Claude Code with scientific knowledge management (equation libraries, derivation histories, simulation templates)
- **Vericoding for scientific code**: Formal verification of LLM-generated PDE solvers, DFT calculations

### 7.3 Long-Term (4+ years)
- **Autonomous scientific textbook generation**: From papers → knowledge graph → interactive multi-format textbook with verified derivations and runnable simulations
- **Cross-domain reasoning**: Systems that genuinely understand physics (not just manipulate math) — requires fundamental advances in causal reasoning
- **Living knowledge structures**: Self-updating knowledge graphs that incorporate new papers and maintain formal consistency
