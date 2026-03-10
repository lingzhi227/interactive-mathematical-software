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
