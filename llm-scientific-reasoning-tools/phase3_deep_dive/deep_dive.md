# Phase 3 Deep Dive: Equations-to-Code Papers

**Research context**: Survey on using LLMs to transform scientific knowledge (math equations, physics) into human-understandable formats (knowledge trees, code, natural language).

**Papers reviewed**:
1. LLM-SR (ICLR 2025 Oral)
2. SymCode (EACL 2026)
3. CodePDE (TMLR)

**Date**: 2026-03-10

---

## Paper 1: LLM-SR -- Scientific Equation Discovery via Programming with Large Language Models

**Citation**: arXiv 2404.18400, ICLR 2025 Oral

### Problem

LLM-SR tackles **data-driven equation discovery** (symbolic regression): finding concise, interpretable symbolic mathematical expressions that approximate unknown equations from observational data. This is an NP-hard problem requiring navigation through extremely high-dimensional combinatorial and nonlinear hypothesis spaces. Traditional symbolic regression methods (genetic programming, deep SR) fail to incorporate domain-specific scientific knowledge that human scientists naturally leverage. LLM-SR's core insight is that LLMs possess embedded scientific priors and strong code generation capabilities that can be harnessed for equation discovery.

### Key Contributions

- **Equations-as-programs representation**: Represents candidate equations as executable Python programs rather than expression trees, enabling differentiable parameter optimization via gradient-based methods (BFGS, Adam). This decouples structure discovery (LLM) from coefficient optimization (numerical optimizer).
- **LLM-powered evolutionary search**: Combines LLM scientific priors with an islands-model evolutionary search, where LLMs generate equation skeletons guided by in-context examples from a fitness-ranked experience buffer.
- **Superior performance with 1000x fewer iterations**: Achieves 100--1000x lower normalized MSE than baselines (PySR, uDSR, DSR, GPlearn) while requiring only ~2,500 iterations vs. 2M+ for baselines.
- **Out-of-domain generalization**: Demonstrates significantly better OOD generalization, attributed to LLM scientific knowledge guiding discovery toward physically meaningful equation forms rather than overfitting artifacts.
- **Novel benchmarks**: Introduces challenging problems across three scientific domains (nonlinear oscillators, bacterial growth, material stress-strain) designed to test genuine reasoning rather than memorization.

### Methodology

**Core mechanism -- Skeleton/parameter separation**:
1. The LLM generates equation *skeletons* containing placeholder parameters (`params[0]`, `params[1]`, etc.) as Python functions.
2. Numerical optimizers (BFGS or Adam) fit the placeholder parameters to data.
3. This separation lets the LLM focus on the structural/functional form while robust optimizers handle precise coefficient tuning.

**Iterative refinement loop** (three stages per iteration):
- **(a) Hypothesis generation**: LLM samples b=4 candidate equation skeletons (temperature=0.8) from a structured prompt containing problem specification, evaluation criteria, and in-context examples of prior high-scoring equations.
- **(b) Data-driven evaluation**: Each skeleton is instantiated with random initial parameters, optimized for 30 seconds via BFGS or Adam, and scored by negative MSE.
- **(c) Experience management**: High-scoring equations are added to an islands-model experience buffer (m=10 islands). Equations are clustered by fitness score; selection uses Boltzmann distribution favoring high scores while preserving diversity.

**Experience sampling** (for prompt construction): Two-stage process -- (1) randomly select one island, (2) sample k=2 equation programs from that island using score-based probability favoring shorter expressions. These become in-context demonstrations for the next iteration.

**Island management**: Every 4 hours, the worst-performing half of islands (m/2) are reinitialized with copies from surviving populations, preventing stagnation in local optima.

### Architecture/Pipeline

```
INPUT: Dataset D = {(x_i, y_i)}, problem description T

Initialize m=10 islands with simple linear equation skeleton
FOR t = 1 to T=2500:
  1. EXPERIENCE SAMPLING: Select island, sample k=2 high-scoring equations
  2. PROMPT CONSTRUCTION: Combine problem spec + eval function + sampled examples
  3. LLM GENERATION: Sample b=4 equation skeletons (temp=0.8)
     - Discard programs that fail execution or exceed 30s timeout
  4. PARAMETER OPTIMIZATION: For each valid skeleton:
     - Initialize params randomly
     - Optimize via BFGS or Adam (30s budget)
     - Score: s = -MSE(f(x, params*), y)
  5. BUFFER UPDATE: Add (skeleton, score) to originating island
     - Cluster by score signature for diversity
  6. ISLAND RESET: Every 4 hours, reinitialize worst m/2 islands

OUTPUT: Best equation f* with optimized parameters
```

### Experiments

**Datasets/Benchmarks** (all novel, designed to prevent memorization):
- **Nonlinear Oscillators (2 variants)**: Complex ODEs with driving force, damping, and restoring terms. OOD test on time range [20, 50) after training on [0, 50).
- **E. coli Bacterial Growth**: Multi-variable growth model with nonlinear temperature and pH dependencies. Variables: population density, substrate, temperature, pH.
- **Material Stress-Strain**: Real-world aluminum 6061-T651 tensile test data across 6 temperatures (20--300C). OOD test: T=200C held out entirely. Based on Johnson-Cook model.

**Baselines**: GPlearn (GP, 2M generations), DSR (RNN-based RL, 2M iterations), uDSR (Transformer+GP hybrid, 2M iterations), PySR (async multi-island GP, 2M iterations).

**Key results (Normalized MSE)**:

| Problem | Best Baseline (ID) | LLM-SR GPT-3.5 (ID) | Best Baseline (OOD) | LLM-SR GPT-3.5 (OOD) |
|---------|-------------------|---------------------|--------------------|-----------------------|
| Oscillation 1 | 0.0003 (uDSR) | 4.65e-7 | 0.0007 (uDSR) | 0.0005 |
| Oscillation 2 | 0.0002 (PySR) | 2.12e-7 | 0.0015 (uDSR) | 3.81e-5 |
| E. coli | 0.0376 (PySR) | 0.0214 | 1.0141 (PySR) | 0.0264 |
| Stress-Strain | 0.0331 (PySR) | 0.0210 | 0.1304 (PySR) | 0.0516 |

- LLM-SR achieves 100--1000x lower in-domain NMSE on oscillation problems.
- OOD advantage is dramatic: E. coli OOD is 0.0037 (Mixtral) vs. best baseline 1.0141 -- a 274x improvement.
- Convergence within ~500--1000 iterations (sharp drops in error), while baselines show gradual improvement over millions of iterations.

**Ablation study (Oscillation 2, GPT-3.5)**:

| Variant | ID NMSE | OOD NMSE |
|---------|---------|----------|
| Full LLM-SR | 2.12e-7 | 3.81e-5 |
| Without problem specification | 4.65e-5 | 7.10e-3 (220x worse) |
| Without iterative refinement | 1.01e-1 | 1.81e-1 |
| Without coefficient optimization | 3.75e-1 | 3.78e-1 |
| numpy+BFGS (default) | 2.12e-7 | 3.81e-5 |
| pytorch+Adam | 1.60e-6 | 2.4e-4 |

All three components (problem description, iterative refinement, parameter optimization) are essential. Problem specification alone provides 220x improvement, confirming that LLM scientific priors are genuinely leveraged.

**Qualitative analysis**: LLM-SR reconstructs physically interpretable equation structures (identifying driving force, damping, restoring force components), while baselines produce opaque combinations of mathematical operations lacking physical meaning.

### Limitations

1. **Computational cost**: Loading LLMs and generating large numbers of equation programs can be substantial, especially for complex problems.
2. **Knowledge bias**: LLM scientific priors may be limited, biased, or incorrect in certain domains, potentially degrading equation quality.
3. **Memorization risk on standard benchmarks**: On Feynman benchmark equations, LLM-SR achieves near-zero error within few iterations, likely through memorization rather than discovery. This motivated the novel benchmark design.
4. **Domain dependence**: Effectiveness depends on the LLM's embedded knowledge for the specific scientific domain being studied.
5. **No end-to-end coefficient learning**: Requires a separate parameter optimization step via external numerical optimizers.

### Connections to Survey Theme

LLM-SR exemplifies the **math-to-code** direction: scientific equations are literally represented as executable Python programs. The LLM acts as a translator between scientific domain knowledge and formal code, while numerical optimizers ground the code in observational data. This is a direct instance of "LLMs transforming scientific knowledge into code." The skeleton/parameter separation is a key architectural pattern: LLMs handle the qualitative structure (what physical phenomena matter) while numerical methods handle the quantitative details (exact coefficient values). The in-context learning loop with fitness feedback creates a knowledge refinement cycle where scientific priors are iteratively corrected by data.

---

## Paper 2: SymCode -- A Neurosymbolic Approach to Mathematical Reasoning via Verifiable Code Generation

**Citation**: arXiv 2510.25975, EACL 2026

### Problem

LLMs struggle with complex mathematical reasoning when generating prose-based solutions (Chain-of-Thought). Such solutions contain arithmetic errors, logical fallacies, and hallucinated steps that are opaque and difficult to verify. SymCode reframes mathematical problem-solving as **verifiable code generation** using symbolic computation, eliminating both the ambiguity of natural language reasoning and arithmetic errors through the SymPy computer algebra system. The key insight is that code itself can serve as the reasoning trace -- not just as a calculator for intermediate steps (as in PAL), but as the complete formal reasoning artifact.

### Key Contributions

- **Reasoning-as-code framework**: A prompt-based neurosymbolic system where LLMs generate self-contained Python scripts using SymPy that embody the entire reasoning process, not just calculations.
- **Self-debugging loop (SymCode+)**: An iterative correction mechanism where execution errors (syntax errors, assertion failures) produce structured feedback that the LLM uses to debug and regenerate code. This is uniquely enabled by the code representation -- prose-based methods lack comparable deterministic correction signals.
- **Up to 13.6 percentage point accuracy gains**: Over baselines (CoT, ToT) on MATH-500, OlympiadBench, and AIME 2024/2025, with larger gains on harder problems.
- **60--77% token reduction**: Code-based reasoning is far more compact than prose alternatives.
- **Error type shift**: Failures shift from opaque logical errors (in prose) toward transparent programmatic errors with clearer debugging paths.

### Methodology

**Core principle -- Code as reasoning modality**: Rather than asking the LLM to explain reasoning in prose and then optionally compute, SymCode instructs it to write a complete Python script where every reasoning step is a code operation with explanatory comments. The code IS the reasoning.

**Six-component prompt structure**:
1. **Symbolic formulation**: Mandatory `import sympy as sp`. SymPy manipulates expressions in exact symbolic form (sqrt(2) precisely, not 1.414...), preventing rounding errors and enabling true algebraic reasoning.
2. **Interpretability**: Step-by-step comments explaining reasoning while maintaining formal structure. Preserves "show your work" benefits while grounding in verifiable code.
3. **Problem scaffolding**: Define symbols with mathematical assumptions (`sp.symbols('x', positive=True, integer=True)`), formalizing constraints upfront to reduce the solution search space.
4. **Intermediate reasoning**: Each step includes comments explaining mathematical logic, using meaningful variable names.
5. **Verification and filtering**: Runtime assertions check that solutions satisfy original equations and domain requirements. Failed assertions halt execution and trigger the self-debugging loop. Final answers printed in LaTeX boxed format.
6. **Execution pipeline**: Script runs in a sandboxed Python interpreter providing deterministic pass/fail signals.

**Neurosymbolic integration**:
- Neural component (LLM): Interprets problem nuances, translates natural language to formal specifications, selects high-level reasoning strategy.
- Symbolic component (SymPy): Executes algebraic operations deterministically, provides exact computation, enables verifiable equation solving.

**SymCode+ self-debugging loop**: If script execution fails (SyntaxError, TypeError, AssertionError), the error message and traceback are appended to the prompt history. The LLM receives "Debug the following code based on the provided error message." This repeats up to 2-3 attempts. This mechanism is unique to code-based approaches.

### Architecture/Pipeline

```
INPUT: Natural language math problem

1. PROMPT CONSTRUCTION: SymCode prompt template + problem text
2. LLM CODE GENERATION: Generate complete Python script using SymPy
3. SANDBOXED EXECUTION: Run in Python interpreter
   |
   +---> SUCCESS: Extract answer in LaTeX box format --> OUTPUT
   |
   +---> FAILURE: Capture error message + traceback
         |
         +---> Append error to prompt history
         +---> LLM generates debugged version
         +---> Re-execute (up to N iterations)
         +---> If all fail: return best attempt or no answer
```

### Experiments

**Datasets**:
- **MATH-500**: 500 challenging high-school competition problems (algebra, geometry, number theory, precalculus).
- **OlympiadBench**: 674 national/international olympiad problems requiring creative formal reasoning.
- **AIME 2024 & 2025**: 60 recent competition problems bridging high-school and olympiad difficulty.

**Models evaluated**: Llama 3.2 (90B), GPT-5-nano, GPT-OSS (20B reasoning-optimized).

**Baselines**: Chain of Thought (CoT), Tree of Thoughts (ToT), Decomposition.

**Key results**:

| Dataset | Model | CoT | ToT | SymCode | SymCode+ |
|---------|-------|-----|-----|---------|----------|
| MATH-500 | Llama 3.2 (90B) | 61.2% | 63.8% | 64.4% | **68.8%** |
| OlympiadBench | Llama 3.2 (90B) | 34.4% | 36.8% | 31.2% | **36.8%** |
| AIME 24-25 | Llama 3.2 (90B) | 20.0% | 23.3% | 25.0% | **31.7%** |
| MATH-500 | GPT-5-nano | 93.4% | 88.2% | 90.8% | **91.4%** |
| OlympiadBench | GPT-5-nano | 63.2% | 68.0% | 76.8% | **80.0%** |
| AIME 24-25 | GPT-5-nano | 51.6% | 51.7% | 61.7% | **65.0%** |
| MATH-500 | GPT-OSS (20B) | 88.4% | 87.0% | 90.4% | **93.2%** |
| OlympiadBench | GPT-OSS (20B) | 22.4% | 24.8% | 35.2% | **38.4%** |
| AIME 24-25 | GPT-OSS (20B) | 11.6% | 10.0% | 18.3% | **21.6%** |

- SymCode+ achieves up to **+13.3 pp** over best prose baseline (GPT-5-nano on AIME).
- Advantages increase on harder problems (AIME > OlympiadBench > MATH-500).
- Code-oriented models (GPT-5-nano) benefit most from SymCode framework.

**Token efficiency**:
- SymCode: ~699 output tokens
- ToT: ~1,770 tokens
- Decomposition: ~1,962 tokens
- CoT: ~2,991 tokens
- Result: 60--77% token reduction

**Error type analysis** (GPT-5-nano on AIME):
- ToT failures: 41.4% arithmetic mistakes, 34.5% logical fallacies, 24.1% other
- SymCode failures: 56.2% problem misinterpretation, 31.3% incorrect API usage, 12.5% other
- Interpretation: SymCode eliminates arithmetic and logic errors entirely, shifting failures to transparent categories.

**Ablation study** (GPT-5-nano on AIME):

| Variant | Accuracy |
|---------|----------|
| SymCode+ (full) | **65.0%** |
| No self-debug | 61.7% |
| No verification (assertions) | 58.5% |
| No SymPy (numeric Python only) | 48.3% |

SymPy alone contributes ~17 pp. Each component is critical.

**Self-debugging activation rates**: >28% of problems require correction for Llama 3.2 (90B). Higher activation correlates with weaker coding fluency but also with larger SymCode-to-SymCode+ gains.

### Limitations

1. **Coding proficiency dependency**: Performance is heavily influenced by the base LLM's code generation abilities. Strong code-oriented models significantly outperform weaker ones even with self-debugging.
2. **Formalizable problems only**: Most effective on problems directly expressible symbolically. Struggles with abstract reasoning tasks (synthetic geometry proofs, induction/contradiction arguments, combinatorial reasoning that resists SymPy formalization).
3. **Library reliability**: Depends on correctness of Python interpreter and SymPy -- while mature, neither is formally verified.
4. **Prompt sensitivity**: Performance is sensitive to prompt engineering; no robustness evaluation across instruction format variations.
5. **Verification quality**: Correctness depends on LLM-generated assertion quality; weak or missing assertions in unexecuted code paths may miss errors.

### Connections to Survey Theme

SymCode represents the **math-reasoning-to-code** direction: mathematical problem-solving is literally transformed into verifiable code generation. The neurosymbolic architecture is a clean example of the "knowledge tree" concept -- the neural LLM provides the high-level reasoning structure while the symbolic CAS provides the computational leaves. The self-debugging loop demonstrates how code as an intermediate representation enables verification and correction cycles impossible with prose-based reasoning. This directly supports the survey thesis that code serves as a bridge between mathematical knowledge and human-understandable, machine-verifiable explanations.

---

## Paper 3: CodePDE -- An Inference Framework for LLM-driven PDE Solver Generation

**Citation**: arXiv 2505.08783, TMLR (submitted May 2025, revised February 2026)

### Problem

Traditional numerical PDE solvers demand substantial computational resources and deep domain expertise, requiring meticulous manual tuning and extensive debugging. Neural PDE solvers (PINNs, FNO, DeepONet) require large training datasets and lack interpretability. CodePDE reframes PDE solving as a **code generation task**: LLMs generate executable solver implementations directly from natural-language PDE descriptions, combining the expressiveness of numerical methods with the accessibility of natural language interfaces. The key insight is that code acts as "a versatile intermediary between natural language and structured symbolic scientific computations."

### Key Contributions

- **First inference framework for LLM-driven PDE solver generation**: A five-step pipeline (task specification, code generation, debugging, evaluation, refinement) combining PDE domain knowledge with LLM self-improvement and test-time scaling.
- **Performance matching tailored numerical solvers**: With inference-time techniques, LLMs achieve accuracy comparable to -- and sometimes exceeding -- expert-crafted numerical solvers across diverse PDE families.
- **Comprehensive empirical study**: Evaluates 16 LLMs across four core capabilities (reasoning, debugging, refinement, test-time scaling) with novel insights into reliability-vs-sophistication tradeoffs.
- **Interpretability advantage over neural solvers**: "Unlike neural solvers, LLMs produce human-readable code that exposes the model's reasoning," enabling diagnosis of specific numerical scheme failures.
- **Open-source platform**: Released codebase integrated into Terminal Bench for systematic LLM evaluation on solver generation.

### Methodology

**Problem formulation**: CodePDE targets PDEs in general form: L u(x) = phi(x) on domain Omega, with boundary conditions B u(x) = psi(x) on the boundary. The goal is to generate correct, efficient Python solver code.

**Five-step framework**:

**Step 1 -- Task Specification**: PDEs formatted into structured natural-language descriptions including governing equations, domain specifications, boundary conditions, and initial conditions.

**Step 2 -- Code Generation**: LLMs use chain-of-thought prompting to generate complete solver implementations with a predefined function signature: `solver(u0_batch, t_coordinate, parameters)` returning solution trajectories of shape [batch_size, T+1, N]. Models can explore multiple numerical methods: finite difference, finite volume, spectral methods for spatial discretization; Explicit Euler, Runge-Kutta, IMEX for time integration. The choice of numerical scheme is left to the LLM.

**Step 3 -- Debugging (Automated Error Correction)**: If the solver crashes, error traces, the original specification, and the failed code are fed back to the LLM. The LLM diagnoses root causes and produces corrected implementations. Iterates up to 4 rounds. Single-shot generation yields only 41% bug-free solvers; debugging increases this to 84%.

**Step 4 -- Evaluation**: Three metrics:
- **nRMSE** (Normalized Root Mean Squared Error): Scale-independent accuracy measure.
- **Convergence test**: Whether error decreases with grid refinement (||u_h - u_{h/2}||_2 approaches 0 as h approaches 0).
- **Execution time**: Computational efficiency.

**Step 5 -- Solver Refinement**: nRMSE scores and solver code are supplied back to the LLM. The model analyzes execution results to identify bottlenecks and numerical instabilities, then generates improved implementations. Best-of-12 sampling selects the lowest nRMSE variant.

**Test-time scaling**: Best-of-n sampling with n in [4, 32]. Most significant gains observed between n=4 and n=16; beyond n=16, diminishing returns. Moderate sampling budgets (~16) often sufficient.

### Architecture/Pipeline

```
INPUT: Natural-language PDE description
  |
  v
[Step 1] TASK SPECIFICATION
  Format PDE into structured description:
  governing equations, domain, BCs, ICs
  |
  v
[Step 2] CODE GENERATION (chain-of-thought prompting)
  LLM generates complete Python solver
  Function signature: solver(u0_batch, t_coordinate, parameters)
  LLM chooses numerical method (FD, spectral, FV, etc.)
  |
  v
[Step 3] DEBUGGING (up to 4 rounds)
  Execute solver --> if crash:
    Feed error trace + code + spec back to LLM
    LLM diagnoses and fixes
    Re-execute
  41% -> 84% bug-free rate
  |
  v
[Step 4] EVALUATION
  Compute nRMSE against reference solutions
  Run convergence test (grid refinement)
  Measure execution time
  |
  v
[Step 5] REFINEMENT
  Feed nRMSE + code back to LLM
  LLM identifies numerical instabilities
  Generates improved solver
  Best-of-12 sampling for lowest nRMSE
  |
  v
OUTPUT: Optimized executable Python PDE solver
```

### Experiments

**Datasets -- Five PDE families (100 instances each)**:
1. **Advection**: Basic transport equation (simple).
2. **Burgers' equation**: Nonlinear PDE combining convection and diffusion (moderate).
3. **Reaction-Diffusion**: Pattern formation equation (challenging -- all LLMs struggle).
4. **Compressible Navier-Stokes (CNS)**: Multi-dimensional fluid dynamics (high complexity).
5. **Darcy Flow**: Porous media, steady-state problem.

Source: PDEBench and Fourier Neural Operator (FNO) paper.

**Models evaluated (16 total)**:
- Proprietary: GPT-4o, GPT-4.1, o3, Gemini 2.0 Flash, Gemini 2.0 Thinking, Gemini 2.5 Pro, Claude 3.5 Haiku, Claude 3.7 Sonnet
- Open-weights: DeepSeek-V3, DeepSeek-R1, Qwen-2.5-Max, QwQ-Plus

**Baselines**:
- Reference numerical solvers (analytical solutions, spectral methods, Strang splitting)
- Neural PDE solvers: FNO (nRMSE 7.70e-3 to 9.80e-3), PINNs (7.80e-3 to 8.50e-1), U-Net (5.00e-2 to 3.60e-1)
- Foundation models: ORCA, PDEformer, UPS
- Agentic workflows: FunSearch, AIDE

**Key results (nRMSE, lower is better)**:

| PDE | Reference Solver | Best CodePDE (Debug) | Best CodePDE (Refine) |
|-----|-----------------|---------------------|---------------------|
| Advection | 1.03e-3 | 1.01e-3 | **9.74e-4** |
| Burgers | 3.55e-4 | 1.23e-4 | **1.06e-4** |
| Reaction-Diffusion | 2.29e-3 | 1.49e-1 | 1.74e-2 |
| CNS | 1.89e-2 | 2.41e-2 | **1.31e-2** |
| Darcy | 4.80e-3 | 4.80e-3 | **4.78e-3** |

- CodePDE with refinement **outperforms reference solvers on 4 out of 5 PDEs**.
- Burgers: 1.06e-4 vs reference 3.55e-4 (70% improvement via Gemini 2.0 Flash).
- Reaction-Diffusion: persistent challenge. All LLMs struggle because they default to finite-difference for the reaction term, while the reference solver exploits an analytical solution for that component.

**Ablation study (Claude 3.7 Sonnet)**:

| Variant | nRMSE |
|---------|-------|
| Full CodePDE | 4.44e-3 |
| Without refinement | 5.15e-3 (+16%) |
| Without refinement + scaling | 8.68e-2 (+1,854%) |
| Without refinement + scaling + debug | 1.49e-1 (+3,258%) |

Each component (debugging, scaling, refinement) is critical.

**Debugging effectiveness**:
- Single-shot: 41% bug-free rate across all PDEs
- After debugging: 84% bug-free rate
- Frontier models post-debugging: >90% bug-free (Gemini 2.5 Pro, Claude 3.7 Sonnet, DeepSeek-R1, GPT-4.1, o3)
- CNS (hardest): 60.5% average convergence failure rate

**Test-time scaling**: Best-of-n sampling shows most significant gains between n=4 and n=16. Beyond n=16, diminishing returns.

**Numerical scheme diversity**: LLMs choose different methods -- most default to finite difference; DeepSeek-R1 favors spectral methods; o3 exhibits the most diversity in scheme choices, indicating better exploration ability.

**Reaction-Diffusion failure analysis**: LLMs apply finite-difference to the reaction term while the reference exploits an analytical solution. This highlights the interpretability advantage: because the solver is code, the exact failure mode (wrong numerical scheme for a specific subcomponent) can be diagnosed precisely.

### Limitations

1. **Reaction-Diffusion challenge**: All LLMs and agentic methods consistently struggle. nRMSE ~0.017--0.22 vs reference 0.0023. Root cause: inability to recognize analytical solutions for problem subcomponents.
2. **Code quality variability**: Single-shot generation highly unreliable (41% success). Requires multiple refinement rounds and test-time compute.
3. **Reasoning vs. refinement gap**: Advanced reasoning models lead to better solvers in the reasoning+debugging stage but "not necessarily better in the refinement stage," suggesting these are distinct capabilities not fully developed in current models.
4. **Complex PDE limitations**: CNS shows 60.5% average convergence failure rate on single-shot generation.
5. **Computational cost**: Test-time scaling (n=16--32 samples) and iterative refinement require significant inference budget.

### Connections to Survey Theme

CodePDE is a direct instance of **physics-to-code**: translating PDE specifications (mathematical physics) into executable solver code via LLMs. The key insight for the survey is the *interpretability argument* -- unlike neural solvers (black boxes), LLM-generated code exposes the reasoning process. When a solver fails, one can read the code to diagnose exactly which numerical scheme was chosen and why it fails (e.g., finite difference vs. analytical solution for the reaction term). This is the "human-understandable format" aspect of the survey's thesis realized in the PDE domain.

---

## Cross-Paper Analysis and Synthesis

### Shared Architectural Patterns

All three papers share a remarkably similar meta-architecture despite targeting different problems:

| Component | LLM-SR | SymCode | CodePDE |
|-----------|--------|---------|---------|
| **Representation** | Equations as Python functions | Math reasoning as Python+SymPy scripts | PDE solvers as Python functions |
| **Generation** | LLM generates equation skeletons | LLM generates complete scripts | LLM generates solver implementations |
| **Evaluation** | MSE against data | Execution + assertions | nRMSE + convergence tests |
| **Refinement** | Evolutionary buffer + re-prompting | Self-debugging loop | nRMSE feedback + regeneration |
| **Iterations** | ~2,500 | 2--3 debug rounds | 4 debug + 12 refinement rounds |

The common pattern is: **LLM generates code --> execute/evaluate --> feed results back to LLM --> iterate**. This generate-evaluate-refine loop is the fundamental building block across all three systems.

### Code as the Universal Bridge

All three papers converge on code as the intermediate representation between scientific/mathematical knowledge and computation:

- **LLM-SR**: Scientific knowledge (equation structure) --> Python function --> numerical optimization
- **SymCode**: Math problem understanding --> SymPy script --> symbolic computation --> verified answer
- **CodePDE**: PDE specification --> Python solver --> numerical solution

Code serves three simultaneous roles:
1. **Executable**: Can be run to produce concrete results
2. **Verifiable**: Execution provides pass/fail signals (assertions, convergence tests, MSE scores)
3. **Interpretable**: Human-readable, enabling diagnosis of failures and understanding of reasoning

### Key Differentiators

| Dimension | LLM-SR | SymCode | CodePDE |
|-----------|--------|---------|---------|
| **Direction** | Data --> equations (discovery) | Problems --> solutions (reasoning) | Equations --> solvers (implementation) |
| **LLM role** | Hypothesis generator | Complete reasoner | Code architect |
| **Verification** | Fitness score on held-out data | Assertion-based runtime checks | nRMSE + convergence analysis |
| **Knowledge source** | LLM scientific priors + data | LLM math understanding + SymPy | LLM numerical methods knowledge |
| **Novelty axis** | Discovers new equations | Solves known problems better | Generates implementations of known methods |

### Implications for the Survey Vision (Math --> Code --> Explanation)

These three papers map cleanly onto three stages of the survey's thesis:

1. **LLM-SR** addresses **discovery**: LLMs discover mathematical relationships from data, encoding them as code. This is the "scientific knowledge generation" step.

2. **SymCode** addresses **reasoning and verification**: LLMs translate mathematical problems into verifiable code, producing not just answers but auditable reasoning traces. This is the "knowledge validation" step.

3. **CodePDE** addresses **implementation and interpretation**: LLMs translate known mathematical physics into executable solvers, making PDE solving accessible to non-experts. This is the "knowledge operationalization" step.

Together, they demonstrate a complete pipeline: discover equations (LLM-SR) --> reason about their properties (SymCode) --> implement solvers for them (CodePDE), all mediated by code as the universal representation.

### Common Limitations Across All Three

1. **LLM knowledge boundaries**: All three systems are limited by what the LLM "knows." LLM-SR may propose wrong equation forms for unfamiliar domains. SymCode cannot formalize problems outside SymPy's scope. CodePDE defaults to finite-difference even when analytical solutions exist.

2. **Verification gaps**: None achieve formal mathematical verification. LLM-SR uses data fitness (could overfit). SymCode relies on LLM-generated assertions (could be incomplete). CodePDE uses reference solution comparison (requires ground truth).

3. **Computational cost**: All require significant inference-time compute. LLM-SR: 2,500 LLM calls. SymCode: 2--3 rounds per problem. CodePDE: up to 32 samples + 12 refinement rounds.

4. **Code quality as bottleneck**: The effectiveness of all three depends critically on the LLM's code generation quality. Better code-generating LLMs consistently produce better results across all three systems.

### Open Directions

- **Integration**: Could LLM-SR discover an equation, SymCode verify its properties, and CodePDE generate a solver for it -- all in a unified pipeline?
- **Formal verification**: Replacing runtime assertions (SymCode) and convergence tests (CodePDE) with formal proofs (e.g., Lean4) would close the verification gap.
- **Knowledge trees**: The equation skeletons (LLM-SR), commented code (SymCode), and numerical scheme choices (CodePDE) all contain structured knowledge that could be organized into navigable knowledge trees for educational or explanatory purposes.
- **Natural language explanations**: All three produce code, but none systematically generate natural-language explanations of the code's meaning. Adding an "explain this code in plain English" step would complete the math --> code --> explanation pipeline.
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
# Phase 3 Deep Dive: Knowledge Representation and Interactive Learning with LLMs

## Papers Covered
- Paper 7: Towards an AI-Augmented Textbook (Learn Your Way / LearnLM)
- Paper 8: Mindalogue: LLM-Powered Nonlinear Interaction for Effective Learning and Task Exploration
- Paper 9: AutoMathKG: The Automated Mathematical Knowledge Graph Based on LLM and Vector Database

---

## Paper 7: Towards an AI-Augmented Textbook (Learn Your Way / LearnLM)

**Citation**: LearnLM Team, Google. "Towards an AI-Augmented Textbook." arXiv:2509.13348, 2025.
**Venue**: Google Research (pre-print)
**Blog**: https://research.google/blog/learn-your-way-reimagining-textbooks-with-generative-ai/

### Problem

Traditional textbooks are a one-size-fits-all medium. Any new representation or adaptation requires arduous human effort, making it impossible to personalize textbooks at scale. Learners have diverse grade levels, interests, and preferred modalities, but a single static textbook cannot accommodate these differences. The core question: can generative AI transform textbooks into personalized, multi-representation learning experiences while maintaining content integrity?

### Key Contributions

- **Multi-representation generation pipeline**: Transforms a single textbook chapter into five distinct formats (immersive text, slides with narration, audio-graphic lesson, mind maps, quizzes) using Gemini 2.5 Pro without fine-tuning (except for visual illustrations).
- **Two-axis personalization**: Adapts content along (1) grade level (via Flesch-Kincaid Grade re-leveling) and (2) learner interests (e.g., sports, music, food), where interest-relevant segments are selectively replaced while preserving ~80% of original structure.
- **Pedagogical validation**: Expert evaluation across 10 rubric dimensions (accuracy, coverage, engagement, cognitive load, metacognition, etc.) with all components scoring >0.90.
- **RCT-based efficacy evidence**: Randomized controlled trial with 60 students (ages 15-18) showing statistically significant improvements in both immediate (p=0.03) and 3-day retention assessments (p=0.03).

### Methodology

**Two-stage pipeline:**

**Stage 1 -- Text Personalization:**
- Grade-level adaptation uses Flesch-Kincaid Grade readability metric. Three levels tested: 7th grade, 10th grade, undergraduate.
- Interest personalization uses a "focused" approach: the system identifies amenable text segments, replaces only those portions with interest-mapped analogies (e.g., Newton's Third Law rewritten for basketball or art), and highlights personalized text visually.

**Stage 2 -- Content Transformations (Multiple Representations):**

1. **Immersive Text (Core View)**: Personalized comprehensive text with embedded enhancements:
   - *Timeline component*: Identifies sequences (historical, experimental, algorithmic), generates visual timelines with interactive drag-and-drop practice.
   - *Memory aids*: Generates mnemonics with semantic association to material.
   - *Visual illustrations*: Fine-tuned model for pedagogical images (standard image generators trained for realism fail at simple/educational images).
   - *Embedded questions*: Multiple-choice, grounded to specific source segments, with immediate feedback.

2. **Slides and Narration**: Slide sequences with interest-capturing questions, engagement activities, and optional narration designed to complement (not repeat) slide text.

3. **Audio-Graphic Lesson**: Simulated teacher-student conversation using independent Gemini personas. The student character may reveal misconceptions before receiving instruction. Coupled with dynamic graphical representation of key concepts and relationships (dual coding theory).

4. **Mind Maps**: Hierarchical organization with expandable/collapsible nodes at multiple granularity levels. Leaf nodes annotated with text and images from source material. Serves as post-session organizational tool or quick-reference reminder.

5. **Quizzes**: Section-level, 5-10 multiple-choice questions at various difficulty levels. Provides numerical score plus targeted feedback with "Glows" (strengths) and "Grows" (areas for improvement).

**Model**: Gemini 2.5 Pro (via LearnLM pedagogy-infused family) for all components; fine-tuned model only for visual illustrations.

### Architecture/Pipeline

```
Source PDF (textbook chapter)
    |
    v
[Stage 1: Personalization]
    |-- Grade-level re-leveling (Flesch-Kincaid)
    |-- Interest-based segment replacement
    v
Personalized Source Text
    |
    v
[Stage 2: Multi-Representation Generation]
    |-- Immersive Text (+ timelines, memory aids, illustrations, embedded questions)
    |-- Slides + Narration
    |-- Audio-Graphic Lesson (teacher-student dialogue + concept graphics)
    |-- Mind Map (hierarchical, expandable nodes)
    |-- Section Quizzes (MCQ + Glows/Grows feedback)
    v
Learn Your Way Interactive Interface
```

### Experiments

**Pedagogical Expert Evaluation:**
- Source materials: 10 OpenStax PDFs (world history, biology, physics, economics, astronomy, sociology, psychology)
- 3 grade levels x 3 interests = 9 combinations, 3 random assignments per PDF
- 3 independent raters per component
- 10-dimension rubric (accuracy, coverage, emphasis, engagement, cognitive load, active learning, metacognition, motivation, adaptability, clarity of learning intentions)
- Rating scale: Agree (1.0), Neutral (0.5), Disagree (0.0)
- **Result**: All components scored >0.90 across all pedagogical axes. Visual illustration was the lowest performer. Slides alone scored lowest on engagement; slides + narration scored significantly higher.

**Randomized Controlled Trial:**
- 60 students (ages 15-18) from Chicago area schools
- Selection based on reading comprehension (scores 4-9 out of 10, mean 6.4, SD 2.3)
- Material: "Brain Development for Adolescents" chapter (LibreTexts)
- Conditions: Learn Your Way (experimental) vs. Adobe Acrobat Reader (control)
- Protocol: 5-min intro, 20-40 min learning, 15-min immediate assessment, 10-min survey, 20-min interview, 3-day follow-up retention test (58/60 completed)
- Assessment: 15 questions across Bloom's taxonomy (short answer, MCQ, matching)

**Key Results:**
| Metric | Learn Your Way | Digital Reader | p-value |
|--------|---------------|----------------|---------|
| Immediate assessment | Higher | Baseline | p=0.03 |
| 3-day retention | 78% | 67% (+11%) | p=0.03 |
| Would use again | 93% | 67% | -- |
| Felt more comfortable | 100% | -- | -- |

- All experience dimensions (engagement, understanding, motivation) significantly favored Learn Your Way.
- All participants used quizzes; majority used at least one additional content transformation.

### Limitations

- **Component attribution**: Cannot isolate which component drives learning gains (quizzes? mind maps? personalization?). The system is evaluated holistically.
- **Narrow scope**: Single chapter evaluation only. Broader topic coverage untested.
- **Cost of detailed experiments**: Systematic ablation studies acknowledged as expensive.
- **No long-term study**: Only 3-day retention measured; semester-long effects unknown.
- **Implicit personalization missing**: Currently relies on explicit learner input (grade, interests) rather than adaptive sensing of learner behavior.

### Connections to Interactive Scientific Learning

This paper is highly relevant to the vision of transforming scientific knowledge into human-understandable formats:

- **Mind maps as knowledge trees**: Directly implements hierarchical, expandable knowledge structures where mathematical/scientific concepts are organized in parent-child relationships with annotations.
- **Multiple representation principle**: Demonstrates that the same scientific content can be rendered as text, audio dialogue, visual slides, mind maps, and interactive assessments -- and that this multiplicity improves learning outcomes.
- **Dual coding for scientific concepts**: Audio-graphic lessons combine auditory explanation with dynamic concept-relationship diagrams, directly relevant to visualizing scientific knowledge structures.
- **Timelines as structured representations**: Converts sequential scientific processes (experimental procedures, algorithmic steps) into interactive visual timelines.
- **Personalization of scientific analogies**: Shows how abstract physics concepts (Newton's Third Law) can be mapped to familiar domains (sports, art) to activate prior knowledge -- a form of knowledge translation.
- **Evidence-based validation**: Provides the strongest empirical evidence (RCT) among the papers surveyed that AI-generated multi-representation learning materials improve both comprehension and retention.

---

## Paper 8: Mindalogue: LLM-Powered Nonlinear Interaction for Effective Learning and Task Exploration

**Citation**: Rui Zhang, Ziyao Zhang, Fengliang Zhu, Jiajie Zhou, Anyi Rao. "Mindalogue: LLM-Powered Nonlinear Interaction for Effective Learning and Task Exploration." CHI 2025, arXiv:2410.10570.
**Venue**: ACM CHI Conference on Human Factors in Computing Systems, April 26-May 1, 2025, Yokohama, Japan
**Affiliations**: Tsinghua University, The New School, Beijing Institute of Technology, HKUST

### Problem

Current LLM interfaces (ChatGPT, Claude, Gemini) use linear, sequential text-based interaction: users pose a question, receive a response, and subsequent questions depend on previous responses. This linear model fails for complex tasks involving multi-layered information, multiple subtasks, and non-sequential exploration (brainstorming, structured knowledge learning, large project analysis). Users are forced to repeatedly scroll, compare, copy, and re-contextualize information across conversation turns, creating cognitive overload and reducing interaction efficiency. Useful information becomes scattered across long conversations with overlaps and conflicts.

### Key Contributions

- **Formative study identifying design requirements**: 11-participant study generating 618 initial codes, refined into 4 design considerations (DC1-DC4) that establish the need for nonlinear interaction, deeper answers, graphical representation, and structured information.
- **"Nodes + Canvas" interaction model**: A non-linear interaction paradigm where LLM responses are decomposed into independent, manipulable nodes on a 2D canvas with a 4-layer mind map hierarchy, replacing the linear chat scroll.
- **Three AI-powered deep exploration functions**: Each node supports Explanation (detailed concept expansion), Examples (3 specific cases), and Exploration (user-directed new queries), with parent-node context propagation.
- **Comparative evaluation**: 16-participant study across 4 task categories showing Mindalogue outperforms linear LLM interfaces on 4 of 5 dimensions (convenience, controllability, cognitive load reduction, result reliability).

### Methodology

**Phase 1: Formative Study**
- 11 participants (ages 19-29, mean=23.91, SD=2.57; 6 women, 5 men)
- 3 novice, 4 moderate, 4 advanced LLM users
- Remote interviews, 45-60 minutes each, recorded and transcribed
- Reflexive thematic analysis yielded 618 initial codes, merged into themes

**Four Design Considerations Emerged:**
1. DC1 (Linear interaction limitations): Frequent scrolling, step-by-step Q&A dependency, cross-topic questioning difficulty
2. DC2 (Deeper answers): Responses too broad and shallow for complex tasks
3. DC3 (Graphical representation): Text alone cannot convey multi-level structures; mind maps and flowcharts deemed more intuitive for memory retention and operational efficiency
4. DC4 (Structured information): Need for tables, flowcharts, and mind maps to organize long conversations and maintain logical coherence

**Phase 2: System Design (Four Design Goals)**
- DG1: Non-linear interaction for exploration freedom (node-based navigation)
- DG2: Deep information feedback (progressive, layered responses)
- DG3: Graphical representation for comprehension (mind map visualization)
- DG4: Hierarchical structure and information management (logical classification)

### Architecture/Pipeline

**Core Model**: "Nodes + Canvas" powered by ChatGPT-4.0

```
User Query
    |
    v
[LLM Generation with Predefined Task Prompts]
    |
    v
[4-Layer Mind Map Structure]
    |-- Layer 1: Root concept (brief phrase)
    |-- Layer 2: Sub-topics (brief phrases)
    |-- Layer 3: Sub-sub-topics (brief phrases)
    |-- Layer 4: Detailed descriptions (complete sentences)
    v
[Interactive Canvas]
    |-- Nodes: draggable, selectable, color-coded by subtopic
    |-- Per-node AI functions:
    |     |-- Explanation: detailed concept expansion
    |     |-- Examples: 3 specific related cases
    |     |-- Exploration: user-directed new queries (child nodes)
    |-- Context propagation: child nodes inherit parent context
    |-- Redundancy detection: prevents duplicate content
    |-- Spatial layout optimization: logical node positioning
    v
[User Exploration]
    |-- Nonlinear navigation (jump between any nodes)
    |-- Zoom/drag/edit capabilities
    |-- Custom node operations (add, delete, edit, move)
```

**Key Technical Features:**
- Vector graphics rendering for scalable node clarity
- Color coding per subtopic for visual hierarchy
- Parent-child context propagation ensures generated content relates to background
- Redundancy detection prevents duplicate nodes
- Predefined prompts per task type prevent overly broad responses

### Experiments

**Evaluation Study Design:**
- 16 participants (6 male, 10 female; ages 21-30, mean=24.06, SD=2.52)
- 3 monthly, 5 weekly, 8 daily LLM users
- Education: 10 master's, 4 bachelor's, 2 doctoral
- ~120 minutes total per participant
- Compared: Mindalogue (S1) vs. Mindmap Master (S2, linear LLM with mind map output)
- Both powered by ChatGPT-4.0 with identical settings

**Four Task Categories:**
1. Knowledge Exploration (unfamiliar): Machine Learning, Dadaism, Relativity, Industrial Revolution
2. Brainstorming (unfamiliar): Future Fashion, Dream Experiences, Digital Food, Event Planning
3. Product Analysis (familiar): Recommend a mobile app
4. City Description (familiar): Describe a favorite city

Each task: max 15 minutes. Cross-domain assignment to ensure unfamiliarity.

**Quantitative Results:**

| Metric | Mindalogue (S1) | Linear (S2) |
|--------|----------------|-------------|
| Avg time per task | 10.16 min (SD=3.20) | 7.78 min (SD=2.50) |
| Avg actions per task | 11.19 (SD=4.90) | 6.81 (SD=3.04) |
| Unfamiliar task time | 12.13 min | 8.94 min |
| Familiar task time | 8.19 min | 6.63 min |

Interpretation: Mindalogue users spent more time and performed more actions, indicating greater exploration space and engagement (not inefficiency).

**Task Support Ratings (1-7 scale):**
- Knowledge Exploration: S1=5.13 vs S2=4.50
- Brainstorming: S1=5.50 vs S2=5.00
- Product Analysis: S1=6.13 vs S2=5.88 (strongest showing)
- City Description: S1=5.50 vs S2=5.00

**Five-Dimension Comparison (Paired t-tests):**
- Convenience: S1 significantly better
- Usability: S2 slightly better (familiar chat interface easier to learn; S1 confidence p=0.008 favoring S2)
- Controllability: S1 better
- Cognitive Load Reduction: S1 better (psychological difficulty p=0.033 favoring S1)
- Result Reliability: S1 better
- Overall: S1 outperforms on 4/5 dimensions

**Cronbach's Alpha**: S1=0.74, S2=0.84 (both >0.7 threshold)

**Qualitative Findings:**
- Node explanations and case functions helped users "quickly grasp the entire task"
- Tree diagrams allow "clearer comprehension, improving efficiency"
- Structured presentation enhanced perceived reliability, especially for unfamiliar topics
- Users reported S2 content as "too repetitive" and "like chatting with a search engine"
- S1 excelled at: learning new knowledge, brainstorming, constructing frameworks
- Users requested: color/bold marking, auto sub-node generation, undo features, image generation, citation sources

### Limitations

- **Short session only**: Study confined to single sessions; does not capture long-term engagement spanning months/years typical of actual learning.
- **Individual use only**: Learning is often collaborative; individual preferences may shape visualization perceptions.
- **Learning curve**: Non-linear interaction's high degree of freedom caused some users to "lose direction" in information-dense tasks.
- **Content redundancy**: Node expansion sometimes produced repetitive content, reducing depth.
- **Annotation accuracy**: Occasional errors in complex multi-layered relationships (e.g., incorrect entity associations, misinterpreted node connections).
- **No longitudinal field study**: Authors call for randomized, controlled, longitudinal study for more accurate assessment.
- **Small sample size**: 16 participants limits generalizability.

### Connections to Interactive Scientific Learning

This paper directly addresses the vision of knowledge trees and mind maps for scientific understanding:

- **Mind map as primary interaction paradigm**: The 4-layer hierarchical mind map IS the interface, not an auxiliary feature. Scientific concepts would be naturally organized as expandable knowledge trees.
- **Nonlinear exploration of knowledge**: Users can jump between any concept nodes without sequential constraints -- exactly the kind of free exploration needed when navigating mathematical or physics concept spaces.
- **Progressive disclosure for complexity management**: The 3 brief layers + 1 detailed layer pattern mirrors how scientific knowledge could be presented: high-level overview first, then drill-down to equations and proofs.
- **Contextual deep exploration**: The Explanation/Examples/Exploration triad per node maps directly to how one might explore a scientific concept: understand the definition, see applications, then investigate related concepts.
- **Color-coded knowledge taxonomy**: Visual categorization by topic mirrors how knowledge trees could distinguish between definitions, theorems, proofs, and applications in mathematics.
- **Parent-child context propagation**: When exploring a child concept, the system automatically provides parent context -- essential for maintaining coherence when navigating deep scientific knowledge hierarchies.
- **Redundancy detection**: Prevents duplicate concepts in the knowledge tree, maintaining clean ontological structure.
- **Key limitation for scientific use**: The system currently uses ChatGPT-4.0 without domain-specific knowledge bases; integrating domain knowledge bases is identified as a critical future direction for professional/scientific domains.

---

## Paper 9: AutoMathKG: The Automated Mathematical Knowledge Graph Based on LLM and Vector Database

**Citation**: Rong Bian, Yu Geng, Zijian Yang, Bing Cheng. "AutoMathKG: The automated mathematical knowledge graph based on LLM and vector database." arXiv:2505.13406, 2025.
**Venue**: Pre-print (Academy of Mathematics and Systems Science, Chinese Academy of Sciences)
**Affiliations**: AMSS/CAS, University of Chinese Academy of Sciences

### Problem

Constructing mathematical knowledge graphs from natural language text faces two fundamental challenges: (1) existing approaches are constrained by corpus completeness, often discarding or manually supplementing incomplete knowledge without exploiting the inherent logic of mathematical knowledge; (2) existing KGs cannot automatically integrate knowledge from different sources, requiring substantial human effort for updates. Furthermore, while formal mathematics (Lean 4, Metamath) enables verification, it does not directly promote human understanding the way natural mathematical language does. The goal is to build a KG that captures mathematics in natural language, supports automatic updating, and enables fuzzy semantic search.

### Key Contributions

- **Multi-source mathematical KG**: Integrates knowledge from 4 sources (ProofWiki, textbooks, arXiv papers, TheoremQA) into a directed graph of 13,388 entities with 29,459 edges, representing Definitions, Theorems, and Problems with 9 tactic-labeled edge types.
- **MathVD vector database with dual embedding strategies**: Two SBERT-based embedding approaches (concatenation and weighted-sum) for entity similarity search, outperforming 5 KG embedding baselines (TransE, KG2E, HoLE, R-GCN, BoxE) on reachability queries at q>=5.
- **Automatic knowledge completion via Math LLM**: A gemma-7b-it model with 3 task-specific LoRA adapters (Application, Calculation, Proof) plus retrieval augmentation and self-calibration for filling missing proofs/solutions.
- **Automatic knowledge fusion mechanism**: New entities are embedded, compared against existing KG via cosine similarity in MathVD, and LLM determines whether to merge with an existing entity or add as new -- enabling automatic KG expansion.

### Methodology

**Mathematical Knowledge Model:**

Mathematics is represented as a directed graph G = {V, E} where:
- V = vertices representing mathematical entities of 3 types:
  - **Definition** (Def): Concise statements of mathematical concepts; foundational building blocks
  - **Theorem** (Thm): Logically proven statements including theorems, propositions, corollaries, lemmas; each includes proof process (potentially multiple proofs)
  - **Problem** (Prob): Mathematical problems with solution processes demonstrating application of definitions and theorems
- E = directed edges representing reference relationships. Edge from vi to vj means vj references vi in its content, proof, or solution.

**Nine Tactic Labels for Edges** (inspired by Lean 4):
premise, assumption, lemma, proposition, corollary, calculation, enumeration, definition, conclusion

**Graph allows cycles**: e.g., Pythagorean Theorem and Sum of Squares of Sine and Cosine are mutually referenced.

**Entity Storage**: JSON format with 17 attributes organized in 3 levels:
1. Basic information (id, type, label, title, field, contents, source)
2. Advanced information (bodylist with step-by-step logical segments and action labels, proofs, solutions)
3. Query information (in_refs, in_ref_ids, out_refs, out_ref_ids, references_tactics)

**Information Extraction Pipeline:**

Step 1 -- Rule-based extraction: LaTeX environment names (\begin{theorem}...\end{theorem}) parsed to extract entity type, title, contents, references, source.

Step 2 -- LLM augmentation via ICL: 12 prompt templates applied using Llama-2-7b to complete remaining attributes (title, field, bodylist, references_tactics, refs). Bodylist segments each entity's content into logical steps with tactic labels (e.g., "Let S be a set" -> action: "premise"; "Then (G,*) is called..." -> action: "definition").

### Architecture/Pipeline

```
[Data Sources]
    |-- ProofWiki (9,496 samples: 4,743 Def, 1,811 Thm, 2,942 Other)
    |-- Textbooks (1,605 samples: 361 Def, 1,209 Thm, 35 Other) [8 textbooks]
    |-- ArXiv Papers (538 samples: 134 Def, 399 Thm, 5 Other) [20 papers]
    |-- TheoremQA (1,749 samples: 1,084 Prob, 665 Other)
    v
[Rule-Based Extraction]
    |-- LaTeX environment parsing
    |-- Basic attribute extraction (id, type, contents, refs, source)
    v
[LLM Augmentation (Llama-2-7b, ICL)]
    |-- 12 prompt templates for attribute completion
    |-- Bodylist generation (logical segmentation with tactic labels)
    |-- Reference discovery and tactic labeling
    v
[AutoMathKG Construction]
    |-- 13,388 entities, 29,459 directed edges, 145 simple cycles
    |-- 4,216 head nodes (no incoming), 3,789 leaf nodes (no outgoing)
    v
[MathVD Construction (SBERT: all-MiniLM-L6-v2, 384-dim)]
    |-- MathVD1: Concatenation strategy (all 5 fields -> single text -> embed)
    |-- MathVD2: Weighted-sum strategy (embed fields separately, weights: content=0.5, title=0.3, field=0.1, in_refs=0.05, out_refs=0.05)
    v
[Automatic Update Mechanisms]
    |
    |-- Knowledge Completion (Math LLM):
    |     |-- Base: gemma-7b-it fine-tuned on MathInstruct
    |     |-- 3 LoRA Adapters:
    |     |     |-- Application (orca-math-200k)
    |     |     |-- Calculation (GSM8K-sympy-v2, PoT)
    |     |     |-- Proof (NaturalProofs, CoT)
    |     |-- 2-Stage Retrieval Augmentation:
    |     |     |-- Exact search in AutoMathKG
    |     |     |-- Fuzzy search in MathVD
    |     |-- Self-Calibration: step-by-step verification with feedback loop
    |
    |-- Knowledge Fusion:
          |-- New text -> Rule-based extraction + LLM augmentation -> Input KG
          |-- Embed Input KG entities via SBERT -> Input VD
          |-- For each input entity, retrieve top-5 similar from Existing VD
          |-- LLM determines: merge with candidate or add as new entity
          |-- If multiple matches: LLM performs secondary judgment for best match
```

### Experiments

**Hardware**: 4x RTX 3090 GPUs (96GB VRAM), Python 3.10

**Experiment 1: Reachability Query Performance**
- Metric: Hits@q for k-hop reachability (can entities A and B be connected by <= k edges?)
- Setup: 100 random entities (50 Def, 30 Thm, 20 Prob), q = {1, 5, 10, 15}, k = {1, 2, 3, 4, 5}
- Baselines: TransE, KG2E, HoLE, R-GCN, BoxE (all via PyKEEN, 384-dim)

**Results (5-hop reachability):**

| Model | Hits@1 | Hits@5 | Hits@10 | Hits@15 |
|-------|--------|--------|---------|---------|
| TransE | 0.9610 | 0.7766 | 0.7182 | 0.6797 |
| BoxE | 0.9351 | 0.8338 | 0.7610 | 0.7247 |
| **MathVD1** | 0.8831 | **0.8364** | **0.8182** | **0.7861** |
| **MathVD2** | **0.8974** | **0.8385** | **0.8013** | **0.7786** |

MathVD outperforms all baselines at q>=5. TransE wins at q=1 due to supervised optimization.

**Experiment 2: Math LLM Reasoning**
- Dataset: 234 questions from GHOSTS benchmark, 6 categories
- Rating: 1-5 scale (5=nearly perfect, 1=irrelevant)

| Category | Samples | Rating |
|----------|---------|--------|
| Algebra | 46 | 3.78 (best) |
| Complement | 35 | 3.37 |
| Theorem | 27 | 3.37 |
| Prealgebra | 43 | 3.26 |
| Topology | 39 | 3.05 |
| Probability | 44 | 3.05 |

Assessment: "Commendable for a 7B parameter model" -- correct reasoning with mostly correct answers, strongest in algebra.

**Experiment 3: Internal Retrieval Precision**
- 50 test samples per VD (25 Def, 15 Thm, 10 Prob), top-10 retrieval
- Manual evaluation of relevance

| Type | MathVD1 | MathVD2 |
|------|---------|---------|
| Definition | 96.0% | 95.3% |
| Theorem | 94.0% | 94.0% |
| Problem | 97.0% | 96.0% |

KS test: no significant distribution difference between MathVD1 and MathVD2.

**Experiment 4: External Retrieval (Natural Language Queries)**
- Query "Expectation in Probability and Statistics" retrieves relevant theorems, definitions, and problems about expectations across both VDs.
- MathVD2 returns "Definition: expectation" as top result (most semantically relevant), while MathVD1 returns "Theorem: expectation of geometric distribution" first.

**Ablation Study: Reference Information Impact**
- Removing in_refs/out_refs from embedding: retrieval becomes purely linguistic (matches titles with similar words).
- With refs: retrieves semantically essential entities (e.g., "Boole's inequality" for "probability measure is subadditive" -- a direct consequence despite dissimilar titles).
- Conclusion: Reference information is crucial for capturing mathematical essence beyond surface-level text similarity.

### Limitations

- **Small LLM scale**: Only Llama-2-7b for augmentation and gemma-7b-it for Math LLM; larger models would likely improve quality significantly.
- **Formal corpus only**: Excludes informal sources (math forums, discussions, Stack Exchange) that contain different types of mathematical knowledge and pedagogical explanations.
- **No systematic quality evaluation**: Missing precision/recall for knowledge fusion accuracy, entity deduplication correctness, or graph-level quality metrics.
- **Limited corpus scale**: 13,388 entities from 8 textbooks and 20 papers is small relative to the breadth of mathematics.
- **No user study**: No evaluation of whether the KG actually helps humans learn or navigate mathematical knowledge.
- **SBERT limitations**: 384-dimensional embeddings with a general-purpose sentence transformer may not capture deep mathematical semantics (mathematical notation, symbolic reasoning).
- **Tactic label set**: 9 labels may be insufficient for the full range of mathematical reasoning patterns.

### Connections to Interactive Scientific Learning

This paper is the most directly relevant to building structured knowledge representations of scientific/mathematical content:

- **Mathematical knowledge as a directed graph**: The core model -- definitions, theorems, and problems as nodes with tactic-labeled reference edges -- is precisely a "knowledge tree" for mathematics, though with cycles allowed.
- **Three-entity ontology**: The Definition/Theorem/Problem trichotomy provides a clean schema for organizing mathematical knowledge that could generalize to physics (Law/Derivation/Application) or other sciences.
- **Tactic labels as edge semantics**: The 9 tactic types (premise, assumption, lemma, corollary, etc.) capture HOW knowledge elements relate, not just THAT they relate -- essential for building comprehensible knowledge structures.
- **Bodylist logical segmentation**: Decomposing theorems and proofs into step-by-step segments with action labels (inspired by Lean 4 tactics) bridges the gap between formal and informal mathematical reasoning.
- **Fuzzy semantic search via vector database**: MathVD enables natural-language queries to navigate the knowledge graph, making it accessible to learners who don't know exact terminology -- a key requirement for "human-understandable" knowledge interfaces.
- **Automatic expansion**: The knowledge fusion mechanism means the KG can grow as new papers/textbooks are published, maintaining currency without manual curation.
- **Reference-aware embeddings**: The ablation showing that reference information (graph structure) improves retrieval beyond text similarity alone validates that knowledge graph structure captures mathematical meaning not present in text.
- **Limitation for the vision**: No interactive visualization, no personalization, no pedagogical layer. This is infrastructure (the KG backend) that would need systems like Mindalogue or Learn Your Way as frontends to become a learning tool.

---

## Cross-Paper Analysis: Synthesis and Connections

### Complementary Roles in the Vision

These three papers address different layers of a complete system for interactive scientific learning:

| Layer | Paper | Role |
|-------|-------|------|
| **Knowledge Backend** | AutoMathKG (#9) | Structured knowledge graph with entity types, relationships, and semantic search |
| **Interaction Paradigm** | Mindalogue (#8) | Nonlinear, node-based exploration interface with progressive disclosure |
| **Content Generation** | Learn Your Way (#7) | Multi-representation content transformation with personalization |

A combined system would use AutoMathKG's structured mathematical knowledge graph as the backend, Mindalogue's node+canvas nonlinear interaction as the exploration interface, and Learn Your Way's multi-representation generation for producing personalized explanations at each node.

### Shared Themes

1. **Hierarchical knowledge organization**: All three papers use hierarchical structures (mind maps in #7 and #8, directed knowledge graphs in #9) as the primary organizational principle.

2. **Multiple representations of the same knowledge**: Learn Your Way generates 5 formats from one source; AutoMathKG stores definitions, proofs, and solutions for the same mathematical entity; Mindalogue provides Explanation, Examples, and Exploration views per node.

3. **LLMs as knowledge transformers**: All three use LLMs not to generate novel knowledge but to transform, restructure, augment, or personalize existing knowledge -- keeping human-authored content as the source of truth.

4. **Addressing cognitive load**: All three explicitly cite cognitive load theory. Learn Your Way reduces it via personalization and progressive disclosure; Mindalogue reduces it via nonlinear navigation and color-coded structure; AutoMathKG reduces it via semantic search over structured entities.

5. **The "comprehension gap"**: All three papers are motivated by the gap between how knowledge is stored/published (textbooks, papers, formal systems) and how humans actually learn and explore it.

### Key Gaps Identified Across Papers

1. **No integration of formal verification with natural language**: AutoMathKG's tactic labels are inspired by Lean 4 but the system does not connect to formal proof systems. A complete vision would link natural-language explanations to machine-verified proofs.

2. **No adaptive learning based on knowledge state**: None of the systems model the learner's evolving knowledge state to adapt which nodes/representations to show. Learn Your Way personalizes on static attributes (grade, interests) not dynamic understanding.

3. **Scale and domain coverage**: AutoMathKG has 13K entities; real mathematical knowledge spans millions of concepts. Scalability of these approaches to comprehensive scientific knowledge is untested.

4. **Collaborative knowledge exploration**: All three focus on individual learners. Scientific learning is often collaborative (study groups, lab discussions, peer instruction).

5. **Equation and symbol handling**: None of the papers deeply address how to render, explain, or interact with mathematical equations and scientific notation -- a critical gap for the vision of making "math equations and physics" understandable.

### Evidence Strength Comparison

| Paper | Study Type | N | Key Evidence |
|-------|-----------|---|--------------|
| Learn Your Way (#7) | RCT | 60 | p=0.03 for both immediate and retention; expert ratings >0.90 |
| Mindalogue (#8) | Within-subjects comparison | 16 | 4/5 dimensions favored; cognitive load p=0.033 |
| AutoMathKG (#9) | Benchmark evaluation | -- | Hits@q superior to 5 baselines at q>=5; 94-97% retrieval precision |

Learn Your Way provides the strongest evidence for learning efficacy (randomized controlled trial with retention testing). Mindalogue provides HCI evidence for interaction design. AutoMathKG provides system-level technical evaluation but no user study.
# Phase 3 Deep Dive: Scientific Agents, MCP Infrastructure, and Verified Code Generation

## Papers Covered
- **Paper 10**: MCP-SIM: A Self-Correcting Multi-Agent LLM Framework for Language-Based Physics Simulation and Explanation
- **Paper 11**: Experiences with Model Context Protocol Servers for Science and High Performance Computing
- **Paper 12**: A Benchmark for Vericoding -- Formally Verified Program Synthesis

---

## Paper 10: MCP-SIM: A Self-Correcting Multi-Agent LLM Framework for Language-Based Physics Simulation and Explanation

**Authors**: Donggeun Park, Hyeonbin Moon, Seunghwa Ryu (KAIST, Korea)
**Venue**: npj Artificial Intelligence, Vol. 2, Article 10 (2026)
**DOI**: 10.1038/s44387-025-00057-z
**Code**: https://github.com/KAIST-M4/MCP-SIM

### Problem

Physics-based simulations are essential in science and engineering but require expert knowledge of numerical solvers and governing equations. LLMs offer the possibility of natural-language-driven simulation, but existing systems fail when prompts are vague, incomplete, or multilingual. One-shot code generators, autonomous simulation agents, and PINNs all assume well-posed input and offer little capacity for error recovery, refinement, or scientific explanation. The fundamental gap is a misalignment between one-shot LLM generation and the iterative, multi-step reasoning that expert modelers use in practice.

### Key Contributions

- **MCP-SIM framework**: A memory-coordinated, multi-agent system (6 specialized agents + orchestrator) that transforms underspecified natural language prompts into validated FEniCS simulations and multilingual explanatory reports.
- **Plan-Act-Reflect-Revise loop**: An iterative control cycle that emulates expert reasoning, enabling self-correction from incomplete or ambiguous inputs rather than relying on one-shot generation.
- **100% benchmark success**: Solved all 12 benchmark tasks across 6 physics domains (linear elasticity, heat conduction, fluid flow, thermo-mechanical coupling, piezoelectric deformation, phase-field fracture), significantly outperforming baseline GPT-4 (6/12).
- **Multilingual educational reports**: The Mechanical Insight Agent generates structured, pedagogically consistent reports in English, Korean, Japanese, and German, explaining governing equations, boundary conditions, and solver choices.

### Methodology

#### Agent Architecture (6 Agents + Orchestrator)

The system is built on GPT-4o via LangChain, with agents implemented as independent callable chains. Agents communicate exclusively through a shared global memory (JSON-based context log). The orchestrator mediates all transitions -- agents never call each other directly.

1. **Input Clarifier Agent**: Receives a vague prompt (e.g., "simulate fluid flow in an L-shaped pipe") and infers essential simulation details: domain geometry, governing PDE (e.g., Navier-Stokes), boundary conditions, material properties. Stores the canonical problem specification in global memory.

2. **Code Builder Agent**: Translates the clarified specification into solver-ready Python code using FEniCS. Uses prompt templates infused with physics-aware heuristics for mesh generation, solver configuration, and boundary condition specification. All assumptions and numerical strategies are logged.

3. **Simulation Executor Agent**: Runs the generated code in a sandboxed Python environment. Monitors physically meaningful indicators -- conservation violations, residual divergence, spurious field oscillations -- that may signal modeling or discretization issues.

4. **Error Diagnosis Agent**: Invoked when runtime failures occur (solver divergence, syntax errors, physical inconsistencies). Interprets failures in physical terms (e.g., insufficient mesh resolution, solver-mismatch). Proposes targeted, physics-aware corrections such as adjusting mesh density, reducing time step, or modifying solver parameters. Corrections are communicated as structured hints.

5. **Input Rewriter Agent**: If the error stems from high-level ambiguity in the original prompt, this agent semantically revises the instruction (e.g., adding "mesh resolution = 64" or "relaxation = 0.7"), and the control loop restarts from the Input Clarifier.

6. **Mechanical Insight Agent**: After successful simulation, generates an interpretive report including symbolic PDE summaries, physical reasoning behind the model, code-level annotations, rendered in multiple languages (English, Korean, Japanese, German).

7. **Memory-Centric Orchestrator**: The central coordination substrate. Captures the full trajectory: prompt history, problem clarifications, code iterations, diagnostic logs, final results. Enables context-aware decision-making across agents and prevents redundant computation or regressions. Orchestration rules: syntax errors trigger Error Diagnosis first; physical anomalies trigger Mechanical Insight analysis.

#### Control Loop

Plan -> Act -> Reflect -> Revise:
- **Plan**: Input Clarifier infers missing details, produces canonical specification
- **Act**: Code Builder generates FEniCS code, Executor runs it
- **Reflect**: If execution fails, Error Diagnosis Agent analyzes failure
- **Revise**: Either the Code Builder fixes based on error hints, or Input Rewriter revises the prompt and the loop restarts

### Architecture / Pipeline

```
User Prompt (vague/incomplete)
    |
    v
[Input Clarifier Agent] --> infers geometry, PDE, BCs, material props
    |                        stores canonical spec in shared memory
    v
[Code Builder Agent] --> generates FEniCS Python code
    |                     uses physics-aware prompt templates
    v
[Simulation Executor Agent] --> sandboxed execution
    |                            monitors conservation, residuals, oscillations
    |
    +-- Success --> [Mechanical Insight Agent] --> multilingual report
    |
    +-- Failure --> [Error Diagnosis Agent] --> physics-aware corrections
                        |
                        +-- Code-level fix --> back to Code Builder
                        +-- Prompt-level ambiguity --> [Input Rewriter] --> back to Input Clarifier
```

All communication flows through the **shared JSON memory** managed by the orchestrator. No direct agent-to-agent calls.

### Experiments

#### Benchmark Design: 12 Tasks Across 6 Physics Domains

Tasks are categorized into three difficulty tiers:
- **Well-posed (Levels 1-4)**: Single-physics problems with sufficient information
- **Intermediate (Levels 5-9)**: Increasing geometric/numerical complexity, some missing parameters
- **Challenging (Levels 10-12)**: Multi-physics with missing assumptions, inspired by published problems without code

Physics domains covered:
- Linear elasticity (Level 1: stress/displacement around a central hole)
- Heat conduction (Level 9: 3D heat diffusion in a coil-shaped domain)
- Fluid flow (Navier-Stokes)
- Thermo-mechanical coupling (Level 10: thermoelectric coupling)
- Piezoelectric deformation (Level 11)
- Phase-field fracture mechanics (Level 12: crack propagation from a single-sentence prompt)

#### Ablation Study Results

| Configuration | Tasks Solved (out of 12) |
|---|---|
| Baseline GPT-4 (one-shot prompt-to-code) | 6 / 12 |
| + Input Clarifier Agent | 8 / 12 |
| + Input Clarifier + Error Diagnosis Agent | 10 / 12 |
| **Full MCP-SIM** (all agents + memory + orchestrator) | **12 / 12 (100%)** |

#### Convergence Efficiency

- MCP-SIM typically converged in **fewer than 5 iterations**, even for the most complex problems
- Baseline models often failed to converge or required more than a dozen iterations
- Gains attributed to structured memory, agent-level specialization, and iterative self-healing

#### Physical Validation

- All simulations met physical convergence criteria with **residuals below 10^{-6}**
- Level 12 (phase-field fracture): After 10 self-correcting cycles, crack propagation patterns showed close agreement with ABAQUS commercial software results in crack initiation and growth direction
- Level 12 reproduced a published fracture simulation (Storvik et al., 2021) from a single-sentence prompt without explicit geometry or equations

### Limitations

**Stated limitations**:
- Built on GPT-4o without domain-specific fine-tuning; performance may degrade for rare material models, advanced PDE solvers, or multiphysics coupling beyond the training distribution
- The iterative reasoning loop introduces computational latency that may limit real-time or resource-constrained deployment
- No experimental or observational data validation; all benchmarks are synthetic

**Apparent limitations**:
- FEniCS-only backend limits to FEM simulations; no support for CFD-specific solvers, molecular dynamics, etc.
- 12-task benchmark is small and curated by the authors themselves (potential for overfitting to the evaluation)
- No comparison with other multi-agent systems (only comparison is ablation against GPT-4 baseline)
- Multilingual reports are demonstrated but not evaluated for accuracy or pedagogical effectiveness
- Shared memory grows with iterations; no discussion of context window limits or memory management
- No user study validating educational value of generated reports

### Connections to Broader Vision

MCP-SIM directly instantiates the vision of **LLMs transforming scientific knowledge into human-understandable formats**:
- Converts physics equations and simulation requirements into executable code + natural language explanations
- The Mechanical Insight Agent creates knowledge trees of sorts: structured reports that explain PDE choices, boundary conditions, solver rationale
- Demonstrates that multi-agent architectures with persistent memory can handle the iterative refinement that scientific computing demands
- The self-correction loop (Plan-Act-Reflect-Revise) is a concrete implementation pattern for scientific agents
- Positions LLMs not just as code generators but as "autonomous scientific assistants that simulate, adapt, and teach"

---

## Paper 11: Experiences with Model Context Protocol Servers for Science and High Performance Computing

**Authors**: Haochen Pan, Ryan Chard, Reid Mello, Christopher Grams, Tanjin He, Alexander Brace, Owen Price Skelly, Will Engler, Hayden Holbrook, Song Young Oh, Maxime Gonthier, Michael Papka, Ben Blaiszik, Kyle Chard, Ian Foster
**Affiliations**: University of Chicago; Argonne National Laboratory; University of Illinois Chicago
**Venue**: arXiv 2508.18489
**Code**: https://github.com/globus-labs/science-mcps (to be released)

### Problem

LLM-powered agents are increasingly used for scientific workflows, yet most research cyberinfrastructure (CI) exposes heterogeneous APIs and implements security models that present barriers for agent use. Each computer, service, instrument, tool, and database has its own APIs, security models, and operational requirements. The fundamental challenge is making distributed scientific computing resources discoverable, invokable, and composable by AI agents.

### Key Contributions

- **MCP as unifying interface for research CI**: Demonstrates that the Model Context Protocol can bridge the heterogeneity gap across distributed HPC systems, authentication domains, and scientific tools.
- **Reference MCP server implementations**: Thin MCP adapters over mature services including Globus Transfer, Compute, and Search; facility status APIs for ALCF and NERSC; Octopus event fabric; Garden ML model catalog; and Galaxy Toolshed (via Rhea).
- **Dynamic tool discovery (Rhea)**: A RAG-based approach using semantic search over Galaxy Toolshed documentation to dynamically materialize MCP tools, solving the context window limitation of exposing thousands of tools.
- **Formal framework and four case studies**: Formal definition of MCP-oriented agentic workflows (Plan-Resolve-Execute) with demonstrations in computational chemistry, bioinformatics, quantum chemistry, and filesystem monitoring.

### Methodology

#### Formal Framework

An agentic application context is defined as: User prompt (p), Coordinating LLM (L), User credentials (Phi_user), Set of MCP servers (M), Optional computing sites (S).

Each MCP server M_k = <C_k, Phi_k> where C_k = capabilities and Phi_k = auth client.
Each capability C_j = <I_j, E_j, D_j, R_j> (inputs, outputs, description, requirements).
Each computing site S_i = <Pi_i, Sigma_i> (software, resources).

#### Three-Stage Agentic Workflow

**Stage 1 -- Plan**: LLM converts user prompt into abstract plan T (set of high-level goals, not tied to specific capabilities or sites).

**Stage 2 -- Resolve**: Translates abstract plan into concrete plan R by finding a feasible tuple (S_i, C_j, M_k) for each task. Feasibility requires: capability available from server, site satisfies technical requirements (software + hardware).

**Stage 3 -- Execute**: Processes each tuple: first requests authorization using server's auth client with user credentials, then invokes capability on target site.

Complete workflow: W(p) = Execute(Resolve(Plan(p, L), M, S), Phi_user)

#### MCP Server Implementations

All servers deployed as separate Docker containers using streamable-HTTP transport (multi-client, bidirectional).

1. **Globus Transfer MCP Server**: Discover collections, browse file systems, transfer files across sites. Handles OAuth authentication flows, manages transfer task lifecycles.

2. **Globus Compute MCP Server**: Execute Python and Shell functions on remote HPC endpoints. Monitors execution, retrieves results.

3. **Globus Search MCP Server**: Create/delete/query search indexes. Translates conversational queries into structured search requests.

4. **Computing Facility MCP Server**: Real-time operational status for ALCF and NERSC systems. Provides system health, queue status, maintenance schedules, resource utilization. Enables agents to decide where and when to submit jobs.

5. **Octopus MCP Server**: Event streaming capabilities via AWS Managed Streaming for Kafka. Create/delete topics, publish/consume events under user identity.

6. **Garden MCP Server**: Discover and run scientific ML models. Agents discover published models, invoke for inference on cloud or HPC.

7. **Rhea MCP Server (Galaxy Toolshed)**: Dynamic tool discovery via RAG over Galaxy Toolshed documentation. Uses Qwen3-Embedding-0.6B for embeddings. Exposes a single `find_tools` tool that accepts natural-language queries. Performs semantic search, dynamically generates corresponding MCP tools, notifies agent of new tools via protocol notification channel.

#### Authentication Strategy

- MCP servers run locally within user's trusted environment
- All Globus service interactions wrapped with authentication handler
- Handler dynamically manages auth flows, acquires additional scopes as needed
- Avoids complexity of hosted token management

### Architecture / Pipeline

```
User Prompt
    |
    v
[Claude Desktop Agent (Claude Sonnet 4)]
    |
    +-- queries --> [Facility Status MCP] --> system availability
    +-- invokes --> [Globus Compute MCP] --> run code on HPC
    +-- invokes --> [Globus Transfer MCP] --> move data between sites
    +-- searches --> [Globus Search MCP] --> discover datasets
    +-- streams --> [Octopus MCP] --> event fabric
    +-- discovers --> [Garden MCP] --> ML models
    +-- finds --> [Rhea MCP] --> bioinformatics tools (RAG-based)

Each MCP Server:
    [Docker Container]
    - Streamable HTTP transport
    - Thin adapter over existing service
    - OAuth 2.0 auth handler
    - Minimal Python runtime
```

### Experiments

All use cases use Claude Sonnet 4 with Claude Desktop as the agent platform.

#### Use Case 1: Molecular Structure with Garden

- Single 32-atom copper structure optimization with MACE model (cloud execution) -> final energy -130.71 eV
- Batch processing of 49 copper structures on ALCF's Edith cluster
- Demonstrated transparent scaling from prototyping to production

#### Use Case 2: Multi-site Phylogenetic Analysis

- Motor proteins from 10 bacterial species
- Multi-site: ALCF Polaris (data acquisition, alignment) and NERSC Perlmutter (RAxML)
- Three phylogenetic tools: FastTree, RAxML (100 rapid bootstraps), IQ-TREE (1000 ultrafast bootstraps)
- Agent successfully created Python functions, correctly passed inputs between tools
- Eliminated custom glue code between computational tools and HPC systems

#### Use Case 3: Quantum Chemistry

- HOMO-LUMO gap calculations for 6 carbonate solvents (battery electrolyte design)
- M06-2X/6-311++G(d,p) level of theory using GPU4PySCF on ALCF Polaris
- Agent autonomously wrote and registered Python functions via Globus Compute
- Results consistent with published literature (Shakourian-Fard et al., 2016)
- Computed gaps: EC ~8.2 eV, PC ~8.0 eV, VC ~7.8 eV, DMC ~8.5 eV, EMC ~8.3 eV, DEC ~8.1 eV

#### Use Case 4: Filesystem Monitoring (Icicle)

- Lustre filesystem monitoring via Octopus event fabric
- Retrieved up to 10,000 events within 10-second timeout
- Generated pie charts of event type distributions, user comparison metrics
- Combined real-time (Octopus) and historical (Globus Search) analysis

#### Rhea Dynamic Tool Discovery Evaluation

- **Benchmark**: 380 test queries from Galaxy Training Tutorials, ground-truth from Galaxy workflows
- **Embedding strategies evaluated** (4 approaches):
  1. Tool names only (poorest)
  2. Names + descriptions
  3. Names + descriptions + extended documentation
  4. Names + descriptions + documentation + repository READMEs (best)
- Progressive improvement in Recall@k with richer textual context
- Strategy 4 (full context) significantly outperformed strategy 1 (names only)

### Limitations

**Stated limitations**:
- OAuth token management in hosted deployments adds complexity; current workaround (local deployment) not scalable
- Cross-domain MCP service hosting under existing authentication models remains an open challenge
- No established benchmarks for evaluating reliability of agent-driven workflows
- Agents made repetitive mistakes, suggesting limited learning from errors within sessions
- Agents sometimes failed to complete outlined tasks or produced inconsistent outputs (different visualizations for similar requests)
- Strengthening resilience for long-running tasks is an open challenge
- Only tested with Claude Sonnet 4 / Claude Desktop; no systematic comparison with other LLMs/agents

**Apparent limitations**:
- Only four use cases, all relatively well-defined scientific workflows
- No adversarial or stress testing
- Rhea evaluated only on Galaxy Toolshed queries; no evaluation on novel tool discovery
- No uncertainty quantification for scientific results
- Thin adapter approach limits intelligent optimization or caching
- Unclear generalization beyond computational science

### Connections to Broader Vision

This paper provides the **infrastructure layer** for scientific agents:
- MCP as a standard protocol makes scientific computing resources discoverable and composable for LLM agents
- The Plan-Resolve-Execute framework is a concrete pattern for agent-driven scientific workflows
- Rhea's dynamic tool discovery via RAG solves the practical problem of exposing thousands of scientific tools without overwhelming context windows
- Demonstrates agents can generate "glue code" on the fly, removing the need for custom integration between scientific tools
- Agent self-correction in scientific workflows (e.g., automatically fixing malformed function calls, correcting file paths) shows practical resilience
- Directly relevant to CLI tools for science: MCP servers ARE the CLI interface that agents use to interact with HPC resources
- The thin-adapter design principle means existing scientific infrastructure can be MCP-enabled with minimal effort

---

## Paper 12: A Benchmark for Vericoding -- Formally Verified Program Synthesis

**Authors**: Sergiu Bursuc, Theodore Ehrenborg, Shaowei Lin, Lacramioara Astefanoaei, Ionel Emilian Chiosa, Jure Kukovec, Alok Singh, Oliver Butterley, Adem Bizid, Quinn Dougherty, Miranda Zhao, Max Tan, Max Tegmark
**Affiliations**: Beneficial AI Foundation; Massachusetts Institute of Technology
**Venue**: arXiv 2509.22908
**Code**: https://github.com/Beneficial-AI-Foundation/vericoding-benchmark

### Problem

AI-generated code (vibe coding) is becoming pervasive -- Google reports over 30% of its software is created this way -- but such code can contain bugs. Traditional testing cannot prove absence of bugs (combinatorial explosion). High-profile failures (Ariane-V explosion 1996, Shellshock 2014, CrowdStrike outage 2024 affecting 8.5M devices) demonstrate the consequences. Formal verification provides machine-checkable correctness proofs but remains niche because it requires far more human labor than programming. The research gap: formal verification lacks large-scale benchmarks (largest < 10^3 examples) compared to theorem proving (>10^5), hampering progress in AI-assisted verified code generation.

### Key Contributions

- **Largest vericoding benchmark**: 12,504 formal specifications across Dafny (3,029), Verus/Rust (2,334), and Lean (7,141), with 6,174 new or translated problems. This is two orders of magnitude larger than previous verification benchmarks.
- **Comprehensive LLM evaluation**: 55,397 experiments across 9 frontier LLMs, revealing success rates of 82% (Dafny), 44% (Verus), 27% (Lean) for model unions.
- **Rapid progress documentation**: Pure Dafny verification improved from 68% to 96% in one year (June 2024 to 2025).
- **Natural language ineffectiveness**: Adding natural-language descriptions to formal specifications provides no statistically significant improvement, questioning the value of combining formal and informal descriptions.

### Methodology

#### Formal Definitions

**Vericoding task**: Context + formal specification + optional documentation -> passed to LLM.
**Vericoding solution**: Implementation + proof + additional context (imports, helper functions, lemmas).

The paper distinguishes:
- **ATPs** (Automated Theorem Provers): Dafny, Verus -- use SMT solvers (Z3); verification is largely automatic
- **ITPs** (Interactive Theorem Provers): Lean -- use tactics; require human-like interactive guidance

#### Benchmark Construction

**Original sources (3 categories)**:
1. Formal verification benchmarks: DafnyBench, VerifiedCogen, Verina, CLEVER
2. Vibe coding benchmarks: APPS (5,000 test instances), FVAPPS (Lean translations), HumanEval
3. Mathematical documentation: NumPy library docs (v2.3), BigNum cryptographic algorithms (13 functions, 62 tasks)

**Task generation**: Delete implementations and proofs from existing formal sources, replace with holes. For vibe coding/documentation sources, autoformalize to generate formal specs.

**SpecTranslator algorithm**: Iterative LLM-based specification translation between languages:
1. Generate or fix translation
2. Verify file in target language, capture (status, errors)
3. Update history
4. If verified, return success; else build fix prompt with error messages
5. Up to k iterations

**Quality assessment**: Score = 1 - sum(w_i * n_i) where w_i = weight for issue type, n_i = count. Near-duplicate detection using sentence transformers with FAISS indexing.

#### Vericoder Algorithm

```
Input: Language L, spec S, prompt templates P, iterations k, LLM M
For i in {1,...,k}:
    1. B_1,...,B_n = M(Q)           # LLM generates code blocks
    2. F = S[B_1,...,B_n]           # Reconstruct file
    3. (v0, w0) = ValidateBlocks()  # Check for cheating patterns
    4. (v1, w1) = VerifyFile()      # Run proof checker
    5. If v0 AND v1: return success
    6. Else: build fix prompt with errors
Return fail
```

Standard: 5 attempts (1 initial + 4 retries). BigNum: 10 attempts.

#### LLM Cheating Detection (ValidateBlocks)

Blocks specific patterns:
- `assume(false)` or `sorry` statements (disabled via compiler options)
- Postcondition trivialization (`ensures true`)
- Implementation leakage from specifications (mitigated via ghost functions)
- Comment section tricks (bracket pairing monitoring)

### Architecture / Pipeline

```
Formal Specification (Dafny / Verus / Lean)
    |
    v
[Prompt Template (language-specific)]
    |-- generation prompt: JSON-only, no explanations, bypass rules
    |-- repair prompt: includes error messages from previous attempt
    |
    v
[LLM] --> generates code blocks (implementation + proof)
    |
    v
[ValidateBlocks] --> checks for cheating patterns
    |                  (assume, sorry, ensures true, leakage)
    |
    v
[VerifyFile] --> runs language-specific proof checker
    |             (Dafny compiler / Verus verifier / Lean kernel)
    |
    +-- Both pass --> SUCCESS (verified solution)
    +-- Either fails --> feed errors back to LLM (up to k iterations)
```

### Experiments

#### Main Results (Model Union = best across all 9 LLMs)

**Dafny (2,161 tasks)**:
| Model | Overall |
|---|---|
| Claude Opus 4.1 | 67.5% |
| GPT-5-Mini | 66.9% |
| GPT-5 | 66.1% |
| Claude Sonnet 4 | 64.6% |
| Gemini 2.5 Pro | 55.0% |
| Grok Code | 53.0% |
| GLM 4.5 | 42.0% |
| Gemini 2.5 Flash | 38.2% |
| DeepSeek Chat v3.1 | 36.2% |
| **Model Union** | **82.2%** |

**Verus (2,166 tasks)**:
| Model | Overall |
|---|---|
| GPT-5 | 30.9% |
| Claude Opus 4.1 | 24.6% |
| Gemini 2.5 Pro | 24.1% |
| Claude Sonnet 4 | 19.9% |
| **Model Union** | **44.3%** |

**Lean (2,361 tasks)**:
| Model | Overall |
|---|---|
| GPT-5 | 17.9% |
| Claude Sonnet 4 | 11.9% |
| Gemini 2.5 Pro | 10.9% |
| Claude Opus 4.1 | 10.7% |
| **Model Union** | **26.8%** |

#### Year-over-Year Progress (DafnyBench, 782 tasks, verification-only)

| Model | Success |
|---|---|
| Opus-3 (June 2024) | 68% |
| Opus-4.1 (2025) | 89% |
| GPT-5 (2025) | **96%** |

This represents a **68% -> 96% improvement in one year**.

#### FVAPPS Results (4,006 Lean tasks)

- GPT-5: 38.0%
- Model Union: 41.8%
- Note: weaker specifications allow trivial solutions

#### Natural Language (Vibe Information) Experiment

Testing with Verina dataset (157 tasks), comparing specs+NL vs specs-only:

- **No statistically significant improvement** from adding natural language descriptions
- Results slightly worse on average with vibe information
- Suggests formal specifications are self-sufficient for vericoding

#### Difficulty Analysis

- **Solution length**: Strongest predictor of difficulty. Longer implementations more likely incorrect.
- **Spec length**: Weakest predictor. Longer specs often contain helper definitions, not harder problems.
- **Spec ratio** (code/spec): Intermediate predictive power.

#### Dataset-Specific Findings

- **BigNum** (cryptographic algorithms): Uniformly poor across all models (max 25.8% Dafny). Specialized domain knowledge required.
- **VerifCogen**: Highest success (90.7% Dafny with Opus 4.1). Well-structured formal specs.
- **NumPy Triple** (Hoare triples with mvcgen): Only 7.6% model union in Lean. New feature likely not in training data.

#### Model Union Analysis

Different models solve different subsets of problems:
- Dafny: 82.2% union vs 67.5% best single (14.7pp gap)
- Verus: 44.3% union vs 30.9% best single (13.4pp gap)
- Lean: 26.8% union vs 17.9% best single (8.9pp gap)

Suggests model routing/ensemble strategies could yield significant gains.

### Limitations

**Stated limitations**:
- Tasks limited to ~100 lines of code; not yet extended to complex benchmarks like SWE-bench
- No extensive prompt optimization; significant room for improvement
- No tree search or reinforcement learning approaches explored
- No collaborative LLM networks tested

**Apparent limitations**:
- ~9% of specs too weak even when vericoding succeeds (trivial solutions possible)
- ~15% have poor translations across languages
- Only 5 retry attempts vs human interactive proving (unlimited feedback cycles)
- Manual inspection limited to 5 samples per language/source
- BigNum results suggest fundamental limitation on specialized domains
- Lean models trained primarily on mathematical theorem proving, not code verification
- Ghost function mitigation of implementation leakage is partial, not complete

### Connections to Broader Vision

This paper is fundamental to the vision of **verified scientific code generation**:
- If scientific agents generate simulation code or data analysis pipelines, vericoding provides the mechanism to guarantee correctness
- The 68% -> 96% improvement in one year suggests formal verification of AI-generated code may become routine
- The finding that natural language descriptions don't help suggests LLMs can work directly with formal mathematical specifications -- relevant for scientific equations
- Model union results (82% Dafny) indicate that combining specialized models could make verified code generation practical
- The benchmark infrastructure (12,504 tasks, 3 languages) provides the training data ecosystem needed for further progress
- Directly relevant to scientific agents: code generated by systems like MCP-SIM could be formally verified to ensure physical correctness
- The Dafny/Verus/Lean language comparison reveals a tradeoff: more automated provers (Dafny) are easier for LLMs, while interactive provers (Lean) are harder but more expressive

---

## Cross-Paper Synthesis

### Common Themes

1. **Multi-agent architectures for science**: Both MCP-SIM (Paper 10) and the MCP servers paper (Paper 11) employ multi-agent systems where specialized components collaborate. MCP-SIM uses 6 agents for simulation; the MCP architecture uses 7+ servers for infrastructure access.

2. **Iterative self-correction**: MCP-SIM's Plan-Act-Reflect-Revise loop and Vericoding's iterative repair with error feedback both embody the principle that scientific code generation requires multiple attempts with structured feedback.

3. **Infrastructure vs. Application layering**: Paper 11 provides the infrastructure (MCP servers for HPC access), Paper 10 provides the application pattern (multi-agent simulation), and Paper 12 provides the verification layer (proving generated code correct). Together they outline a full stack for trustworthy scientific computing.

4. **The glue code problem**: Paper 11 explicitly identifies that agents can "remove the need to create custom glue code." Paper 10 demonstrates this by having agents write FEniCS simulation code from natural language. Paper 12 provides the mechanism to verify that glue code is correct.

5. **Scalability of tool discovery**: Rhea's RAG-based tool discovery (Paper 11) and the Vericoding benchmark's automated spec translation (Paper 12) both address the challenge of scaling AI systems across large ecosystems of tools or specifications.

### Implications for Scientific Agents

- **Complete pipeline**: A scientific agent could use MCP servers (Paper 11) to access HPC resources, employ MCP-SIM-like multi-agent reasoning (Paper 10) to generate simulation code from natural language, and use vericoding (Paper 12) to formally verify correctness before execution.
- **Trust hierarchy**: MCP authentication (Paper 11) provides infrastructure trust, self-correction loops (Paper 10) provide empirical trust (convergence below 10^{-6}), and formal verification (Paper 12) provides mathematical trust.
- **The NL-to-formal gap**: Paper 12's finding that natural language descriptions don't help vericoding suggests that scientific agents should aim to produce formal specifications directly, rather than relying on informal descriptions as intermediaries.
- **Rapid capability growth**: Dafny verification going from 68% to 96% in one year (Paper 12) and MCP-SIM solving all 12/12 benchmark tasks (Paper 10) both suggest that AI-driven scientific computing capabilities are advancing rapidly.

### Open Questions

1. Can MCP-SIM's generated FEniCS code be formally verified using Dafny or Lean specifications?
2. Can MCP servers expose formal verification tools (Dafny, Lean) as capabilities for agents?
3. How do self-correction loops (Paper 10) compare to formal verification (Paper 12) in terms of guarantees? MCP-SIM achieves residuals < 10^{-6} but this is empirical, not provable.
4. Can Rhea's RAG-based tool discovery (Paper 11) be extended to discover formal specifications or verification strategies?
5. What happens when these systems encounter problems outside their training distribution -- novel physics, new programming languages, or interdisciplinary challenges?
