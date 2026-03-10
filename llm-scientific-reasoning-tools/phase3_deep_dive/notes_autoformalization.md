# Phase 3 Deep Dive: Autoformalization of Scientific Knowledge

## Overview

These three papers represent a coherent thread in the emerging field of **autoformalization** -- using LLMs to translate scientific knowledge (PDEs, quantum computation, physics) from informal natural language and LaTeX into formally verifiable representations (Signal Temporal Logic, Lean 4 proofs). Together, they demonstrate that LLMs can serve as bridges between human-readable scientific statements and machine-verifiable formal code, covering applied mathematics (PDEs), quantum computing, and college-level physics.

---

## Paper 4: PDE-Controller -- LLMs for Autoformalization and Reasoning of PDEs

**Venue**: ICML 2025 | **arXiv**: 2502.00963

### Problem

While LLMs have made significant progress in pure mathematics, they struggle with **applied mathematics**, specifically PDE control. Traditional PDE control requires manual formalization of informal specifications, deep expertise in both physics and coding, and understanding of complex spatiotemporal dynamics. The paper asks: "Can LLMs control PDEs with scientific reasoning?" The goal is automating the full pipeline from natural-language PDE specifications to optimized control trajectories.

### Key Contributions

- **Autoformalization of PDEs into Signal Temporal Logic (STL)**: Achieves >64% accuracy converting natural language PDE control specifications into formal STL constraints; >82% executable code generation rate.
- **Scientific reasoning via subgoal decomposition**: A Controller LLM trained with DPO proposes intermediate subgoals that decompose hard PDE control problems, yielding 62% improvement in control utility over baseline LLM prompting.
- **Large-scale synthetic dataset**: 2.13 million training samples constructed via principled template synthesis + ChatGPT paraphrasing, plus 34 manually-written real-world problems.
- **End-to-end pipeline**: Four-stage system (Translator -> Controller -> Coder -> Gurobi Optimizer) that goes from natural language to solved PDE control.

### Methodology

**Formal Representation -- Signal Temporal Logic (STL)**:
PDEs are formalized using STL constraints of the form:
```
phi = T[t1,t2](forall x in [x1,x2], u(x) </>/= (ax+b))
```
where T is a temporal operator (G = globally/always, F = eventually), combined with logical operators (AND, OR). This captures spatiotemporal constraints on PDE solutions (e.g., "temperature must stay within 3K of a linear profile between x=0.2 and x=0.8 for all times t in [0.5, 1.0]").

**Utility is computed via continuous STL semantics**:
- r(u, u >= ax+b, t) = u(x,t) - (ax+b)
- r(phi1 AND phi2, t) = min{r(phi1,t), r(phi2,t)}
- r(G[a,b] phi, t) = inf over [t+a, t+b) of r(phi, tg)
- Positive robustness r > 0 means the constraint is satisfied.

**PDEs Covered**:
- Heat equation: rho*c * du/dt - kappa * d^2u/dx^2 = 0
- Wave equation: rho * d^2u/dt^2 - E * d^2u/dx^2 = 0

**PDE Discretization**:
- Finite Element Method (FEM) for spatial discretization
- Temporal finite differences
- Produces system: (M + dt*K) u^{k+1} = M*u^k + dt*F^{k+1}
- Formulated as Mixed-Integer Linear Program (MILP) with binary variables for spatial/temporal constraint application

### Architecture / Pipeline

```
Natural Language Specification
        |
        v
  [TRANSLATOR LLM] -- Autoformalization to STL (fine-tuned MathCoder2-DeepSeekMath-7B via SFT)
        |                  >64% IoU accuracy
        v
  [CONTROLLER LLM] -- PDE Reasoning: decomposes anchor problem phi into subgoal phi'
        |                  Trained via DPO on 10,772 win-lose STL pairs
        |                  Sequential solving: optimize phi' first, use result as new IC, then solve phi
        v
  [CODER LLM]      -- Program Synthesis: generates Python/Gurobi code (SFT, >82% executable)
        |
        v
  [GUROBI OPTIMIZER] -- Solves MILP formulation
        |
        v
  Control Trajectory r(phi)
```

Key design choices:
- Controller trained with **Direct Preference Optimization (DPO)** on pairs where subgoal phi' either improves or degrades control utility
- DPO regularized with SFT loss to prevent degradation
- Multiple samples from Controller for robustness
- Gurobi timeout: 120s per subgoal solve

### Experiments

**Dataset Construction**:
- 1,374 STL syntax templates (6 single-constraint formats combined with logical operators, up to 3 constraints)
- Hyperparameter sampling for initial conditions, simulation domains, physical properties
- ChatGPT rephrases each sample 5x -> 2.13M total samples
- 34 real-world problems written by domain experts (undergrad/grad students in Math, EE, CS, Physics)

| PDE  | Constraints | Train    | Test     |
|------|-------------|----------|----------|
| Heat | 1           | 3,840    | 960      |
| Heat | 2           | 45,792   | 11,448   |
| Heat | 3           | 817,776  | 204,768  |
| Wave | 1           | 3,840    | 960      |
| Wave | 2           | 45,504   | 11,304   |
| Wave | 3           | 795,744  | 196,992  |

**Autoformalization Results (Synthetic Data)**:

| Model              | Heat IoU | Heat Exec | Heat RMSE | Wave IoU | Wave Exec | Wave RMSE |
|--------------------|----------|-----------|-----------|----------|-----------|-----------|
| **Ours**           | **0.992**| **0.998** | **0.017** | **0.992**| **0.962** | **0.008** |
| MathCoder2         | 0.772   | 0.959     | 0.206     | 0.772   | 0.934     | 0.109     |
| GPT-4o             | --      | 0.581     | 0.045     | --      | 0.680     | 0.087     |
| GPT-o1-mini        | --      | 0.356     | 0.090     | --      | 0.404     | 0.076     |

**Real-World Data**: Translator achieves 0.71 IoU (heat), 0.65 IoU (wave); Coder achieves 0.82 executability (heat), 1.0 (wave). Baselines struggle with noisy human writing.

**PDE Reasoning (Controller) Results** -- Heat Problems:

| Difficulty | Our P_bar | Our Delta_r | MathCoder2 Delta_r | GPT-4o Delta_r |
|------------|-----------|-------------|-------------------|----------------|
| Easy       | 0.966     | **2.233**   | 0.795             | 1.614          |
| Medium     | 0.877     | **1.090**   | -0.109            | 0.222          |
| Hard       | 0.592     | **1.035**   | -0.964            | 0.855          |
| **All**    | **0.812** | **1.453**   | -0.093            | 0.897          |

**Wave Problems**:

| Difficulty | Our P_bar | Our Delta_r | MathCoder2 Delta_r | GPT-4o Delta_r |
|------------|-----------|-------------|-------------------|----------------|
| Easy       | 0.936     | 1.423       | 1.601             | 1.706          |
| Medium     | 0.833     | 0.901       | 0.830             | 0.652          |
| Hard       | 0.328     | -0.349      | -0.670            | -0.609         |
| **All**    | **0.699** | **0.658**   | 0.587             | 0.583          |

On heat problems, the Controller achieves **62% improvement** (Delta_r = 1.453 vs GPT-4o = 0.897) and generates **82.7% valid STLs** (vs MathCoder2 = 42.45%).

### Limitations

1. **Real-world performance degrades**: Models struggle with inconsistent notation, hallucinated numbers, and invalid time constraints in human-written specifications.
2. **Narrow PDE scope**: Only 1D heat/wave equations; extension to higher dimensions or nonlinear PDEs is unclear.
3. **Hard problems remain unsolved**: All models fail on "hard" wave problems (negative Delta_r).
4. **Scalability of RLHF**: Requires 10K+ preference labels per dataset for DPO training.
5. **Linear constraints only**: STL profiles limited to form ax+b; nonlinear constraints not supported.

### Connections to Broader Vision

- **Bridges natural language -> formal specification -> executable code**: The Translator performs autoformalization (NL -> STL), analogous to informal-to-formal translation in theorem proving.
- **Integrates symbolic solver**: Gurobi provides grounded, verified solutions, unlike purely neural approaches.
- **STL from formal methods**: Adopts Signal Temporal Logic from model checking and verification (Maler & Nickovic 2004), connecting PDE control to the formal methods community.
- **Physics-grounded reward signals**: DPO training uses actual PDE solution quality as reward, grounding LLM reasoning in physical reality.
- **Demonstrates domain-specific autoformalization**: Shows that autoformalization extends beyond pure math to applied mathematics/engineering, a key step toward formalizing all of science.

---

## Paper 5: MerLean -- An Agentic Framework for Autoformalization in Quantum Computation

**Venue**: arXiv preprint | **arXiv**: 2602.16554

### Problem

The quantum physics community faces a verification bottleneck: the arXiv quant-ph category received 11,891 submissions in 2025 alone, overwhelming peer review. Mathematical statements in quantum computing papers are highly rigorous (rooted in linear algebra, group theory, algebraic topology) yet lack formal verification. MerLean addresses this by automating the translation of entire quantum computing research papers from LaTeX into formally verified Lean 4 code -- without human-in-the-loop intervention during formalization.

### Key Contributions

- **Fully automated end-to-end autoformalization**: Formalizes complete research papers (not isolated theorems) into verified Lean 4 code built on Mathlib, without human domain guidance during the formalization loop.
- **Research-scale results**: Generated 2,050 Lean declarations from 114 statements across three quantum computing papers, totaling 41,000+ lines of verified Lean 4 code.
- **Bidirectional pipeline ("autoinformalization")**: Converts verified Lean 4 code back into human-readable LaTeX, enabling domain experts to validate semantic alignment without formal methods expertise.
- **Scalable synthetic data generation**: Produces high-quality (natural language, formal code) pairs for training future reasoning models.
- **Transparent axiom handling**: When proofs fail after maximum attempts, blocking subgoals are converted to explicit axiom declarations, clearly distinguishing verified results from assumptions.

### Methodology

**Phase 1 -- Statement Extraction**:
- Agent extracts mathematical statements (definitions, theorems, lemmas, propositions, corollaries, remarks) from LaTeX source files
- Outputs structured JSON with unique identifiers, natural language content, explicit dependencies, and proof sketches
- Multi-pass iterative refinement: expands vague phrases, adds missing intermediate lemmas, ensures dependency ordering

**Phase 2 -- Iterative Formalization Loop**:
- For each statement, generates Lean 4 declarations (often multiple per statement for auxiliary definitions/lemmas)
- Writes to library structure and compiles
- On failure: parses Lean compiler error messages (type mismatches, unknown identifiers, tactic failures) and feeds back to the agent
- Continues until compilation succeeds or reaches maximum attempt limit (30 attempts)
- Agent autonomously introduces helper lemmas, auxiliary definitions, and typeclass instances

**Phase 3 -- Faithfulness Checking**:
After successful compilation, the agent reflects on whether the formalized code actually matches the original mathematical meaning. This is critical because an LLM can produce code that type-checks but misrepresents the mathematics (e.g., proving a trivial variation instead of the intended theorem).

**Phase 4 -- Axiom Phase**:
When formalization fails after 30 attempts, blocking subgoals are converted to explicit axiom declarations, transparently marking the boundary between what is verified and what is assumed.

**Phase 5 -- Autoinformalization**:
A separate LLM pass converts verified Lean 4 code back to LaTeX (without access to original paper, preventing data leakage). Produces:
1. Interactive blueprint compatible with leanblueprint for dependency exploration
2. Textbook-style narrative for readers without formal methods background
All unverified assumptions (axioms) are prominently highlighted.

**Tool Integration via Model Context Protocol (MCP)**:
The framework equips agents with:
- `lean-lsp-mcp`: Language server for Lean (compilation, error diagnostics)
- `lean_goal`: Inspect proof states at specific cursor positions
- `lean_hover_info`: Type signatures and documentation lookup
- `leansearch` and `loogle`: Semantic search for discovering relevant Mathlib lemmas
- `grep` through Mathlib source code for usage patterns

### Architecture / Pipeline

```
LaTeX Research Paper
        |
        v
  [Statement Extraction Agent]  -- Multi-pass extraction to structured JSON
        |                           (definitions, theorems, lemmas, dependencies)
        v
  [Iterative Formalization Agent]  -- Generate Lean 4 code
        |                              |
        |                     [Lean Compiler + MCP Tools]
        |                              |
        |                     Error feedback loop (up to 30 attempts)
        |                              |
        v                              v
  [Faithfulness Check]            [Axiom Phase] (if formalization fails)
        |
        v
  Verified Lean 4 Library (41K+ lines)
        |
        v
  [Autoinformalization Agent]  -- Lean 4 -> LaTeX (no original paper access)
        |
        v
  Human-Readable Blueprint + Narrative
```

**Primary Agent**: Claude Code (frontier LLM) performs all tasks: extraction, Lean generation, error diagnosis, refinement, faithfulness checking, and natural language translation.

### Experiments

**Papers Evaluated**:

| Paper | Domain | Focus | Contamination Risk |
|-------|--------|-------|--------------------|
| A: Balanced Product Codes (Breuckmann & Eberhardt 2021) | Quantum Codes | Tensor products, fiber bundles, chain complexes, homological algebra, expander graphs | Published |
| B: Fault-Tolerant QC (Williamson & Yoder 2024) | Stabilizer Codes | Fault-tolerant protocols, stabilizer formalism, Pauli algebra, transversal gates | Published |
| C: Quantum Topology (unpublished) | Algebraic Properties | Group-theoretic properties of quantum computational systems | **Zero contamination** |

**Quantitative Results**:

| Metric | Paper A | Paper B | Paper C | Total |
|--------|---------|---------|---------|-------|
| Statements formalized | 44 | 47 | 23 | **114** |
| Lines of Lean 4 | 14,997 | 18,557 | 7,761 | **41,315** |
| Lean declarations | 730 | 923 | 397 | **2,050** |
| Wall-clock time | 20h 4m | 11h 41m | 7h 51m | **~42 hours** |

**By Statement Type**:

| Type | Count | Avg Time | Avg Compile Attempts | Axioms Required |
|------|-------|----------|----------------------|-----------------|
| Definition | 49 | 18m 0s | 11.7 | Minimal |
| Theorem | 15 | 39m 41s | 22.4 | Some |
| Lemma | 20 | 33m 22s | 18.3 | Some |
| Remark | 26 | 10m 34s | 7.1 | None |
| Corollary | 4 | 19m 23s | 5.5 | None |

**Key Observations**:
- Theorems are hardest: 39m 41s average, 22.4 compile attempts
- Remarks are easiest: 10m 34s, 7.1 attempts, zero axioms
- Distribution is heavily right-skewed: most statements resolve in 1-10 attempts, with a long tail of 20+ iterations
- Balanced Product Codes paper required the most axioms (9.1% of statements) due to missing Mathlib support for Kunneth formula, spectral sequences, and Tor terms over F_2 chain complexes

### Limitations

1. **Mathlib gaps**: Specialized constructs (Kunneth formula for F_2-chain complexes, spectral sequences, certain Tor terms) lack Lean 4 formalization, requiring axiomatization of classical results.
2. **Faithfulness is hard to guarantee**: LLMs can produce code that type-checks but misrepresents the intended mathematics (e.g., proving trivial variations).
3. **Scope of axioms**: Choosing the "right" abstraction level for axioms is non-trivial -- too elementary means re-proving classical math; too high trivializes the derivation.
4. **Domain-specific evaluation only**: Tested on quantum computation; generalizability to other mathematical fields with deeper dependency chains remains unvalidated.
5. **Existing benchmarks inadequate**: miniF2F and ProofNet focus on isolated theorems, not interconnected full-paper formalizations. No established benchmark exists for this task.

### Connections to Broader Vision

- **Full-paper autoformalization**: Goes beyond single-theorem translation to formalize entire research papers with interdependent definitions, lemmas, and theorems -- a step toward machine-verifiable scientific literature.
- **Bidirectional translation**: The autoinformalization pipeline (Lean 4 -> LaTeX) closes the loop, enabling domain experts to verify semantic correctness without formal methods training. This is critical for adoption.
- **Transparent trust boundaries**: Explicit axiom declarations make the verification boundary visible, distinguishing what the machine has proven from what it has assumed. This is essential for scientific credibility.
- **Synthetic training data**: Verified (NL, formal code) pairs can train future models, creating a virtuous cycle of improving autoformalization capability.
- **Connects quantum computing to formal verification**: Brings the rigor of proof assistants to a field where correctness is critical (quantum error correction, fault tolerance) but formal verification has been rare.
- **Comparison to related agentic systems**: Distinguished from Ax-Prover (requires human guidance), Numina-Lean-Agent (required substantial human guidance for Putnam problems), and Agentic Proof Automation (required human-provided definitions/strategies) by achieving full automation.

---

## Paper 6: Lean4Physics -- Comprehensive Reasoning Framework for College-level Physics in Lean4

**Venue**: ICLR 2026 | **arXiv**: 2510.26094

### Problem

Formal reasoning research has focused almost exclusively on mathematics (miniF2F, ProofNet, PutnamBench). Physics remains largely unfformalized. Two specific gaps exist: (1) **lack of foundational infrastructure** -- unlike mathematics which has Mathlib, there is no established Lean 4 library for physics concepts (units, physical laws, dimensional analysis); and (2) **absence of benchmarks** -- no dataset exists to evaluate LLM capabilities in formal physics reasoning. This means we cannot measure or improve LLMs' ability to perform verified physics reasoning.

### Key Contributions

- **Lean4PHYS framework**: The first comprehensive system for college-level formal physics reasoning in Lean 4, combining a library and benchmark.
- **PhysLib**: A community-driven Lean 4 repository providing unit systems, fundamental physical theorems, and physics-specific infrastructure (dimensional analysis, unit conversion, physical quantity types).
- **LeanPhysBench**: The first formal physics benchmark, containing 200 hand-crafted problems across six physics topics and three difficulty levels.
- **Baseline evaluation**: Systematic assessment of 8 major LLMs (both general and math-specialized provers), revealing that expert math provers underperform general LLMs on physics tasks.
- **Key finding**: PhysLib provides an average 11.75% performance improvement across all models, demonstrating the importance of domain-specific infrastructure.

### Methodology

**Lean 4 Encoding of Physics**:
The formalization pipeline transforms natural language physics problems through four steps:
1. **Format alignment**: Convert question-answering format to proof statements (hypotheses -> conclusion)
2. **Condition extraction**: Identify physical laws and initial conditions as Lean 4 hypotheses
3. **Goal definition**: Specify proving targets as numerical values, expressions, or logical formulas
4. **Lean 4 formalization**: Encode with explicit type signatures, physical quantity types, and unit annotations

**Unit System (Foundation of PhysLib)**:
Built on Lean's UnitSystem kernel:
- Seven basic SI units: time (s), length (m), mass (kg), electric current (A), temperature (K), amount of substance (mol), luminous intensity (cd)
- Normed space definitions and algebraic operations for physical quantities
- Interchangeability proofs between physics quantities and mathematical real numbers
- Dimension casting mechanisms (e.g., converting Force to Mass * Acceleration)
- Example type signatures: `(m : Mass) (v : Speed) (t : Time) (ha : a = (v1 - v0)/dt)`

**Three-Layer PhysLib Hierarchy**:
1. **Layer 1 -- Foundation**: SI unit system, basic algebraic operations on units, dimensional analysis infrastructure
2. **Layer 2 -- Topic-specific units**: Specialized units for mechanics (Newton, Joule), thermodynamics (Kelvin, specific heat), electromagnetism (Coulomb, Volt), etc.
3. **Layer 3 -- Domain-specific theorems**: Formalized physics laws (Newton's laws, conservation of energy, Ohm's law, etc.) as Lean 4 theorems with proofs

### Architecture / Pipeline

```
Natural Language Physics Problem (from textbook/competition)
        |
        v
  [Human Expert Formalization]  -- Encode as Lean 4 theorem statement
        |                           with physical quantity types, unit annotations,
        |                           and hypotheses capturing physical laws
        v
  LeanPhysBench (200 problems across 6 topics, 3 difficulty levels)
        |
        v
  [LLM Inference]  -- Input: NL statement + Lean 4 theorem + PhysLib context (optional)
        |               Output: Lean 4 proof (tactic mode)
        v
  [Lean 4 Compiler]  -- Automatic verification of proof correctness
        |
        v
  Pass/Fail (verified or rejected)
```

**Six Physics Topics**:
1. Mechanics
2. Waves & Acoustics
3. Thermodynamics
4. Electromagnetism
5. Optics
6. Modern Physics

**Three Difficulty Levels**:
1. **College-level** (104 problems): University textbook problems emphasizing formula application and unit handling
2. **Competition-easy** (62 problems): Multi-step formula derivations, similar to traditional Lean math problems
3. **Competition-hard** (34 problems): Complex symbolic reasoning, functional relationships, integration, limits

### Experiments

**Models Tested (8 total)**:
- Open-source: DeepSeek-R1-8B, Qwen3-8B, Kimina-Prover-8B, Goedel-Prover-V2-8B, DeepSeek-Prover-V2-7B
- Closed-source: GPT-4o, Claude-Sonnet-4, Gemini-2.5-Pro

**Key Results (Pass@16)**:

| Model | Without PhysLib | With PhysLib | Delta |
|-------|-----------------|--------------|-------|
| Gemini-2.5-Pro | 7.5% | **39.5%** | +32.0% |
| Claude-Sonnet-4 | 2.0% | **34.5%** | +32.5% |
| GPT-4o | 3.0% | 20.0% | +17.0% |
| DeepSeek-Prover-V2-7B | 11.5% | 14.5% | +3.0% |
| Goedel-Prover-V2-8B | 10.0% | 13.0% | +3.0% |
| DeepSeek-R1-8B | 2.0% | 7.5% | +5.5% |
| Qwen3-8B | 1.5% | 6.5% | +5.0% |
| Kimina-Prover-8B | 4.5% | 6.5% | +2.0% |

**Critical Findings**:
1. **Expert math provers underperform general LLMs on physics**: DeepSeek-Prover-V2 achieves only 14.5% with PhysLib vs Gemini-2.5-Pro at 39.5%. Math-specialized provers may "overthink" unfamiliar domains.
2. **PhysLib is transformative**: Average 11.75% improvement; enables advanced tactics (simp, norm_num with physics-specific lemmas) that are unavailable without domain context.
3. **General LLMs benefit more from PhysLib**: Gemini and Claude gain 32+ percentage points, while expert provers gain only 2-3 points. This suggests general LLMs are better at leveraging new domain libraries.
4. **Difficulty matters**: Performance drops sharply on competition-hard problems requiring calculus, limits, and complex symbolic manipulation.
5. **Topic variation**: Models show better performance on mechanics and electromagnetism (better represented in training data) than thermodynamics or modern physics.

**Proof Strategy Differences**:
- Without PhysLib: Models resort to mechanical Mathlib tactics (simp, rw, exact, aesop) that often fail on physics problems
- With PhysLib: Models use structured approaches with `Scalar.val_inj` lemma for dimension casting, physics-specific simplification lemmas

### Limitations

1. **Expert math provers show limited transfer**: Despite similar Lean 4 syntax, math-specialized models struggle with physics formalization, suggesting fundamental differences in reasoning patterns.
2. **Hard competition problems remain largely unsolved**: Problems requiring calculus, limits, and complex symbolic manipulation defeat all tested models.
3. **Performance remains low overall**: Even the best model (Gemini-2.5-Pro) solves only 39.5% of problems, indicating substantial room for improvement.
4. **Manual benchmark creation**: The 200 problems were hand-crafted by experts, limiting scalability.
5. **College-level scope**: Does not cover graduate-level or research-level physics.
6. **PhysLib coverage**: The library covers six core topics but does not include all of physics (e.g., quantum mechanics, statistical mechanics, general relativity are absent or minimal).

### Connections to Broader Vision

- **Extends formal verification from math to science**: Lean4Physics is the first systematic effort to bring Lean 4's formal verification capabilities from pure mathematics into physics, establishing "a general principle for extending the formalization of physics and other natural sciences beyond mathematics into a verifiable system."
- **Bridges natural language and formal reasoning**: The formalization pipeline explicitly maps informal physics problem statements (from textbooks) to machine-verifiable Lean 4 proofs, closing the gap between how physicists communicate and how machines verify.
- **Domain-specific infrastructure is essential**: The dramatic performance improvement from PhysLib (avg +11.75%) demonstrates that general-purpose mathematical libraries (Mathlib) are insufficient for science -- domain-specific formal libraries are needed for each scientific field.
- **Reveals the transfer gap**: The finding that expert math provers underperform general LLMs on physics is significant for the broader vision: formalizing science is not simply "more math." Physics introduces units, dimensional analysis, physical intuition, and domain-specific reasoning patterns that require dedicated approaches.
- **Complements PhysLean/HepLean**: Sits alongside emerging efforts like PhysLean (high-energy physics formalization) and HepLean (particle physics), building toward a comprehensive formal physics ecosystem.
- **Community-driven approach**: Open-source PhysLib + LeanPhysBench encourages community contributions, similar to how Mathlib grew for mathematics.

---

## Cross-Paper Analysis and Synthesis

### Common Themes

1. **Autoformalization as the core challenge**: All three papers tackle the translation of informal scientific knowledge into formal, machine-verifiable representations. PDE-Controller uses STL, MerLean and Lean4Physics use Lean 4.

2. **LLMs as translation engines**: In all cases, LLMs serve as the bridge between human-readable scientific text and formal code. The key insight is that LLMs can handle the "soft" translation task that requires understanding both natural language and formal syntax.

3. **Iterative refinement with compiler feedback**: Both MerLean (30-attempt compile-fix loop) and PDE-Controller (multi-stage pipeline with Gurobi verification) use external verification to ground LLM outputs. Lean4Physics uses Lean 4's type checker as the verifier. This pattern of "generate-then-verify" with formal tools is universal.

4. **Domain-specific infrastructure matters**: Lean4Physics shows that Mathlib alone is insufficient (+11.75% from PhysLib). MerLean finds gaps in Mathlib for quantum-specific constructs. PDE-Controller builds its own STL representation. Each domain needs its own formal infrastructure.

5. **Scaling from single theorems to full papers/systems**: MerLean formalizes entire research papers (114 statements, 41K lines). PDE-Controller handles full control pipelines. Lean4Physics benchmarks 200 problems. The field is moving beyond toy examples toward research-scale formalization.

### Key Differences

| Aspect | PDE-Controller | MerLean | Lean4Physics |
|--------|---------------|---------|--------------|
| **Domain** | Applied math (PDE control) | Quantum computation | College physics |
| **Formal language** | Signal Temporal Logic (STL) | Lean 4 + Mathlib | Lean 4 + PhysLib |
| **Scale** | 2.13M synthetic + 34 real | 114 statements, 41K lines | 200 benchmark problems |
| **Automation level** | Fully automated pipeline | Fully automated (no human in loop) | LLM generates proofs; humans create problems |
| **Verification** | Gurobi MILP solver | Lean 4 type checker | Lean 4 type checker |
| **Key innovation** | DPO-trained reasoning via subgoals | Full-paper agentic formalization with MCP tools | Physics-specific Lean 4 infrastructure |
| **Best performance** | 99.2% IoU (synthetic), 62% utility gain | 42 hours for 3 papers | 39.5% Pass@16 (Gemini-2.5-Pro) |

### Implications for the Broader Vision

Together, these papers suggest a clear trajectory for formalizing scientific knowledge:

1. **Formal languages for science are emerging but fragmented**: STL for PDEs, Lean 4 for math/physics/quantum -- there is no unified formal framework for all of science yet. Each domain develops its own representation.

2. **LLMs are ready for science formalization**: All three papers demonstrate that current LLMs (Claude, GPT-4o, Gemini, DeepSeek) can perform meaningful autoformalization of scientific content, though with significant room for improvement.

3. **The bottleneck is infrastructure, not models**: PhysLib's dramatic impact, MerLean's Mathlib gaps, and PDE-Controller's need for custom STL representation all point to the same conclusion -- we need more formal libraries for science, not just better LLMs.

4. **Bidirectional translation enables adoption**: MerLean's autoinformalization (Lean -> LaTeX) and PDE-Controller's NL -> formal -> code pipeline both demonstrate that the value of formalization is amplified when results can be communicated back to domain experts in their native language.

5. **Verification is the key differentiator**: Unlike NL-based reasoning where correctness is approximate, formal verification (via Lean or Gurobi) provides mathematical guarantees. This is especially valuable in safety-critical domains (quantum error correction, PDE control in engineering).

### Open Questions

- Can these approaches scale to graduate-level and research-frontier physics?
- How do we handle the axiom gap (MerLean's 9.1% axiomatization rate) as we move to more advanced topics?
- Can a unified formal framework cover physics, chemistry, and biology, or will each discipline need its own?
- How do we close the transfer gap between math provers and science provers (Lean4Physics finding)?
- Can the synthetic data from autoformalization (MerLean's verified pairs, PDE-Controller's 2.13M samples) bootstrap better models in a virtuous cycle?
