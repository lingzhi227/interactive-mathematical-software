# Phase 1 (Frontier) Survey: LLMs for Transforming Scientific Knowledge into Human-Understandable Formats

**Search date**: 2026-03-10
**Coverage**: 2024--2026 papers across AI4Science agents, autoformalization, and scientific coding benchmarks

---

## Dimension 1: AI for Science Agents & Autonomous Scientific Discovery

### Surveys

**From Automation to Autonomy: A Survey on Large Language Models in Scientific Discovery**
- Authors: Tianshi Zheng (first), Zheye Deng, Hong Ting Tsang, Weiqi Wang, Jiaxin Bai, Zihao Wang, Yangqiu Song
- Year: 2025 | Venue: EMNLP 2025 Main | arXiv: 2505.13259
- Proposes a three-tier framework (Tool, Analyst, Scientist) capturing escalating levels of LLM autonomy throughout the scientific process. Maps LLM capabilities against the traditional scientific method while identifying challenges including robotic integration and iterative self-enhancement. Accompanied by a curated GitHub repository (HKUST-KnowComp/Awesome-LLM-Scientific-Discovery).

**Towards Scientific Intelligence: A Survey of LLM-based Scientific Agents**
- Authors: Shuo Ren (first), Can Xie, Pu Jian, Zhenjiang Ren, Chunlin Leng, Jiajun Zhang
- Year: 2025 | arXiv: 2503.24047
- Comprehensive survey examining LLM-based scientific agents with domain-specific expertise, specialized tools, and validation mechanisms. Covers agent architectures, design methodologies, performance benchmarks, real-world applications across scientific disciplines, and ethical considerations. Provides a roadmap for developing more efficient and reliable scientific discovery systems.

**Agentic AI for Scientific Discovery: A Survey of Progress, Challenges, and Future Directions**
- Authors: Mourad Gridach (first), Jay Nanavati, Khaldoun Zine El Abidine, Lenon Mendes, Christina Mack
- Year: 2025 | Venue: ICLR 2025 | arXiv: 2503.08979
- Examines AI systems capable of reasoning, planning, and autonomous decision-making across chemistry, biology, and materials science. Categorizes existing systems and tools, covers evaluation metrics and implementation frameworks, and proposes future research prioritizing collaborative human-AI partnerships.

**From AI for Science to Agentic Science: A Survey on Autonomous Scientific Discovery**
- Authors: Jiaqi Wei (first) + 26 co-authors
- Year: 2025 | arXiv: 2508.14111
- Introduces "Agentic Science" as a pivotal paradigm where AI progresses from partial assistance to full scientific agency. Unifies foundational AI capabilities, core discovery processes, and domain-specific applications through a comprehensive framework. Provides domain-oriented review across life sciences, chemistry, materials, and physics.

### Agentic Systems for Computational Science

**AlphaEvolve: A Coding Agent for Scientific and Algorithmic Discovery**
- Authors: Alexander Novikov (first) + 17 co-authors (Google DeepMind)
- Year: 2025 | arXiv: 2506.13131
- Evolutionary coding system that amplifies LLM capabilities by orchestrating multiple language models to autonomously refine algorithms through iterative code modifications. Discovered a matrix multiplication algorithm using 48 scalar multiplications for 4x4 complex matrices -- the first improvement over Strassen's algorithm in 56 years. A scheduling solution discovered by AlphaEvolve continuously recovers 0.7% of Google's worldwide compute resources.

**FunSearch: Mathematical Discoveries from Program Search with Large Language Models**
- Authors: Bernardino Romera-Paredes (first) et al. (Google DeepMind)
- Year: 2024 | Venue: Nature | DOI: 10.1038/s41586-023-06924-6
- Evolutionary procedure pairing a pre-trained LLM with a systematic evaluator to search in function space. Applied to the cap set problem in extremal combinatorics, discovering constructions exceeding all known ones, including the largest improvement in 20 years to the asymptotic lower bound. Also found improved bin packing algorithms. Demonstrates that verifiable scientific discoveries are possible using LLMs.

**DREAMS: Density Functional Theory Based Research Engine for Agentic Materials Simulation**
- Authors: Ziqi Wang (first), Hongshuo Huang, Hancheng Zhao, Changwen Xu, Shang Zhu, Jan Janssen, Venkatasubramanian Viswanathan
- Year: 2025 | arXiv: 2507.14267
- Hierarchical multi-agent framework for DFT simulation combining a central LLM planner with domain-specific agents for structure generation, convergence testing, HPC scheduling, and error handling. Validated on the Sol27LC lattice-constant benchmark with average errors below 1% compared to human DFT experts. Approaches L3-level automation (autonomous exploration of a defined design space), significantly reducing reliance on human expertise.

**ChemGraph: An Agentic Framework for Computational Chemistry Workflows**
- Authors: Thang D. Pham (first), Aditya Tanikanti, Murat Keceli
- Year: 2025 | Venue: Communications Chemistry | arXiv: 2506.06363
- AI-powered framework combining graph neural network models with LLMs for natural language understanding and task planning in computational chemistry. Supports molecular structure generation, geometry optimization, vibrational analysis, and thermochemistry calculations. Multi-agent decomposition enables smaller LLMs to match or exceed single-agent GPT-4o performance on complex workflows.

**AgenticSciML: Collaborative Multi-Agent Systems for Emergent Discovery in Scientific Machine Learning**
- Authors: Qile Jiang (first), George Karniadakis
- Year: 2025 | arXiv: 2511.07262
- Framework coordinating over 10 specialized AI agents (proposers, critics, engineers, retrievers, evaluators) that iteratively generate, analyze, and refine SciML solutions. Achieves up to four orders of magnitude error reduction over baselines across physics-informed and operator learning tasks. Agents produce novel strategies (adaptive mixture-of-expert architectures, decomposition-based PINNs) not present in the curated knowledge base.

**Autonomous Agents for Scientific Discovery: Orchestrating Scientists, Language, Code, and Physics**
- Authors: Lianhao Zhou (first)
- Year: 2025 | arXiv: 2510.09901
- Describes a comprehensive framework where LLM agents orchestrate interactions across human scientists, natural language, programming code, and physical principles. Traces the complete scientific discovery lifecycle from hypothesis generation through experimental design and implementation to analysis and refinement. Identifies current shortcomings and proposes strategies for more resilient, widely applicable agents.

**34 Examples of LLM Applications in Materials Science and Chemistry**
- Authors: Yoel Zimmermann (first) + 34 co-authors including Ben Blaiszik, Ian Foster, Philippe Schwaller
- Year: 2025 | arXiv: 2505.03049
- Review of 34 projects from the 2nd annual LLM Hackathon for Applications in Materials Science and Chemistry. Spans seven research areas: molecular/material property prediction, molecular/material design, automation and novel interfaces, scientific communication, research data management, hypothesis generation, and knowledge extraction. Documents how LLMs function as versatile predictive models and rapid prototyping platforms.

---

## Dimension 2: Autoformalization

### Surveys

**Autoformalization in the Era of Large Language Models: A Survey**
- Authors: Ke Weng (first), Lun Du, Sirui Li, Wangyue Lu, Haozhe Sun, Hengyu Liu, Tiancheng Zhang
- Year: 2025 | arXiv: 2505.23486
- Comprehensive survey on converting informal mathematical statements into formal, verifiable representations. Covers three primary areas: (1) autoformalization across different mathematical domains and difficulty levels, (2) the complete workflow from data preparation through model development and assessment, and (3) emerging potential for autoformalization to enhance LLM reliability and trustworthiness by improving verifiability.

### PDE and Physics Autoformalization

**PDE-Controller: LLMs for Autoformalization and Reasoning of PDEs**
- Authors: Mauricio Soroco (first), Jialin Song, Mengzhou Xia, Kye Emond, Weiran Sun, Wuyang Chen
- Year: 2025 | Venue: ICML 2025 | arXiv: 2502.00963
- Framework enabling LLMs to transform informal natural language PDE descriptions into formal specifications (>64% accuracy) and executable programs integrating with external tools (>82% accuracy). Built the first comprehensive datasets for PDE control designed for LLMs, including over 2 million samples of natural/formal language, code programs, and PDE control annotations. Achieves up to 62% improvement in utility gain for PDE control over baseline models.

**PDE-SHARP: PDE Solver Hybrids through Analysis and Refinement Passes**
- Authors: Shaghayegh Fazliani (first), Madeleine Udell
- Year: 2025 | arXiv: 2511.00183
- Three-stage framework (Analysis, Genesis, Synthesis) replacing expensive scientific computation with cheaper LLM inference. Requires fewer than 13 solver evaluations on average (vs 30+ for baselines) while achieving approximately 4x average accuracy improvement. Demonstrates consistent effectiveness across general-purpose, coding-specific, and reasoning LLM architectures.

**CodePDE: An Inference Framework for LLM-driven PDE Solver Generation**
- Authors: Shanda Li (first), Tanya Marwah, Junhong Shen, Weiwei Sun, Andrej Risteski, Yiming Yang, Ameet Talwalkar
- Year: 2025 | Venue: TMLR | arXiv: 2505.08783
- First inference framework framing PDE solving as a code generation task. Instructs LLMs to produce diverse solver implementations (finite difference, spectral methods, etc.) given problem descriptions. With inference-time techniques (automated debugging, self-refinement, test-time scaling), achieves performance comparable to human experts on average and exceeds expert-level on 4 of 5 evaluated tasks.

**AutoNumerics: An Autonomous, PDE-Agnostic Multi-Agent Pipeline for Scientific Computing**
- Authors: Jianda Du (first), Youran Sun, Haizhao Yang
- Year: 2026 | arXiv: 2602.17607
- Multi-agent framework autonomously designing, implementing, debugging, and verifying numerical PDE solvers from natural language descriptions. Generates transparent solvers grounded in classical numerical analysis (not black-box). Includes coarse-to-fine execution strategy and residual-based self-verification. Competitive or superior accuracy on 24 canonical and real-world PDE problems vs neural and LLM-based baselines.

### Math Autoformalization

**Process-Driven Autoformalization in Lean 4 (FormL4)**
- Authors: Jianqiao Lu (first), Yingjia Wan, Zhengying Liu, Yinya Huang, Jing Xiong + 8 others
- Year: 2024 | Venue: ICLR 2025 | arXiv: 2406.01940
- Introduces FormL4, a large-scale dataset for evaluating autoformalization in Lean 4 encompassing both statements and proofs. Proposes Process-Driven Autoformalization (PDA) framework leveraging precise Lean 4 compiler feedback to enhance autoformalization. Demonstrates higher compiler accuracy and human-evaluation scores using less filtered training data.

**MerLean: An Agentic Framework for Autoformalization in Quantum Computation**
- Authors: Yuanjie Ren (first), Jinzheng Li, Yidi Qi
- Year: 2026 | arXiv: 2602.16554
- Fully automated agentic framework extracting mathematical statements from LaTeX source files, formalizing them into verified Lean 4 code on Mathlib, and translating results back to human-readable LaTeX. Evaluated on three theoretical quantum computing papers, generating 2,050 Lean declarations from 114 statements with end-to-end formalization on all three papers. Demonstrates that agentic autoformalization can scale to frontier research, serving both as a verification tool for peer review and a synthetic data engine.

**ProofBridge: Auto-Formalization of Natural Language Proofs in Lean via Joint Embeddings**
- Authors: Prithwish Jana (first), Kaan Kale, Ahmet Ege Tanriverdi, Cruise Song, Sriram Vishwanath, Vijay Ganesh
- Year: 2025 | arXiv: 2510.15681
- Unified framework for translating entire NL theorems and proofs into Lean 4. Core innovation is a joint embedding model aligning NL and FL theorem-proof pairs in shared semantic space. Integrates retrieval-augmented fine-tuning with iterative proof repair using Lean's type checker. Improves cross-modal retrieval by 3.28x Recall@1 and achieves +31.14% semantic correctness over Kimina-Prover-RL-1.7B baseline.

**IndiMathBench: Autoformalizing Mathematical Reasoning Problems with a Human Touch**
- Authors: (Microsoft Research)
- Year: 2025 | arXiv: 2512.00997
- Benchmark of 312 Indian Mathematical Olympiad problems with human-verified Lean 4 formalizations. Uses AI-powered human-assisted pipeline with category-based retrieval, iterative compiler feedback, and multi-model ensembles. Claude Opus 4 achieves the highest BEq score (67/312) and compilation success rate (243/312). Demonstrates substantial gaps between syntactic validity and semantic correctness in autoformalization.

### Physics Formalization

**Lean4Physics: Comprehensive Reasoning Framework for College-level Physics in Lean4**
- Authors: Yuxin Li (first), Minghao Liu, Ruida Wang, Wenzhao Ji, Zhitao He, Rui Pan, Junming Huang, Tong Zhang, Yi R. Fung
- Year: 2025 | Venue: ICLR 2026 Poster | arXiv: 2510.26094
- First formal physics benchmark in Lean4, consisting of PhysLib (community-driven repository of unit systems and theorems) and LeanPhysBench (200 hand-crafted, peer-reviewed physics problems from textbooks and competitions). Best performance: Claude-Sonnet-4 at 35%, DeepSeek-Prover-V2-7B at 16%. PhysLib achieves average 11.75% model performance improvement.

**PhysLean/HepLean: Digitalising Physics in Lean 4**
- Authors: Joseph Tooby-Smith
- Year: 2024--2025 | Venue: Computer Physics Communications (Vol. 308, 2025)
- Project to formalize physics results in Lean 4, covering Maxwell's equations, quantum harmonic oscillator, canonical ensembles, tight-binding model, twin paradox, two-Higgs doublet model, and Wick's theorem. Aspires to be the definitive physics library in Lean analogous to Mathlib for mathematics. Accompanied by arXiv:2411.07667 on formalization of physics index notation.

---

## Dimension 3: Scientific Coding Benchmarks

### Core Benchmarks

**SciCode: A Research Coding Benchmark Curated by Scientists**
- Authors: Minyang Tian (first) + collaborators
- Year: 2024 | Venue: NeurIPS 2024 D&B Track | arXiv: 2407.13168
- Benchmark of 80 main problems (338 subproblems) across 16 subdomains from physics, math, materials science, biology, and chemistry. Converted from real research problems requiring knowledge recall, reasoning, and code synthesis. Best model (Claude 3.5 Sonnet) solves only 4.6% in the most realistic setting. Demonstrates vast gap between LLM capabilities and real scientific coding needs.

**FEM-Bench: A Structured Scientific Reasoning Benchmark for Evaluating Code-Generating LLMs**
- Authors: Saeed Mohammadzadeh (first), Erfan Hamdi, Joel Shor, Emma Lejeune
- Year: 2025 | arXiv: 2512.20732
- Computational mechanics benchmark spanning FEM 1D, FEM 2D, and 3D matrix structural analysis. Tasks assess element-level routines, mesh generation, quadrature, geometric mappings, stiffness/load assembly, and eigenvalue solves. Best function-writing: Gemini 3 Pro (30/33 tasks at least once); best unit test writing: GPT-5 (73.8% Average Joint Success Rate). Reveals significant gaps between current LLM capabilities and physics-based scientific computing requirements.

**HeuriGym: An Agentic Benchmark for LLM-Crafted Heuristics in Combinatorial Optimization**
- Authors: Hongzheng Chen (first)
- Year: 2025 | Venue: ICLR 2026 | arXiv: 2506.07972
- Framework evaluating LLM ability to generate and refine heuristic algorithms for combinatorial optimization. Nine problems across computer systems, logistics, and biology, deliberately excluding well-known problems to prevent memorization. Introduces Quality-Yield Index (QYI) metric capturing both pass rate and quality. Even top models (GPT-4o-mini-high, Gemini-2.5-Pro) achieve QYI of only 0.6 vs expert baseline of 1.0.

**PHYBench: Holistic Evaluation of Physical Perception and Reasoning in Large Language Models**
- Authors: Shi Qiu (first) + collaborators
- Year: 2025 | arXiv: 2504.16074
- Benchmark of 500 original physics problems from high school to Physics Olympiad level, covering mechanics, electromagnetism, thermodynamics, optics, modern physics, and advanced physics. Introduces Expression Edit Distance (EED) Score improving sample efficiency by 204% over binary scoring. Best model (Gemini 2.5 Pro) achieves 36.9% vs human experts at 61.9%.

### Equation Discovery Benchmarks

**LLM-SR: Scientific Equation Discovery via Programming with Large Language Models**
- Authors: Parshin Shojaee (first), Kazem Meidani, Shashank Gupta, Amir Barati Farimani, Chandan K. Reddy
- Year: 2024 | Venue: ICLR 2025 Oral | arXiv: 2404.18400
- Novel approach treating equations as executable programs, combining LLMs' scientific knowledge with evolutionary search. LLM iteratively proposes equation skeleton hypotheses informed by domain knowledge, then parameters are optimized against data. Significantly outperforms state-of-the-art symbolic regression baselines, particularly in out-of-domain generalization across physics, biology, and materials science benchmarks.

**LLM-SRBench: A New Benchmark for Scientific Equation Discovery with Large Language Models**
- Authors: Parshin Shojaee (first), Ngoc-Hieu Nguyen, Kazem Meidani, Amir Barati Farimani, Khoa D. Doan, Chandan K. Reddy
- Year: 2025 | Venue: ICML 2025 Oral | arXiv: 2504.10415
- Comprehensive benchmark with 239 problems across chemistry (36), biology (24), physics (43), and materials science (25). Two categories: LSR-Transform (transforms common models into unfamiliar representations to test reasoning beyond memorization) and LSR-Synth (synthetic discovery-driven problems requiring data-driven reasoning). Best-performing system achieves only 31.5% symbolic accuracy.

**Think like a Scientist: Physics-guided LLM Agent for Equation Discovery (KeplerAgent)**
- Authors: Jianke Yang (first), Ohm Venkatachalam, Mohammad Kianezhad, Sharvaree Vadgama, Rose Yu
- Year: 2026 | arXiv: 2602.12259
- Introduces KeplerAgent, an agentic framework that mimics the scientific method for equation discovery. First infers physical properties (symmetries), then uses these as priors to constrain the equation search space. Coordinates physics-based tools with symbolic regression engines (PySINDy, PySR). Achieves substantially higher symbolic accuracy and greater noise robustness than both LLM and traditional baselines.

---

## Cross-cutting Themes & Key Takeaways

1. **The formalization gap is real but narrowing**: PDE-Controller achieves >82% accuracy generating executable PDE programs from natural language; MerLean fully formalizes quantum computing papers into Lean 4; yet Lean4Physics shows best LLM achieves only 35% on college-level physics formalization.

2. **Multi-agent architectures dominate agentic science**: DREAMS, ChemGraph, AgenticSciML, and AutoNumerics all use specialized agent ensembles (planner, executor, critic, debugger). AgenticSciML shows 4 orders of magnitude error reduction through agent collaboration.

3. **Scientific coding benchmarks expose fundamental limitations**: SciCode (4.6% solve rate), FEM-Bench (73.8% best joint success), LLM-SRBench (31.5% symbolic accuracy), PHYBench (36.9% vs 61.9% human) all reveal major gaps between LLM capabilities and research-grade scientific coding.

4. **Evolutionary code search is a proven paradigm**: FunSearch (Nature 2024) and AlphaEvolve (2025) demonstrate that LLM+evaluator+evolution can yield genuine scientific discoveries (cap set improvements, Strassen algorithm advances).

5. **Autoformalization is expanding beyond pure math**: PDE-Controller (applied math/PDEs), Lean4Physics (physics), MerLean (quantum computation), PhysLean (high-energy physics) show formalization spreading to scientific domains, not just theorem proving.

---

## Sources

- [From Automation to Autonomy (EMNLP 2025)](https://arxiv.org/abs/2505.13259)
- [Towards Scientific Intelligence](https://arxiv.org/abs/2503.24047)
- [Agentic AI for Scientific Discovery (ICLR 2025)](https://arxiv.org/abs/2503.08979)
- [From AI for Science to Agentic Science](https://arxiv.org/abs/2508.14111)
- [AlphaEvolve](https://arxiv.org/abs/2506.13131)
- [FunSearch (Nature)](https://www.nature.com/articles/s41586-023-06924-6)
- [DREAMS](https://arxiv.org/abs/2507.14267)
- [ChemGraph](https://arxiv.org/abs/2506.06363)
- [AgenticSciML](https://arxiv.org/abs/2511.07262)
- [Autonomous Agents for Scientific Discovery](https://arxiv.org/abs/2510.09901)
- [34 Examples of LLM Applications in MatSci/Chem](https://arxiv.org/abs/2505.03049)
- [Autoformalization Survey](https://arxiv.org/abs/2505.23486)
- [PDE-Controller (ICML 2025)](https://arxiv.org/abs/2502.00963)
- [PDE-SHARP](https://arxiv.org/abs/2511.00183)
- [CodePDE (TMLR)](https://arxiv.org/abs/2505.08783)
- [AutoNumerics](https://arxiv.org/abs/2602.17607)
- [FormL4 / Process-Driven Autoformalization (ICLR 2025)](https://arxiv.org/abs/2406.01940)
- [MerLean](https://arxiv.org/abs/2602.16554)
- [ProofBridge](https://arxiv.org/abs/2510.15681)
- [IndiMathBench](https://arxiv.org/abs/2512.00997)
- [Lean4Physics (ICLR 2026)](https://arxiv.org/abs/2510.26094)
- [PhysLean/HepLean](https://github.com/HEPLean/PhysLean)
- [SciCode (NeurIPS 2024)](https://arxiv.org/abs/2407.13168)
- [FEM-Bench](https://arxiv.org/abs/2512.20732)
- [HeuriGym (ICLR 2026)](https://arxiv.org/abs/2506.07972)
- [PHYBench](https://arxiv.org/abs/2504.16074)
- [LLM-SR (ICLR 2025 Oral)](https://arxiv.org/abs/2404.18400)
- [LLM-SRBench (ICML 2025 Oral)](https://arxiv.org/abs/2504.10415)
- [KeplerAgent](https://arxiv.org/abs/2602.12259)
