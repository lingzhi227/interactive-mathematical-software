# Phase 2: Survey — LLM-Assisted Scientific Reasoning & Knowledge Tools

**Date**: 2026-03-10 | **Total papers**: 82 (deduplicated) | **Period**: 2013-2026

## Research Question

How can LLMs transform complex scientific knowledge (mathematical equations, physics theories) into human-understandable formats (knowledge trees, graphs, natural language, executable code), and how can coding assistant paradigms be extended to support interactive scientific reasoning?

---

## Taxonomy: 11 Thematic Clusters

### A. Foundational Infrastructure

#### Cluster 1: Mathematical Reasoning Foundations (8 papers)
The bedrock techniques enabling LLMs to reason about mathematics.

- **Chain-of-Thought** (Wei+ NeurIPS 2022): Intermediate reasoning steps in prompts
- **Self-Consistency** (Wang+ ICLR 2023): Sampling + majority vote over reasoning paths
- **PAL** (Gao+ ICML 2023): Programs as intermediate reasoning steps
- **Program of Thoughts** (Chen+ TMLR 2023): Disentangling computation from reasoning via code
- **Minerva** (Lewkowycz+ NeurIPS 2022): Training on scientific papers + math web data
- **Llemma** (Azerbayev+ ICLR 2024): Open math LLM on Proof-Pile-2
- **Tree of Thoughts** (Yao+ NeurIPS 2023): Tree-structured exploration with backtracking
- **Least-to-Most** (Zhou+ ICLR 2023): Hierarchical decomposition of complex problems

**Evolution**: CoT (2022) → Self-Consistency (2023) → Code-as-reasoning (PAL/PoT, 2023) → Tree-structured reasoning (ToT, 2023) → Domain-specific models (Minerva/Llemma)

#### Cluster 2: Knowledge Representation & Ontology (7 papers)
Structured representations for mathematical and scientific knowledge.

- **KG Survey** (Hogan+ ACM Computing Surveys 2021): Canonical reference for knowledge graphs
- **Math Ontologies** (Lange, SWJ 2013): MathML, OpenMath, OMDoc for semantic math
- **AutoMathKG** (Bian+ 2025): Auto-updating math KG from ProofWiki/arXiv/textbooks
- **Math KG for Manufacturing** (Cha+ 2026): Equation ontology with variables, assumptions, validity
- **Enhancing Math KGs** (MDPI 2025): LaTeX→KG with hypothesis/proof ontology
- **PyIRK for Control** (Fiedler+ 2025): Interactive semantic layer for engineering
- **MindMap KG Prompting** (Wen+ ACL 2024): KG-grounded reasoning with transparency

**Key insight**: Ontology-guided LLM extraction consistently outperforms free-form extraction. Knowledge graphs serve as the structured bridge between unstructured text and machine-interpretable formats.

### B. Core Technical Capabilities

#### Cluster 3: Equation-to-Code Translation (8 papers)
The central technical challenge: converting mathematical notation to executable programs.

- **LLM-SR** (Shojaee+ ICLR 2025 Oral): Equations as executable programs + evolutionary search
- **LLM-SRBench** (Shojaee+ ICML 2025 Oral): 239 problems, best=31.5% accuracy
- **PiT-PO** (Wang+ 2026): RL-based equation discovery with physics constraints
- **SymCode** (Bagheri Nezhad+ EACL 2026): Math→SymPy code, +13.6pp over baselines
- **KeplerAgent** (Yang+ 2026): Physics-guided agent mimicking scientific method
- **QuaSAR** (Ranaldi+ ACL 2025): Quasi-symbolic CoT without full formalization
- **DotaMath** (Li+ 2024): Decompose math→code subtasks with self-correction
- **Drori+ university math** (PNAS 2022): Codex solving MIT courses via program synthesis

**Paradigm shift**: From "LLMs do arithmetic" to "LLMs write programs that do arithmetic" (PAL/PoT → LLM-SR → SymCode). Code is the universal bridge between math notation and computation.

#### Cluster 4: Autoformalization (12 papers)
Converting natural language math/physics to formally verified specifications.

*Foundational*: Szegedy (2020), GPT-f (2020), Wu+ (NeurIPS 2022), DSP (ICLR 2023 Oral), NaturalProofs (2021)

*Current*: FormL4 (ICLR 2025), ProofBridge (2025), IndiMathBench (2025), Autoformalization Survey (2025), Open Proof Corpus (2025), Formal Math Reasoning position paper (2024)

*Expanding to physics*:
- **PDE-Controller** (ICML 2025): NL→formal PDE specs (>64% accuracy)
- **MerLean** (2026): LaTeX→Lean4 for quantum computation (2,050 declarations)
- **Lean4Physics** (ICLR 2026): First physics benchmark in Lean4 (best=35%)
- **PhysLean/HepLean** (Comp.Phys.Comm. 2025): Maxwell, QHO, Wick's theorem in Lean4

**Trend**: Autoformalization is expanding from pure math → applied math (PDEs) → physics (quantum, HEP). The formalization gap is narrowing but still substantial.

#### Cluster 5: PDE Solving as Code Generation (7 papers)
A specialized but rapidly growing subfield treating PDE solving as a code generation task.

- **CodePDE** (TMLR): First systematic framework; expert-competitive with 4 debug rounds
- **AutoNumerics** (2026): Multi-agent, 24 PDE problems, coarse-to-fine strategy
- **PDE-SHARP** (2025): 3-stage framework, <13 solver evals, 4x accuracy gain
- **Lang-PINN** (2025): NL→PINN, 3-5 OOM MSE reduction
- **OpInf-LLM** (2026): Operator inference + LLM for parametric PDEs
- **ATHENA** (NeurIPS 2025): Hierarchical evolutionary algorithms, 10^-14 validation error
- **MCP-SIM** (npj AI 2026): 100% success on 12 physics simulation tasks

**Pattern**: Multi-agent architectures with self-correction dominate. Self-debugging (4+ rounds) is essential.

### C. Applications & Systems

#### Cluster 6: AI for Science Agents (8 papers)
LLM-powered autonomous agents for scientific discovery workflows.

*Surveys*: Zheng+ (EMNLP 2025), Gridach+ (ICLR 2025), Wei+ (2025), Ren+ (2025)

*Systems*:
- **AlphaEvolve** (DeepMind 2025): Improved Strassen algorithm, saves 0.7% Google compute
- **FunSearch** (Nature 2024): Verified math discoveries via evolutionary code search
- **DREAMS** (2025): Multi-agent DFT with <1% error vs. human experts
- **AgenticSciML** (2025): 10+ agents, 4 OOM error reduction
- **SciAgents** (Adv. Materials 2025): KG + multi-agent for materials discovery

**Architecture**: Central planner + specialized agents (structure generator, executor, critic, debugger) is the dominant pattern.

#### Cluster 7: AI-Generated Textbooks & Interactive Learning (6 papers)
Making scientific knowledge accessible through personalized, interactive representations.

- **Learn Your Way/LearnLM** (Google 2025): +11% retention via multi-representation AI textbook
- **KMP-Bench** (2026): Solving ≠ teaching — pedagogical intelligence needs distinct training
- **Inquizzitor** (AAAI 2026): Adaptive scaffolding grounded in cognitive science
- **PedagogicalRL-Thinking** (2026): RL for pedagogical reasoning traces
- **Mindalogue** (CHI 2025): Nonlinear "nodes+canvas" interface beats linear chat
- **Concept Map Generation** (Zhai 2025): Systematic review of 28 studies

**Key finding**: Nonlinear interfaces (trees, maps, canvases) substantially improve learning. Solving ability ≠ teaching ability — pedagogical alignment requires distinct training.

#### Cluster 8: IDE/CLI & Scientific Workflow Tools (5 papers)
Extending coding assistant paradigms for scientific work.

- **MCP for Science** (Pan+ LBNL 2025): MCP servers for Globus, Galaxy, NERSC/HPC
- **Jupiter** (AAAI 2026): MCTS over Jupyter notebook workflows
- **Jupyter AI**: Official JupyterLab generative AI extension
- **NotebookLM** (Google): Source-grounded AI for scientific document synthesis
- **Scientific Workflows** (Yildiz+ 2024): LLMs struggle with domain-specific workflows

**Emerging standard**: MCP (Model Context Protocol) as the interface layer connecting LLM agents to HPC resources, simulation tools, and scientific databases.

### D. Evaluation & Verification

#### Cluster 9: Scientific Coding Benchmarks (9 papers)
Measuring LLM capabilities on real scientific coding tasks.

| Benchmark | Domain | Best Score | Gap |
|-----------|--------|-----------|-----|
| SciCode (NeurIPS 2024) | 16 subfields | 4.6% realistic | Vast |
| FEM-Bench (2025) | Computational mechanics | 73.8% joint | Large |
| HeuriGym (ICLR 2026) | Combinatorial optimization | QYI=0.6 | Significant |
| PHYBench (2025) | Physics problems | 36.9% vs 61.9% human | Large |
| UGPhysics (ICML 2025) | Undergrad physics | 49.8% | Large |
| PhysReason (2025) | Physics reasoning | <60% overall | Large |
| LLM-SRBench (ICML 2025) | Equation discovery | 31.5% symbolic | Large |
| ASyMOB (2025) | Symbolic math | 70.3% degradation on perturbations | Pattern-matching risk |
| TeXpert (ACL 2025) | LaTeX generation | Drops with complexity | Moderate |

**Bottom line**: LLMs solve <10% of research-grade scientific coding problems in realistic settings. The gap between demo capabilities and real-world scientific computing is enormous.

#### Cluster 10: Verification & Formal Methods (2 papers)
Ensuring LLM-generated scientific code is correct.

- **Astrogator** (2025): Formal verification of LLM code — 83% verification, 92% error detection
- **Vericoding Benchmark** (2025): 12,504 specs; Dafny improved 68%→96% in one year

**"Vericoding" vs "vibecoding"**: The gap is closing fast. Formally verified scientific code generation may be practical within 1-2 years.

#### Cluster 11: Cross-Domain Transfer (2 papers)
Can mathematical reasoning transfer to physics?

- **Knowledge or Reasoning** (Wu+ 2025): Math reasoning does NOT naturally transfer to physics
- **UGPhysics/PhysReason**: Physics requires situational modeling + causal grounding beyond formal math

**Implication**: Building LLM tools for cross-domain scientific reasoning (math→physics) requires domain-specific training, not just mathematical skill.

### E. Supporting Infrastructure

#### LaTeX Pipeline (3 papers)
- **Nougat** (ECCV 2024): PDF→LaTeX/markup via Vision Transformer
- **ASyMOB** (2025): LaTeX→SymPy parsing with LLM fallback
- **TeXpert** (ACL 2025): Benchmark for LLM LaTeX generation

#### Scientific Software Maintenance (4 papers)
- **ChronoLLM** (2025): Domain-specific LLM for PyChrono simulation
- **CodeScribe** (2024): Fortran→C++ for LHC particle simulation
- **HPC-Coder-V2** (2024): Best open-source parallel code LLM
- **MatTools** (2025): Materials science tool benchmark

---

## Paper Statistics

| Metric | Value |
|--------|-------|
| Total papers | 82 |
| Peer-reviewed | 48 (59%) |
| Top venues (NeurIPS/ICLR/ICML/ACL/AAAI/CHI) | 28 |
| Journal (Nature/Science/PNAS/TMLR/npj AI) | 11 |
| arXiv preprint only | 34 |
| Year range | 2013-2026 |
| Median year | 2025 |

| Cluster | Count |
|---------|-------|
| Math Reasoning Foundations | 8 |
| Knowledge Representation | 7 |
| Equation-to-Code | 8 |
| Autoformalization | 12 |
| PDE as Code Gen | 7 |
| AI4Science Agents | 8 |
| AI Textbooks & Learning | 6 |
| IDE/CLI & Workflow | 5 |
| Benchmarks | 9 |
| Verification | 2 |
| Cross-Domain | 2 |
| LaTeX Pipeline | 3 |
| Scientific Software | 4 |
| **Total (some papers in multiple)** | **82 unique** |

---

## Evolution Timeline

```
2013-2016: Mathematical ontologies (MathML/OpenMath) + Jupyter notebooks
2019-2021: Math benchmarks (MATH, GSM8K) + theorem proving (GPT-f)
2022: CoT prompting + Minerva + Autoformalization with LLMs + Codex
2023: PAL/PoT (code-as-reasoning) + DSP + Llemma + Tree of Thoughts + Nougat
2024: FunSearch (Nature) + SciCode + SciAgents + ACL concept maps
2025: LLM-SR (ICLR Oral) + PDE-Controller (ICML) + DREAMS + MCP for Science
      + Learn Your Way + Lean4Physics + Vericoding benchmark
2026: MerLean + AutoNumerics + MCP-SIM + KeplerAgent + PiT-PO + KMP-Bench
```

---

## Key Open Questions for the User's Vision

1. **Knowledge representation gap**: No unified ontology bridges math notation ↔ natural language ↔ code across physics domains
2. **Verification gap**: How to ensure LLM-generated scientific code correctly implements equations?
3. **Pedagogical gap**: LLMs that solve ≠ LLMs that teach — how to build both capabilities?
4. **Cross-domain gap**: Math reasoning doesn't transfer to physics — how to bridge?
5. **Tooling gap**: No IDE/CLI extension specifically designed for scientific knowledge management (math ↔ code ↔ explanation)
6. **Interactive format gap**: Nonlinear interfaces (trees, graphs) work better but no standard for scientific knowledge navigation

*Phase 2 complete. Proceeding to Phase 3: Deep Dive.*
