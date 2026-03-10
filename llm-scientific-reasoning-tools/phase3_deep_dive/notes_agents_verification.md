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
