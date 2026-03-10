# Deep Analysis: Lean 4 Autoformalization Ecosystem for Physics

**Date:** 2026-03-10
**Repositories analyzed:**
1. PhysLean (HEPLean/PhysLean) -- Lean4 physics formalization library
2. MerLean -- Agentic autoformalization for quantum computation
3. lean-lsp-mcp -- MCP server for Lean theorem prover

---

## 1. PhysLean (formerly HEPLean)

**Repository:** https://github.com/HEPLean/PhysLean
**Stars:** 507 | **Forks:** 80 | **Contributors:** 30 | **License:** Apache-2.0
**Created:** 2024-04-16 | **Last push:** 2026-03-09 (actively maintained)
**Commits:** 2,531 | **Open issues:** 70

### 1.1 Repository Structure

```
PhysLean/
├── .github/                    # CI/CD workflows
├── .vscode/                    # Editor configuration
├── PhysLean/                   # Main Lean 4 source library (16 physics modules)
│   ├── ClassicalMechanics/     # Lagrangian, Hamilton, oscillators, pendulum, waves
│   ├── CondensedMatter/        # Tight-binding chain model
│   ├── Cosmology/              # FLRW cosmology
│   ├── Electromagnetism/       # Maxwell's equations, charges, currents, point particles
│   ├── Mathematics/            # Supporting math (SO3, calculus, distributions, geometry)
│   ├── Meta/                   # Custom tactics, linters, metaprogramming
│   ├── Optics/                 # Polarization
│   ├── Particles/              # Standard Model, BSM, flavor physics, neutrinos, SUSY
│   ├── QFT/                    # Perturbation theory, QED, anomaly cancellation
│   ├── QuantumMechanics/       # D-dimensional, finite-target, 1D, Planck constant
│   ├── Relativity/             # Lorentz group/algebra, Minkowski, SL(2,C), Clifford, tensors
│   ├── SpaceAndTime/           # Spacetime foundations
│   ├── StatisticalMechanics/   # Canonical ensemble, Boltzmann constant
│   ├── StringTheory/           # F-theory/SU(5) models
│   ├── Thermodynamics/         # Ideal gas, temperature
│   └── Units/                  # Dimensional analysis framework
├── docs/                       # Documentation sources
├── scripts/                    # Linting, statistics, CI scripts (20+ executables)
├── PhysLean.lean               # Root import file
├── lakefile.toml               # Build configuration and dependencies
├── lean-toolchain              # Lean version pin
└── lake-manifest.json          # Resolved dependency versions
```

### 1.2 Dependencies

- **Lean 4:** v4.28.0
- **Mathlib:** v4.28.0 (the standard mathematics library for Lean 4; provides algebra, topology, analysis, category theory, etc.)
- **doc-gen4:** v4.28.0 (documentation generation)
- **Language:** 99.3% Lean

### 1.3 Architecture and Module Organization

PhysLean is organized by **physics domain** rather than by mathematical structure. Each top-level directory corresponds to a branch of physics and contains `.lean` files that formalize definitions, theorems, and calculations specific to that domain.

**Key architectural patterns:**

1. **Domain-first organization:** Physics concepts live under their natural domain (e.g., `Electromagnetism/Basic.lean` defines `ElectricField`, `MagneticField`, `EMSystem`), not under abstract mathematical categories.

2. **Dimensional analysis framework (`Units/`):**
   - `Dimension` is a structure with five `Q`-valued exponents: `length`, `time`, `mass`, `charge`, `temperature`
   - Base dimensions: `L_d` (length), `T_d` (time), `M_d` (mass), `C_d` (charge), `Theta_d` (temperature)
   - Dimensions form a commutative group under multiplication (exponent addition)
   - `UnitChoices` bundles five fundamental unit selections; `dimScale` computes scaling factors between unit systems
   - `Dimensionful M` is a subtype enforcing correct dimensional scaling -- quantities that transform properly under unit changes
   - `HasDimension` is a proposition verifying a value scales correctly
   - Works over reals (not floats) for mathematical rigor

3. **Physical constants as dependent types:**
   - Planck's constant `hbar` is typed as `Subtype (fun x : R => 0 < x)`, so positivity is a type-level guarantee
   - Speed of light derived as `c = 1/sqrt(mu_0 * epsilon_0)` within `EMSystem`
   - Boltzmann constant similarly formalized

4. **Mathlib integration:** PhysLean builds directly on Mathlib, using its:
   - Linear algebra (`LinearMap`, `PiTensorProduct`)
   - Group theory (Lorentz group as a matrix group)
   - Analysis (derivatives, integrals)
   - Topology (manifold structures for spacetime)

5. **Custom tactics and metaprogramming (`Meta/`):**
   - `TransverseTactics.lean`: Physics-specific proof automation
   - `Linters/`: Custom linting for physics code quality
   - `Informal/`: Infrastructure for documenting informal physics alongside formal proofs
   - `Sorry.lean`: Tracking incomplete proofs
   - `Notes/`, `Remark/`, `TODO/`: Organizational metadata

### 1.4 What Physics Is Formalized

**Classical Mechanics:**
- Lagrangian mechanics and Euler-Lagrange equations
- Hamilton's equations
- Harmonic oscillator (including damped)
- Pendulum dynamics
- Rigid body mechanics
- Wave equation
- Scattering theory
- Vibration modes

**Electromagnetism:**
- Maxwell's equations (full formalization)
- Electric and magnetic fields as spacetime maps
- Vector potentials
- Charge density (to be generalized to signed measures)
- Current density
- Vacuum solutions
- Point particle interactions
- EMSystem structure with epsilon_0, mu_0, derived c and Coulomb constant

**Quantum Mechanics:**
- Planck constant (positive real, with proof of positivity/non-zero)
- D-dimensional quantum systems
- Finite-target quantum mechanics
- 1-dimensional quantum systems
- Quantum harmonic oscillator

**Relativity:**
- Special relativity: proper time, twin paradox
- Minkowski metric/matrix
- Lorentz group and Lorentz algebra
- SL(2,C) representations
- Pauli matrices
- Clifford algebra
- Speed of light
- Tensor formalism (index notation)

**Quantum Field Theory:**
- Perturbation theory
- Wick's theorem (formally digitalized)
- QED anomaly cancellation
- General anomaly cancellation framework

**Particle Physics:**
- Standard Model basics and representations
- Higgs boson (two-Higgs doublet model)
- CKM matrix (flavor physics)
- Neutrino physics
- Beyond the Standard Model (BSM)
- Supersymmetry foundations

**Statistical Mechanics:**
- Canonical ensemble
- Boltzmann constant

**Thermodynamics:**
- Ideal gas laws
- Temperature formalization

**Condensed Matter:**
- Tight-binding chain model

**Cosmology:**
- FLRW (Friedmann-Lemaitre-Robertson-Walker) cosmology

**String Theory:**
- F-theory/SU(5) models

**Optics:**
- Polarization

**Supporting Mathematics:**
- SO(3) group theory
- Trigonometric identities
- Special functions
- Variational calculus
- Kronecker delta
- Schur triangulation
- Distribution theory
- Inner product spaces

### 1.5 Usability for Non-Lean Experts

**Entry barriers:**
- Lean 4 syntax is required for contributing formal proofs
- Mathlib conventions must be followed
- Local installation requires Lean 4 toolchain setup

**Accessibility features:**
- **GitPod integration:** Browser-based development without local setup (`.gitpod.yml`)
- **Live editor:** https://live.physlean.com/ for online experimentation
- **Search tool:** https://loogle.physlean.com/ for searching formalized results
- **Documentation:** https://physlean.com/docs/ with physics-oriented documentation
- **Informal contributions:** Contributors can write informal physics results (natural language) without Lean expertise, which are then formalized by others
- **Physics-first documentation:** Each formalization includes physics-based documentation, not just mathematical specifications

**Verdict:** A physicist can *explore* formalized results through the documentation and online tools without Lean expertise. *Contributing* formal proofs requires learning Lean 4, but informal contributions are explicitly welcomed.

### 1.6 Community and Documentation Quality

- **30 contributors** with 2,531 commits indicate sustained community effort
- **507 stars** place it among the most popular Lean 4 projects outside Mathlib itself
- **Multiple peer-reviewed publications** document the project's methodology
- **External adoption:** PhysLean theorems have been used in neural theorem-proving evaluation datasets
- **Governance:** Community-driven, no corporate affiliation
- **Active maintenance:** Last push within 24 hours of this analysis

---

## 2. MerLean: Agentic Autoformalization for Quantum Computation

**Paper:** "MerLean: An Agentic Framework for Autoformalization in Quantum Computation" (arXiv:2602.16554, Feb 2026)
**Authors:** Yuanjie Ren (MIT), Jinzheng Li (Northeastern), Yidi Qi (Northeastern)
**Example repo:** https://github.com/doxtor6/MerLean-examples (1 star, 64 commits)
**Website:** https://arthurmerlean.com/
**Interactive blueprint:** https://arthurmerlean.com/blueprint/

### 2.1 Repository Structure (MerLean-examples)

The MerLean framework itself is not open-sourced as a standalone tool; the published repository contains the **output** of the MerLean pipeline applied to one quantum error correction paper.

```
MerLean-examples/
├── .github/workflows/          # CI for building blueprint and docs
├── QEC1/                       # Generated Lean 4 library (~18,500 lines)
│   ├── Definitions/            # 12 formalized definitions
│   │   ├── Def_1_BoundaryAndCoboundaryMaps.lean
│   │   ├── Def_2_GaussLawAndFluxOperators.lean
│   │   ├── Def_3_DeformedOperator.lean
│   │   ├── Def_4_DeformedCode.lean
│   │   ├── Def_5_GaugingMeasurementAlgorithm.lean
│   │   ├── Def_6_CycleSparsifiedGraph.lean
│   │   ├── Def_7_SpaceAndTimeFaults.lean
│   │   ├── Def_8_Detectors.lean
│   │   ├── Def_9_Syndrome.lean
│   │   ├── Def_10_FaultTolerantGaugingProcedure.lean
│   │   ├── Def_11_SpacetimeLogicalFault.lean
│   │   └── Def_12_SpacetimeFaultDistance.lean
│   ├── Theorems/               # 2 main theorems + 1 corollary
│   │   ├── Thm_1_GaugingMeasurementCorrectness.lean
│   │   ├── Thm_2_FaultTolerantGaugingDistance.lean
│   │   └── Cor_1_WorstCaseOverhead.lean
│   ├── Lemmas/                 # 7 supporting lemmas
│   │   ├── Lem_1_DeformedCodeChecks.lean
│   │   ├── Lem_2_DecongestionLemmaBound.lean
│   │   ├── Lem_3_SpaceDistance.lean
│   │   ├── Lem_4_SpacetimeCodeDetectors.lean
│   │   ├── Lem_5_SpacetimeStabilizers.lean
│   │   ├── Lem_6_TimeFaultDistance.lean
│   │   └── Lem_7_SpaceTimeDecoupling.lean
│   ├── Remarks/                # 26 formalized remarks
│   │   ├── Rem_1_NotationBinaryVectors.lean
│   │   ├── Rem_2_NotationPauliOperators.lean
│   │   ├── ...
│   │   └── Rem_26_CohenEtAlSchemeRecovery.lean
│   ├── content.tex             # Auto-generated LaTeX blueprint source
│   └── print.tex               # Print-ready LaTeX
├── blueprint/src/              # leanblueprint-compatible output
├── home_page/                  # Website assets for arthurmerlean.com
├── QEC1.lean                   # Root import file
├── lakefile.toml               # Build config
├── lean-toolchain              # leanprover/lean4:v4.28.0-rc1
├── lake-manifest.json          # Resolved deps
├── MerLean_arxiv.pdf           # The arXiv paper
└── README.md
```

### 2.2 Dependencies

- **Lean 4:** v4.28.0-rc1
- **Mathlib:** v4.28.0-rc1
- **checkdecls:** Patrick Massot's declaration checking tool
- **doc-gen4:** Documentation generator
- **Language composition:** 73.4% TeX, 25.9% Lean (reflects the bidirectional nature: generated LaTeX blueprints + Lean proofs)

### 2.3 The Agentic Loop Architecture

MerLean is a fully automated, three-stage pipeline with no human-in-the-loop:

```
                    ┌─────────────────────────────────────────────┐
                    │              MerLean Pipeline                │
                    │                                             │
  LaTeX paper ─────>│  Stage 1: EXTRACTION                       │
                    │    Parse LaTeX → structured JSON            │
                    │    {id, content, dependencies, proof_sketch}│
                    │    Iterative expansion of vague phrases     │
                    │                                             │
                    │  Stage 2: FORMALIZATION (compile-fix loop)  │
                    │    For each statement:                      │
                    │      1. Generate Lean 4 code                │
                    │      2. Compile via Lean/LSP                │
                    │      3. Diagnose failures                   │
                    │      4. Search Mathlib (LeanSearch/Loogle)  │
                    │      5. Refine code                         │
                    │      6. Repeat (up to 30 attempts)          │
                    │      7. If stuck → axiom phase              │
                    │      8. Faithfulness check                  │
                    │                                             │
                    │  Stage 3: AUTOINFORMALIZATION               │
                    │    Verified Lean → human-readable LaTeX     │
                    │    Interactive blueprint + narrative         │
                    │    Axiom declarations flagged prominently    │
                    │                                             │
  Verified Lean ◄──│──────────────────────────────────────────── │
  + LaTeX blueprint │                                             │
                    └─────────────────────────────────────────────┘
```

**Stage 1: Statement Extraction**

The LLM (Claude Opus 4.5) processes the LaTeX source and extracts each mathematical statement (definition, lemma, theorem, remark, corollary) into structured JSON:
- Unique identifier (e.g., `Def_1`, `Thm_2`)
- Mathematical content in natural language
- Explicit dependencies on other statements
- Proof sketches when available

The extraction is iterative: vague phrases like "by standard arguments" are expanded into concrete proof steps across multiple passes.

**Stage 2: Iterative Formalization (Compile-Fix Loop)**

This is the core agentic loop. For each extracted statement:

1. The agent generates Lean 4 code with one or more declarations
2. Code is compiled via the Lean 4 compiler (through lean-lsp-mcp)
3. On compilation failure, the agent:
   - Reads and diagnoses error messages
   - Uses `lean_goal` to inspect proof states at specific positions
   - Uses `lean_hover_info` to retrieve type signatures and documentation
   - Searches Mathlib via `leansearch` (natural language) and `loogle` (type signature search)
   - Uses `grep` to find usage patterns in Mathlib source
   - Refines the code and recompiles
4. This iterates up to **30 attempts**
5. If still failing, the **axiom phase** converts blocking subgoals to explicit axioms
6. After successful compilation, a **faithfulness check** verifies the formalization matches the original mathematical meaning (prevents trivially-true type-checked code)

**Stage 3: Autoinformalization**

The same LLM translates each verified Lean declaration back to natural language LaTeX:
- Parses all Lean files and constructs dependency graphs
- Produces an interactive blueprint (leanblueprint-compatible)
- Produces a standalone narrative (textbook style)
- Prominently flags unverified axiom declarations

### 2.4 MCP Tools Used

MerLean interfaces with the Lean theorem prover through **lean-lsp-mcp**, using these specific tools:

| Tool | Purpose |
|------|---------|
| `lean_goal` | Inspect proof state (goals) at a specific file position |
| `lean_hover_info` | Retrieve type signatures, documentation for any term |
| `lean_diagnostic_messages` | Get compilation errors and warnings |
| `lean_leansearch` | Natural language search for Mathlib lemmas |
| `lean_loogle` | Type-signature-based search for Mathlib lemmas |
| `lean_local_search` | ripgrep-based search through project and Mathlib source |
| `lean_completions` | IDE-style autocompletion |
| `lean_run_code` | Execute self-contained Lean snippets |

### 2.5 Quality of Generated Code

Examining the generated Lean code reveals high quality:

**Example -- Boundary and Coboundary Maps (Def_1):**
```lean
/-- The boundary map partial : Z_2^E -> Z_2^V.
For gamma in Z_2^E, (partial gamma)_v = sum_{e ni v} gamma_e (mod 2). -/
noncomputable def boundaryMap [Fintype G.edgeSet] :
    (G.edgeSet -> ZMod 2) ->l[ZMod 2] (V -> ZMod 2) where
  toFun gamma v := sum e : G.edgeSet, if (v in (e : Sym2 V)) then gamma e else 0
  map_add' := by intro gamma_1 gamma_2; ext v; simp only [Pi.add_apply]; ...
  map_smul' := by intro r gamma; ext v; simp only [...]; ...
```

Key observations:
- Proper use of Mathlib types (`ZMod 2`, `Sym2`, `LinearMap`)
- Full proofs of linearity (map_add, map_smul)
- Physics-meaningful documentation strings
- Correct namespace organization with `open` statements
- Appropriate use of `noncomputable` for real-valued constructions

**Example -- Gauging Measurement Correctness (Thm_1):**
- Imports from its own dependency graph (Def_5, Def_2, Def_1, Rem_5, Rem_2)
- Rich documentation including statement, proof sketch, and main results list
- Uses Mathlib's `SimpleGraph.Connectivity.Connected`
- Defines intermediate constructions (like `gaussSubsetProduct`) as stepping stones

### 2.6 Evaluation Results

MerLean was evaluated on three quantum computation papers (114 total statements):

| Paper | Statements | Lean Lines | Declarations | Time | Axioms needed |
|-------|-----------|-----------|--------------|------|---------------|
| Balanced Product Codes | 44 | 14,997 | 730 | 20h 4m | 9.1% of stmts |
| Fault-Tolerant QC | 47 | 18,557 | 923 | 11h 41m | -- |
| Quantum Topology | 23 | 7,761 | 397 | 7h 51m | -- |
| **Total** | **114** | **41,315** | **2,050** | **39h 36m** | -- |

**Iteration statistics by statement type:**

| Type | Count | Avg. Time | Avg. Compile Attempts |
|------|-------|-----------|----------------------|
| Definition | 49 | 18 min | 11.7 |
| Theorem | 15 | 40 min | 22.4 |
| Lemma | 20 | 33 min | 18.3 |
| Remark | 26 | 11 min | 7.1 |

Theorems are hardest (40 min, 22 iterations on average), while remarks are easiest. The long tail of 21+ iteration statements typically involves algebraic domains where Mathlib lacks specialized infrastructure (e.g., Kunneth formula for F_2-chain complexes, tensor-homology commutativity).

### 2.7 Usability

MerLean is **not yet available as a standalone tool** for general use. The published artifacts are:
- The arXiv paper describing the methodology
- One example output repository (MerLean-examples)
- The interactive blueprint website

A scientist could not currently use MerLean to formalize their own paper. However, the architecture (Claude Opus 4.5 + lean-lsp-mcp + Mathlib) is reproducible from the paper's description.

### 2.8 Community

- **Very early stage:** 1 star, 0 forks, 3 authors
- Created February 2026, last updated February 2026
- The framework itself (the agentic pipeline code) is not open-sourced
- Only the output examples are public

---

## 3. lean-lsp-mcp: MCP Server for Lean Theorem Prover

**Repository:** https://github.com/oOo0oOo/lean-lsp-mcp
**Stars:** 312 | **Forks:** 50 | **Contributors:** 13 | **License:** MIT
**Created:** 2025-03-29 | **Last push:** 2026-03-08 (actively maintained)
**Version:** 0.23.2

### 3.1 Repository Structure

```
lean-lsp-mcp/
├── .github/workflows/          # CI/CD
├── src/lean_lsp_mcp/           # Main Python source (14 files)
│   ├── __init__.py
│   ├── __main__.py             # Entry point
│   ├── server.py               # Core MCP server (21 tools)
│   ├── repl.py                 # Fast REPL-based tactic execution
│   ├── client_utils.py         # LSP client utilities
│   ├── file_utils.py           # File I/O helpers
│   ├── instructions.py         # System prompts/instructions
│   ├── loogle.py               # Loogle integration (local + remote)
│   ├── models.py               # Data models (Pydantic/dataclass)
│   ├── outline_utils.py        # Token-efficient file outline extraction
│   ├── profile_utils.py        # Proof performance profiling
│   ├── search_utils.py         # Local search via ripgrep
│   ├── utils.py                # General utilities
│   └── verify.py               # Axiom checking + soundness scanning
├── tests/                      # Test suite
├── pyproject.toml              # Package configuration
├── .python-version             # Python version pin
├── uv.lock                     # Dependency lock file
├── release.sh                  # Release automation
└── LICENSE                     # MIT
```

### 3.2 Dependencies

- **Python:** >= 3.10
- **leanclient:** 0.9.3 (Python LSP client for Lean 4)
- **mcp[cli]:** 1.26.0 (Model Context Protocol SDK)
- **orjson:** >= 3.11.1 (fast JSON)
- **certifi:** >= 2024.0.0 (TLS certificates)
- **Optional:** PyYAML, ruff (linting), pytest (testing)

### 3.3 Architecture

The server implements the **Model Context Protocol (MCP)**, which is the standard interface for LLM agents to interact with external tools. It bridges two worlds:

```
LLM Agent (Claude, etc.)
    |
    | MCP protocol (stdio / HTTP SSE)
    v
lean-lsp-mcp server (Python)
    |
    | LSP protocol
    v
Lean 4 Language Server
    |
    v
Lean 4 Project (with Mathlib)
```

**Key architectural decisions:**

1. **Shared resources:** Global singletons for Loogle manager (~6 GB RSS) and thread locks prevent resource exhaustion
2. **REPL fast path:** The `Repl` class provides sub-second tactic evaluation by maintaining a persistent Lean subprocess with cached imports, avoiding full recompilation
3. **Rate limiting:** External search APIs are rate-limited (3-10 requests per 30 seconds) via timestamp-tracking decorators
4. **Transport flexibility:** Supports stdio (for local IDEs), HTTP streaming, and server-sent events
5. **Authentication:** Optional token-based auth via environment variable

### 3.4 Complete Tool Inventory (21 Tools)

#### File Interaction Tools

| Tool | Description | Read-only |
|------|-------------|-----------|
| `lean_file_outline` | Token-efficient declaration extraction from a file | Yes |
| `lean_diagnostic_messages` | Returns errors/warnings with optional severity filtering | Yes |
| `lean_goal` | Retrieves proof state (goals) at a specific position; omit column for goals_before/goals_after | Yes |
| `lean_term_goal` | Gets expected type at cursor position | Yes |
| `lean_hover_info` | Type signatures, documentation, and metadata for any term | Yes |
| `lean_completions` | IDE-style autocompletion with relevance sorting | Yes |
| `lean_declaration_file` | Locates where a symbol is defined (go-to-definition) | Yes |
| `lean_code_actions` | LSP code actions including "Try This" tactic suggestions | Yes |
| `lean_get_widgets` | Panel widgets and interactive proof visualizations | Yes |
| `lean_get_widget_source` | JavaScript source of custom Lean widgets | Yes |

#### Code Execution Tools

| Tool | Description | Read-only |
|------|-------------|-----------|
| `lean_multi_attempt` | Tries multiple tactics via REPL (fast) or LSP fallback | No |
| `lean_run_code` | Executes self-contained Lean snippets | No |
| `lean_build` | Builds the full project with progress reporting | No |

#### Search Tools (with rate limits)

| Tool | Rate Limit | Description |
|------|-----------|-------------|
| `lean_local_search` | N/A | Fast ripgrep-based search across project + `.lake/packages` + Lean stdlib |
| `lean_leansearch` | 3/30s | Natural language search via leansearch.net API |
| `lean_loogle` | 3/30s (remote) | Type-signature pattern matching; local mode or remote API fallback |
| `lean_leanfinder` | 10/30s | Semantic search via Hugging Face model |
| `lean_state_search` | 6/30s | Lemma suggestions from premise-search.com based on current goal state |
| `lean_hammer_premise` | 6/30s | Automation tactic premise suggestions |

#### Analysis Tools

| Tool | Description | Read-only |
|------|-------------|-----------|
| `lean_verify` | Checks what axioms a theorem depends on + scans for unsound patterns | Yes |
| `lean_profile_proof` | Runs `lean --profile` for per-line timing analysis | Yes |

### 3.5 How an LLM Agent Interacts with Lean

The interaction flow for a typical proof attempt:

1. **Agent writes Lean code** to a `.lean` file
2. **Agent calls `lean_diagnostic_messages`** to get compilation errors
3. **If errors exist**, agent reads the specific error messages and:
   - Calls `lean_goal` at the error position to see the proof state
   - Calls `lean_hover_info` on relevant terms to understand types
   - Calls `lean_leansearch` or `lean_loogle` to find applicable lemmas
   - Calls `lean_local_search` to find usage examples in Mathlib
4. **Agent modifies the code** and repeats from step 2
5. **For tactic exploration**, agent uses `lean_multi_attempt` to test multiple tactics in parallel via the REPL (much faster than full recompilation)
6. **After proof completion**, agent calls `lean_verify` to confirm no unsound axioms are used

**The REPL fast path** is particularly important for performance:
- Regular LSP: Each code change requires waiting for Lean to recheck the entire file
- REPL: Maintains a persistent subprocess with cached imports; evaluates individual tactics in milliseconds
- The `run_snippets()` method: loads imports once, executes body with `sorry` to extract proof state, then tests each tactic candidate against that state

**Verification scanning** checks for 13 suspicious patterns:
- `@[implemented_by]` and `@[extern]` (native code escape hatches)
- `set_option ... true` for debug/unsafe options
- `opaque` definitions
- `Lean.Elab` and `Lean.Meta` imports (metaprogramming that could compromise soundness)
- `local` scoping that might hide assumptions

### 3.6 Usability

**For LLM agent developers:**
- Install with `uv`: straightforward Python package installation
- Configure via environment variables (project path, log level, REPL mode)
- Works with Claude Code, Cursor, VSCode via MCP configuration
- Well-documented tool schemas with `Annotated` parameter descriptions

**For scientists without Lean expertise:**
- lean-lsp-mcp is infrastructure, not a user-facing tool
- A scientist would use it indirectly through an LLM agent (like MerLean's pipeline)
- The agent handles all Lean interaction; the scientist provides natural language or LaTeX

**Configuration example for Claude Code:**
```json
{
  "mcpServers": {
    "lean-lsp-mcp": {
      "command": "uvx",
      "args": ["lean-lsp-mcp"],
      "env": {
        "LEAN_PROJECT_PATH": "/path/to/lean/project"
      }
    }
  }
}
```

### 3.7 Community

- **312 stars, 50 forks** -- significant adoption in the Lean/formal verification community
- **13 contributors** with active development
- **Adopted by MerLean** and other formalization research projects
- **Beta status:** Documentation explicitly notes it is a research tool without comprehensive input validation
- **Rapid iteration:** Version 0.23.2 as of analysis date, indicating frequent releases

---

## 4. Cross-Cutting Analysis

### 4.1 How the Three Projects Relate

```
                 PhysLean
                 (Lean 4 physics library)
                 Manually curated, 30 contributors
                 507 stars, production quality
                      |
                      | imports as dependency
                      v
    MerLean ──────> Mathlib v4.28.0 <────── lean-lsp-mcp
    (agentic         (standard math           (MCP server)
    pipeline)         library)                     |
       |                                           |
       | uses as tool                              | used by
       +───────────────────────────────────────────+
                                                   |
                                            LLM agents
                                          (Claude, etc.)
```

- **PhysLean** is the **destination**: a curated, human-maintained library of formalized physics
- **lean-lsp-mcp** is the **bridge**: enabling LLM agents to interact with the Lean compiler
- **MerLean** is the **automation**: using lean-lsp-mcp to drive an LLM through the full formalization pipeline

### 4.2 Lean Version Alignment

All three projects target the same Lean 4 / Mathlib version ecosystem:
- PhysLean: Lean 4.28.0, Mathlib 4.28.0
- MerLean-examples: Lean 4.28.0-rc1, Mathlib 4.28.0-rc1
- lean-lsp-mcp: Compatible with any Lean 4 project (version-agnostic)

### 4.3 Formalization Coverage Comparison

| Domain | PhysLean | MerLean |
|--------|----------|---------|
| Quantum mechanics | Yes (basic) | Yes (error correction, stabilizers) |
| Electromagnetism | Yes (Maxwell's eqs) | No |
| Relativity | Yes (special + tensors) | No |
| Particle physics | Yes (Standard Model) | No |
| Quantum field theory | Yes (Wick's theorem) | No |
| Quantum error correction | No | Yes (3 papers) |
| Classical mechanics | Yes (Lagrangian, Hamilton) | No |
| Statistical mechanics | Yes (canonical ensemble) | No |
| String theory | Yes (F-theory) | No |

PhysLean has **breadth** across physics; MerLean has **depth** in quantum error correction. They are complementary.

### 4.4 Manual vs. Automated Formalization

| Aspect | PhysLean (manual) | MerLean (automated) |
|--------|-------------------|---------------------|
| Code quality | High, curated | High but with axiom gaps |
| Proof completeness | Full (no `sorry`) | Mostly full; axioms for Mathlib gaps |
| Time per statement | Hours-days (human) | 10-40 min (automated) |
| Faithfulness | Guaranteed (expert review) | Checked by LLM + compilation |
| Scalability | Limited by contributors | Limited by compute + API costs |
| Physics accuracy | Domain expert verified | LLM-assessed faithfulness |
| Mathlib integration | Deep, idiomatic | Functional but sometimes workaround-heavy |

### 4.5 Key Limitations and Open Problems

**PhysLean:**
- Many physics domains are shallow (1-2 files in Cosmology, Optics, Condensed Matter)
- Some files are explicitly marked "old" with TODOs for modernization
- Quantum mechanics formalization is basic compared to the relativity/particle physics depth
- No numerical computation capability (purely symbolic/logical)

**MerLean:**
- Framework code is not open-sourced (only output examples)
- Depends on Claude Opus 4.5 (expensive, proprietary)
- 9.1% axiom rate in the hardest paper (Balanced Product Codes) reflects Mathlib gaps
- No guarantee of semantic correctness beyond LLM-based faithfulness checks
- 40-minute average for theorems means a full paper takes 10-20 hours of API calls

**lean-lsp-mcp:**
- Beta quality with acknowledged security concerns (file system access without input validation)
- Rate limits on external search APIs constrain rapid iteration
- REPL mode requires separate binary (`repl`) that may not be available in all Lean projects
- 6 GB RSS for Loogle is substantial memory overhead

### 4.6 Implications for Scientific Reasoning

These three projects together represent a maturing pipeline for **machine-verified scientific reasoning**:

1. **Scientists write physics** in natural language or LaTeX
2. **MerLean-style agents** extract and formalize statements using lean-lsp-mcp
3. **Lean 4 + Mathlib** provide the trusted verification kernel
4. **PhysLean-style libraries** accumulate verified results for reuse
5. **Autoinformalization** produces human-readable output for expert review

The main bottleneck is **Mathlib coverage**: when the mathematics library lacks a theorem or construction needed for a physics proof, the system must either axiomatize (losing verification value) or the theorem must be manually added to Mathlib first. This is the fundamental tension between automated and manual formalization.

---

## 5. Summary Table

| Property | PhysLean | MerLean | lean-lsp-mcp |
|----------|----------|---------|--------------|
| **Type** | Lean 4 library | Agentic pipeline | MCP server |
| **Language** | Lean 4 (99.3%) | Not open-sourced | Python |
| **Stars** | 507 | 1 (examples only) | 312 |
| **Contributors** | 30 | 3 | 13 |
| **Lean version** | 4.28.0 | 4.28.0-rc1 | Any Lean 4 |
| **Mathlib** | Yes (4.28.0) | Yes (4.28.0-rc1) | N/A |
| **Physics scope** | 16 domains | QEC (3 papers) | N/A (tool) |
| **Automation** | Manual | Fully automated | Enables automation |
| **LLM required** | No | Claude Opus 4.5 | Any MCP-compatible |
| **Scientist-friendly** | Docs + online editor | Not yet available | Indirect (via agents) |
| **License** | Apache-2.0 | N/A | MIT |
| **Maturity** | Production | Research prototype | Beta |
