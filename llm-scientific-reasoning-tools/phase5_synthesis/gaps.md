# Phase 5: Gap Analysis — What's Missing for the User's Vision

## The Vision Recap
Build a system where users can interactively learn math/physics through a coding assistant (Codex/Claude Code CLI) that manages three knowledge formats: natural language ↔ LaTeX math ↔ numerical code. Example: Dirac equation derivation → materials simulation code.

---

## Gap 1: No Unified Three-Format Bridge
**Status**: Critical gap — no existing system
**What exists**:
- MerLean: LaTeX ↔ Lean4 (2 formats)
- MCP-SIM: NL → Code → Explanation (3 formats, but not round-trip)
- SymCode: Math problem → SymPy code (1 direction)
**What's needed**: A system maintaining semantic consistency across all three representations simultaneously, with round-trip capability (change the code → update the LaTeX → update the explanation).
**Research pointers**: PAL/PoT showed code-as-reasoning; MerLean showed bidirectional translation is possible; no one has done three-way yet.

## Gap 2: No Interactive Derivation Trees for Physics
**Status**: Critical gap — partially addressed
**What exists**:
- AutoMathKG: Knowledge graph for math (definitions, theorems, problems)
- Learn Your Way: Mind maps from textbook chapters
- Mindalogue: Nonlinear node+canvas interaction
**What's needed**: Step-by-step derivation trees where each node is a derivation step, with the ability to zoom into any step for explanation, see the LaTeX, and run the corresponding code. Example: Dirac equation → non-relativistic limit → Schrödinger equation → finite element discretization → Python solver.
**Research pointers**: Tree of Thoughts provides the tree-structured reasoning; AutoMathKG provides the ontology (Definition/Theorem/Problem); Lean4Physics provides formal verification of steps.

## Gap 3: No "Science Mode" for Coding CLIs
**Status**: Major gap — infrastructure exists but not integrated
**What exists**:
- MCP for Science (LBNL): MCP servers connecting to HPC
- MATLAB MCP Server: MATLAB integration
- Codex CLI / Claude Code: General-purpose coding agents
- Jupyter AI: LLM in notebooks
**What's needed**: A CLI extension or MCP server that provides:
  1. Equation library: Browse and search mathematical equations by physics domain
  2. Derivation history: Track which equations derive from which
  3. Code template library: Standard numerical methods (FEM, spectral, Monte Carlo) with equations annotated
  4. Verification hooks: Auto-check units, dimensional analysis, conservation laws
  5. Simulation launcher: Submit HPC jobs from the CLI with results streamed back
**Research pointers**: MCP protocol is the right integration layer; DREAMS shows the multi-agent pattern; InfraNodus shows knowledge graph in IDE.

## Gap 4: Cross-Domain Transfer Math↔Physics
**Status**: Fundamental research gap
**What exists**:
- UGPhysics: Shows best LLM scores 49.8% on undergrad physics
- Knowledge or Reasoning (Wu+ 2025): Math reasoning doesn't transfer to physics
- PhysReason: Identifies 4 bottlenecks in physics reasoning
**What's needed**: LLMs that understand physics, not just manipulate math. This requires:
  1. Physical intuition (symmetry, conservation, causality)
  2. Dimensional analysis as first-class reasoning
  3. Connection between formalism and physical meaning
  4. Multi-scale reasoning (quantum → classical → continuum)
**Research pointers**: Lean4Physics's PhysLib approach (domain-specific infrastructure) may be more productive than waiting for better general models. Physics simulation feedback (like MCP-SIM's Plan-Act-Reflect-Revise) can ground abstract reasoning.

## Gap 5: Verification of Scientific Code
**Status**: Rapidly closing but not yet applied to science
**What exists**:
- Vericoding Benchmark: Dafny 96%, Lean 27%
- Astrogator: 83% verification of LLM code
- CodePDE: Convergence testing as verification
- MerLean: Full Lean4 proofs
**What's needed**: Lightweight verification specifically for scientific code:
  1. Unit/dimensional consistency checking
  2. Conservation law verification
  3. Convergence order verification
  4. Comparison with analytical limits
  5. Symmetry preservation testing
**Research pointers**: Lean4Physics's PhysLib has unit systems; CodePDE's convergence testing is a start; domain-specific formal methods need development.

## Gap 6: Scale of Knowledge Structures
**Status**: Significant gap
**What exists**:
- AutoMathKG: 13,388 entities (small)
- Learn Your Way: Single chapter at a time
- PhysLean: Handful of physics results formalized
**What's needed**: Comprehensive knowledge structures covering major physics domains:
  1. Classical mechanics → quantum mechanics → quantum field theory
  2. Thermodynamics → statistical mechanics → kinetic theory
  3. Electromagnetism → optics → plasma physics
  4. Solid-state physics → materials science → DFT
**Scale required**: ~100K entities with ~1M edges for a comprehensive undergraduate physics coverage

## Gap 7: Pedagogical Adaptation for STEM
**Status**: Partially addressed
**What exists**:
- Learn Your Way: Grade-level + interest personalization (+11% retention)
- KMP-Bench: Shows solving ≠ teaching
- PedagogicalRL-Thinking: Theory-grounded pedagogical prompting
**What's needed**: Adaptive scientific tutoring that models:
  1. Learner's current mathematical background
  2. Which derivation steps are understood vs. need explanation
  3. Which analogies/examples best connect to learner's domain
  4. When to show math notation vs. code vs. natural language
**Research pointers**: Inquizzitor's adaptive scaffolding + AutoMathKG's knowledge graph could form the basis.

## Gap 8: Living, Self-Updating Knowledge Systems
**Status**: Not yet addressed
**What exists**:
- AutoMathKG: Automatic expansion from new sources
- SciAgents: KG + LLM for hypothesis generation
**What's needed**: Knowledge structures that:
  1. Auto-incorporate new papers (via Nougat → extraction → integration)
  2. Maintain formal consistency when updated
  3. Flag when new results contradict existing knowledge
  4. Track provenance (which paper introduced which equation)
**Research pointers**: AutoMathKG's fusion mechanism is a start; formal verification (Lean4) could ensure consistency.

---

## Priority Ranking for the User's Vision

| Gap | Priority | Difficulty | Impact |
|-----|----------|------------|--------|
| Gap 1: Three-format bridge | ★★★★★ | High | Foundational — everything else depends on this |
| Gap 3: Science mode CLI | ★★★★★ | Medium | Most immediate user-facing value |
| Gap 2: Derivation trees | ★★★★☆ | Medium | Core interaction paradigm |
| Gap 5: Scientific verification | ★★★★☆ | High | Trust and reliability |
| Gap 7: Pedagogical adaptation | ★★★☆☆ | Medium | Learning effectiveness |
| Gap 4: Cross-domain transfer | ★★★☆☆ | Very High | Fundamental capability |
| Gap 6: Scale | ★★☆☆☆ | Medium | Incremental effort |
| Gap 8: Self-updating | ★★☆☆☆ | High | Long-term sustainability |
