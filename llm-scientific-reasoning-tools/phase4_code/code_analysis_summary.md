# Phase 4: Code Analysis Summary — Cross-Repository Architectural Insights

**Date**: 2026-03-10 | **Repos analyzed**: 12 | **Total analysis**: 3,117 lines

---

## Repositories Analyzed

| Category | Repo | Stars | Language | Key Insight |
|----------|------|-------|----------|-------------|
| **Equation Discovery** | LLM-SR | ~215 | Python | Equations as Python function bodies + evolutionary islands |
| | PySR | 3.4K | Python/Julia | Expression trees + tournament selection, scikit-learn API |
| **Autoformalization** | PhysLean | 507 | Lean4 | 16 physics domains, dimensional analysis as type system |
| | MerLean | N/A | Claude+MCP | Fully automated LaTeX→Lean4→LaTeX, 30-iteration compile-fix |
| | lean-lsp-mcp | 312 | TypeScript | 21 MCP tools for Lean4, REPL fast path |
| **Scientific Agents** | MCP-SIM | New | Python | 6-agent linear pipeline for FEniCS, GPT-4o |
| | DREAMS | New | Python | LangGraph + Claude 3.7, 30 DFT tools, shared CANVAS state |
| | Science MCPs | New | Python | 6 FastMCP servers for Globus/NERSC/ALCF |
| **Infrastructure** | Nougat | 9.9K | Python | Swin Transformer + mBART, 76.5% math F1, 350M params |
| | Jupyter AI | 4.1K | Python/TS | 12+ LLM providers, `%%ai` magic, `-f math` for LaTeX |
| | SymPy (LaTeX) | 14.5K | Python | ANTLR/Lark parsers, no LLM integration yet |
| | MATLAB MCP | 224 | Go | 9 tools, session management, hardware access |

---

## Cross-Repository Architectural Patterns

### Pattern 1: Equation Representation

| System | Representation | Pros | Cons |
|--------|---------------|------|------|
| LLM-SR | Python function bodies | Flexible, any math possible | Hard to analyze symbolically |
| PySR | Expression trees (Julia) | Composable, exportable to 5 formats | Fixed operator vocabulary |
| SymPy | Symbolic expression objects | Full CAS capabilities | Limited LaTeX parsing |
| PhysLean | Lean4 propositions with types | Formally verified, dimensional analysis | Requires Lean expertise |

**Gap**: No system seamlessly converts between all four representations. A unified equation representation layer would enable: LLM-SR discovers equation → PySR verifies with expression tree → SymPy performs symbolic manipulation → PhysLean formally verifies.

### Pattern 2: LLM Integration Approaches

| System | LLM Role | Integration Method | Models |
|--------|----------|-------------------|--------|
| LLM-SR | Hypothesis generator | Direct API call with dynamic prompts | Mixtral, GPT-3.5/4 |
| MerLean | Full reasoning agent | Claude Code + MCP tools | Claude Opus 4.5 |
| MCP-SIM | 6 specialized agents | LangChain with GPT-4o | GPT-4o |
| DREAMS | 3 hierarchical agents | LangGraph with Claude 3.7 | Claude 3.7 Sonnet |
| Jupyter AI | User-facing assistant | Provider abstraction layer | 12+ providers |

**Key contrast**: LLM-SR uses LLMs as "suggestion engines" within a search loop (minimal autonomy). MerLean/DREAMS use LLMs as autonomous agents with tool access (maximum autonomy). Jupyter AI is purely interactive (user-driven).

### Pattern 3: Self-Correction Loops

All successful systems use iterative refinement:

| System | Verification Method | Max Iterations | Success Improvement |
|--------|-------------------|----------------|-------------------|
| LLM-SR | Fitness on held-out data | 2,500 generations | 100-1000x NMSE reduction |
| MerLean | Lean4 type checker | 30 compile attempts | Eventually formalizes 114/114 |
| MCP-SIM | FEniCS runtime | <5 iterations | 50%→100% success rate |
| DREAMS | DFT convergence | Plan-replan cycles | <1% error vs experts |
| CodePDE | nRMSE + convergence | 4 debug rounds | 41%→84% bug-free |

**Universal pattern**: Generate → Execute → Check → Fix → Repeat. No system uses one-shot generation in production.

### Pattern 4: State Management in Multi-Agent Systems

| System | State Mechanism | Scope |
|--------|----------------|-------|
| MCP-SIM | Shared memory dict | Session-level |
| DREAMS | CANVAS (pickle-backed) | Persistent across tasks |
| MerLean | MCP tool context | Per-statement |
| Science MCPs | Stateless (tool-level) | Per-call |

**DREAMS' CANVAS pattern** is notable: a persistent shared state object that all agents read/write, backed by pickle serialization. This enables plan-and-replan: the supervisor can inspect intermediate DFT results and adjust the strategy.

### Pattern 5: The LaTeX Pipeline Gap

Current state of the LaTeX→Code pipeline:

```
PDF ──Nougat──→ LaTeX ──SymPy parser──→ Symbolic ──codegen──→ NumPy/Code
     (76.5%)        (ANTLR/Lark)        (SymPy)       (lambdify)
                    Limitations:
                    - No \begin{cases}
                    - No custom macros
                    - 18.5% fallback to LLM
                    - No physics-aware parsing
```

**Critical gap**: SymPy's LaTeX parser handles ~80% of standard math but fails on physics notation (Dirac notation, tensor indices, field theory conventions). Issue #26128 proposes LLM-based parsing but is blocked by lack of training data.

**Opportunity**: Nougat (76.5% equation F1) + LLM-augmented SymPy parsing + PhysLean dimensional checking could form a robust pipeline.

---

## Implementation Quality Assessment

| Repo | Tests | Docker | Docs | CI/CD | Prod-Ready? |
|------|-------|--------|------|-------|-------------|
| PySR | ✓✓ | ✓ | ✓✓ | ✓ | Yes |
| Nougat | ✓ | ✓ | ✓ | ✓ | Yes |
| SymPy | ✓✓ | — | ✓✓ | ✓ | Yes |
| Jupyter AI | ✓✓ | — | ✓✓ | ✓ | Yes |
| PhysLean | ✓ | — | ✓✓ | ✓ | Yes (for Lean users) |
| lean-lsp-mcp | ✓ | — | ✓ | — | Yes |
| Science MCPs | ✓ | ✓ | ✓ | ✓ | Yes |
| MATLAB MCP | ✓ | — | ✓ | ✓ | Yes |
| LLM-SR | — | — | ✓ | — | Research prototype |
| MCP-SIM | — | — | ○ | — | Research prototype |
| DREAMS | — | — | ✓ | — | Research prototype |
| MerLean | N/A | N/A | N/A | N/A | Not released |

**Clear divide**: Production tools (PySR, SymPy, Nougat, Jupyter AI) vs. research prototypes (LLM-SR, MCP-SIM, DREAMS). The scientific agent repos lack tests, Docker, and proper dependency management.

---

## Composability Analysis: What Can Be Combined?

### Viable Integration 1: Equation Discovery → Verification
```
LLM-SR (discover equation as Python)
  → PySR (export to SymPy expression)
    → PhysLean (formally verify with dimensional analysis)
```
**Feasibility**: Medium. PySR already exports to SymPy. SymPy→Lean4 translation is MerLean's core capability.

### Viable Integration 2: Document → Simulation
```
Nougat (PDF → LaTeX)
  → SymPy parser + LLM fallback (LaTeX → symbolic)
    → CodePDE/MCP-SIM (symbolic → numerical solver)
      → Jupyter AI (present results in notebook)
```
**Feasibility**: High. Each component exists independently. The glue code is straightforward.

### Viable Integration 3: Interactive Learning Pipeline
```
AutoMathKG (knowledge graph backend)
  → Mindalogue-style UI (nonlinear exploration)
    → Jupyter AI `%%ai -f math` (inline equation explanation)
      → SymPy lambdify (run the equation)
```
**Feasibility**: Medium. AutoMathKG and Jupyter AI exist; the UI integration is the challenge.

### Viable Integration 4: Science CLI via MCP
```
Claude Code / Codex CLI
  → lean-lsp-mcp (verify equations in Lean4)
  → MATLAB MCP (run simulations)
  → Science MCPs (access HPC, transfer data)
  → Custom MCP (equation library, derivation history)
```
**Feasibility**: High. MCP is the right integration layer. The custom "science knowledge" MCP server is the missing piece.

---

## Key Takeaways

1. **MCP is the integration standard**: lean-lsp-mcp (Lean4), MATLAB MCP (MATLAB), Science MCPs (HPC) all converge on Model Context Protocol. Building a "science knowledge MCP" is the highest-leverage contribution.

2. **Research prototypes need engineering**: LLM-SR, MCP-SIM, DREAMS have novel architectures but lack tests, Docker, and API stability. Productionizing any of these would be high-impact.

3. **The LaTeX parser is the weakest link**: SymPy's ANTLR-based parser covers ~80% of standard math but fails on physics notation. LLM-augmented parsing (Issue #26128) is the right direction.

4. **PhysLean is the most mature formal physics infrastructure**: 16 domains, dimensional analysis as types, community contributions. Building on PhysLean rather than starting fresh is the pragmatic choice.

5. **DREAMS' CANVAS pattern deserves adoption**: Persistent shared state across agents is simple but powerful for multi-step scientific workflows.
