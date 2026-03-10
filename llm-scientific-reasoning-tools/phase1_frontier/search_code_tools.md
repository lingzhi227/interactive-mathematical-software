# Phase 1 (Frontier): LLMs for Scientific Knowledge Translation & Code Tools

**Survey Date**: 2026-03-10
**Scope**: 2024--2026 papers on LLM-driven equation-to-code translation, IDE/CLI extensions for science, and AI-assisted scientific software maintenance.

---

## Dimension 1: LLM Equation-to-Code & Symbolic-to-Numerical Translation

### 1.1 LLM-SR: Scientific Equation Discovery via Programming with Large Language Models
- **Authors**: Parshin Shojaee, Kazem Meidani, Shairaj Patel, Amir Barati Farimani, Chandan K. Reddy
- **Year**: 2024 (updated 2025)
- **Venue**: ICLR 2025 (Oral)
- **arXiv**: 2404.18400
- **Summary**: LLM-SR treats equations as programs with mathematical operators and combines LLMs' scientific priors with evolutionary search over equation programs. The LLM iteratively proposes equation skeleton hypotheses drawn from domain knowledge, which are then optimized against data to estimate parameters. Discovers physically accurate equations that significantly outperform state-of-the-art symbolic regression baselines, particularly in out-of-domain generalization across physics, biology, and materials science benchmarks.

### 1.2 LLM-SRBench: A New Benchmark for Scientific Equation Discovery with Large Language Models
- **Authors**: Parshin Shojaee, Ngoc-Hieu Nguyen, Kazem Meidani, Amir Barati Farimani, Khoa D. Doan, Chandan K. Reddy
- **Year**: 2025
- **Venue**: ICML 2025 (Oral)
- **arXiv**: 2504.10415
- **Summary**: Introduces a comprehensive benchmark with 239 challenging problems across four scientific domains, designed to prevent trivial memorization. Features two components: LSR-Transform (transforms known physical models into uncommon mathematical representations) and LSR-Synth (synthetic discovery-driven problems requiring data-driven reasoning). Best-performing system achieves only 31.5% symbolic accuracy, revealing substantial room for improvement.

### 1.3 PiT-PO: LLM-Based Scientific Equation Discovery via Physics-Informed Token-Regularized Policy Optimization
- **Authors**: Boxiao Wang, Kai Li, Tianyi Liu, Chen Li, Junzhe Wang, Yifan Zhang, Jian Cheng
- **Year**: 2026
- **Venue**: arXiv preprint
- **arXiv**: 2602.10576
- **Summary**: Introduces PiT-PO, a unified framework that evolves the LLM into an adaptive equation generator via reinforcement learning. Uses a dual-constraint mechanism enforcing hierarchical physical validity while applying fine-grained token-level penalties (based on a Support Exclusion Theorem) to suppress redundant structures. Achieves state-of-the-art on standard benchmarks and discovers novel turbulence models for fluid dynamics; enables small-scale models to outperform closed-source giants.

### 1.4 SymCode: A Neurosymbolic Approach to Mathematical Reasoning via Verifiable Code Generation
- **Authors**: Sina Bagheri Nezhad, Yao Li, Ameeta Agrawal
- **Year**: 2025
- **Venue**: EACL 2026 Findings
- **arXiv**: 2510.25975
- **Summary**: Reformulates mathematical problem-solving as verifiable code generation using SymPy's symbolic computation capabilities. Achieves up to 13.6 percentage point improvement over baselines on MATH-500 and OlympiadBench. Shifts model errors from opaque logical fallacies to transparent, debuggable programming errors, anchoring LLM reasoning in deterministic symbolic computation. More token-efficient than prose-based approaches.

### 1.5 CodePDE: An Inference Framework for LLM-driven PDE Solver Generation
- **Authors**: Shanda Li, Tanya Marwah, Junhong Shen, Weiwei Sun, Andrej Risteski, Yiming Yang, Ameet Talwalkar
- **Year**: 2025
- **Venue**: TMLR (Transactions on Machine Learning Research)
- **arXiv**: 2505.08783
- **Summary**: Reframes PDE solving as a code generation task. Provides the first systematic evaluation of LLM capabilities for PDE solving across reasoning, debugging, self-refinement, and test-time scaling. Demonstrates that LLMs can generate solvers competitive with or superior to expert-written solvers. Identifies a key trade-off between solver reliability and sophistication; self-debugging for up to 4 rounds is essential for reliable performance.

### 1.6 OpInf-LLM: Parametric PDE Solving with LLMs via Operator Inference
- **Authors**: Zhuoyuan Wang, Hanjiang Hu, Xiyu Deng, Saviz Mowlavi, Yorie Nakahira
- **Year**: 2026
- **Venue**: arXiv preprint
- **arXiv**: 2602.01493
- **Summary**: Bridges operator inference with LLM interfaces for parametric PDE solving. Uses a small amount of solution data with shared reduced-order bases across PDE families to enable accurate, generalizable solutions under varying boundary conditions. Achieves improved trade-offs between accuracy and execution success rate compared to pure code-generation approaches like CodePDE.

### 1.7 PDE-SHARP: PDE Solver Hybrids through Analysis and Refinement Passes
- **Authors**: Shaghayegh Fazliani, Madeleine Udell
- **Year**: 2025
- **Venue**: arXiv preprint
- **arXiv**: 2511.00183
- **Summary**: Reduces computational cost of LLM-driven PDE solver generation by replacing expensive numerical evaluation with cheaper LLM inference. Three-stage framework: Analysis (mathematical chain-of-thought with PDE classification and stability analysis), Genesis (solver generation from insights), and Synthesis (collaborative selection-hybridization tournaments). Requires fewer than 13 solver evaluations (vs 30+ for baselines) while improving accuracy 4x on average.

### 1.8 Lang-PINN: From Language to Physics-Informed Neural Networks via a Multi-Agent Framework
- **Authors**: Xin He, Liangliang You, Hongduan Tian, Bo Han, Ivor Tsang, Yew-Soon Ong
- **Year**: 2025
- **Venue**: arXiv preprint (submitted to ICLR 2026)
- **arXiv**: 2510.05158
- **Summary**: Multi-agent LLM system that builds trainable PINNs directly from natural language task descriptions. Coordinates four agents: PDE Agent (parses descriptions into symbolic PDEs), PINN Agent (selects architectures), Code Agent (generates implementations), and Feedback Agent (executes and diagnoses). Reduces MSE by 3--5 orders of magnitude, improves execution success by >50%, and reduces time overhead by up to 74% vs baselines.

### 1.9 AutoNumerics: An Autonomous, PDE-Agnostic Multi-Agent Pipeline for Scientific Computing
- **Authors**: Jianda Du, Youran Sun, Haizhao Yang
- **Year**: 2026
- **Venue**: arXiv preprint
- **arXiv**: 2602.17607
- **Summary**: Multi-agent framework that autonomously designs, implements, debugs, and verifies numerical solvers for general PDEs from natural language descriptions. Generates transparent solvers grounded in classical numerical analysis (not black-box neural solvers). Features coarse-to-fine execution (debug on low-res first), residual-based self-verification for problems without analytical solutions, and history decimation for large-scale temporal simulations. Tested on 24 canonical and real-world PDE problems.

### 1.10 ATHENA: Agentic Team for Hierarchical Evolutionary Numerical Algorithms
- **Authors**: Juan Diego Toscano, Daniel T. Chen, George Em Karniadakis
- **Year**: 2025
- **Venue**: NeurIPS MTI-LLM 2025
- **arXiv**: 2512.03476
- **Summary**: Autonomous lab framework for end-to-end computational research lifecycle, bridging theoretical conceptualization and computational implementation. Core is the HENA loop, a contextual bandit process that analyzes prior trials to select structural actions guided by expert blueprints (universal approximation, physics-informed constraints). Achieves validation errors of 10^-14 on PDE benchmarks and enables human-in-the-loop improvements with order-of-magnitude enhancements.

### 1.11 Beyond Static Tools: Test-Time Tool Evolution for Scientific Reasoning
- **Authors**: Jiaxuan Lu, Ziyu Kong, Yemin Wang, Rong Fu, Haiyuan Wan, Cheng Yang, Wenjie Lou, Haoran Sun, Lilong Wang, Yankai Jiang, Xiaosong Wang, Xiao Sun, Dongzhan Zhou
- **Year**: 2026
- **Venue**: arXiv preprint
- **arXiv**: 2601.07641
- **Summary**: Proposes Test-Time Tool Evolution (TTE), enabling LLM agents to synthesize, verify, and evolve executable tools during inference rather than relying on static pre-defined tool libraries. Introduces SciEvo benchmark with 1,590 scientific reasoning tasks and 925 automatically evolved tools. Achieves state-of-the-art in accuracy and tool efficiency with effective cross-domain adaptation. Addresses a fundamental limitation: scientific domains have sparse, heterogeneous, and intrinsically incomplete tools.

---

## Dimension 2: IDE/CLI Extensions for Scientific Workflows

### 2.1 Experiences with Model Context Protocol Servers for Science and High Performance Computing
- **Authors**: Haochen Pan, Ryan Chard, Reid Mello, Christopher Grams, Tanjin He, Alexander Brace, Owen Price Skelly, Will Engler, Hayden Holbrook, Song Young Oh, Maxime Gonthier, Michael Papka, Ben Blaiszik, Kyle Chard, Ian Foster
- **Year**: 2025
- **Venue**: arXiv preprint
- **arXiv**: 2508.18489
- **Summary**: Reports on building MCP servers as a unifying interface for scientific research infrastructure at LBNL/NERSC. Wraps mature services (Globus Transfer, Compute, Search; facility status APIs; Octopus event fabric; Garden; Galaxy) with MCP adapters. Case studies in computational chemistry, bioinformatics, quantum chemistry, and filesystem monitoring demonstrate that agents can remove the need for custom glue code, generating it as needed. Key lesson: build thin MCP adapters for broad services rather than creating new ones.

### 2.2 MATLAB MCP Core Server
- **Authors**: MathWorks (official release)
- **Year**: 2025
- **Venue**: MathWorks Blog / GitHub
- **URL**: https://github.com/matlab/matlab-mcp-core-server
- **Summary**: Official MCP server from MathWorks enabling AI agents (Claude Code, VS Code Copilot) to start MATLAB, write and run code, and assess code quality. December 2025 update adds coding guidelines integration: agents auto-correct behavior based on MATLAB AI Coding Rules (e.g., using 1i for imaginary numbers, proper variable naming). Also enables hardware access via Image Acquisition Toolbox, making physical instruments part of the AI reasoning loop.

### 2.3 GitHub Copilot CLI (General Availability, February 2026)
- **Authors**: GitHub
- **Year**: 2026
- **Venue**: GitHub Changelog
- **URL**: https://github.blog/changelog/2026-02-25-github-copilot-cli-is-now-generally-available/
- **Summary**: Terminal-native agentic coding environment that plans, builds, reviews, and remembers across sessions. Supports Claude Opus 4.6, Claude Sonnet 4.6, GPT-5.3-Codex, and Gemini 3 Pro. Since public preview in September 2025, expanded from a terminal assistant into a full agentic development environment. Relevant for scientific workflows as it integrates with any CLI-driven simulation pipeline.

### 2.4 Claude and Codex on GitHub Agent HQ
- **Authors**: GitHub
- **Year**: 2026
- **Venue**: GitHub Blog
- **URL**: https://github.blog/news-insights/company-news/pick-your-agent-use-claude-and-codex-on-agent-hq/
- **Summary**: Claude (Anthropic) and OpenAI Codex available as coding agents for Copilot Pro+ and Enterprise customers. Agents can be started from issues, pull requests, and VS Code, enabling cross-client agentic workflows. For scientific software teams, this enables autonomous code review, feature implementation, and debugging of simulation codebases directly from GitHub issues.

### 2.5 InfraNodus VSCode/Cursor/Windsurf Extension
- **Authors**: Nodus Labs (Dmitry Paranyushkin)
- **Year**: 2025
- **Venue**: VSCode Marketplace / GitHub
- **URL**: https://github.com/infranodus/infranodus-vscode-extension
- **Summary**: Knowledge graph visualization extension for VSCode, Windsurf AI, and Cursor AI that builds network graphs from text/markdown files, applying network analysis metrics to detect structural gaps and reveal blind spots. Uses GPT-4/GPT-4o for AI analysis. Can be applied to research articles, PDFs, and Obsidian vaults to reveal main concepts, topics, and gaps between them. Currently in alpha (v0.5.6 as of Jan 2025).

### 2.6 Jupiter: Enhancing LLM Data Analysis Capabilities via Notebook and Inference-Time Value-Guided Search
- **Authors**: Shuocheng Li, Yihao Liu, Silin Du, Wenxuan Zeng, Zhe Xu, Mengyu Zhou, Yeye He, Haoyu Dong, Shi Han, Dongmei Zhang
- **Year**: 2025
- **Venue**: AAAI 2026
- **arXiv**: 2509.09245
- **Summary**: Formulates data analysis as a search problem using Monte Carlo Tree Search (MCTS) over Jupyter notebook workflows. Introduces NbQA, a large-scale dataset of task-solution pairs from real notebooks reflecting authentic tool-use patterns. Smaller models (Qwen2.5-7B/14B) trained with Jupiter match or surpass GPT-4o on InfiAgent-DABench (77.82% and 86.38% task solve rate). Demonstrates how IDE-integrated notebook workflows can be enhanced with search-based reasoning.

### 2.7 Do Large Language Models Speak Scientific Workflows?
- **Authors**: Orcun Yildiz, Tom Peterka
- **Year**: 2024 (updated 2025)
- **Venue**: arXiv preprint
- **arXiv**: 2412.10606
- **Summary**: Experimental study evaluating whether LLMs can handle scientific workflow tasks using current workflow systems. Tests multiple models (proprietary and open-source) across five workflow-specific experiments: configuring, annotating, translating, explaining, and generating scientific workflows. Finds that LLMs often struggle with workflow-related tasks due to lack of knowledge of scientific workflows, with performance varying by experiment and workflow system. Highlights the gap between general coding ability and domain-specific workflow expertise.

---

## Dimension 3: Scientific Software Maintenance with AI

### 3.1 DREAMS: Density Functional Theory Based Research Engine for Agentic Materials Simulation
- **Authors**: Ziqi Wang, Hongshuo Huang, Hancheng Zhao, Changwen Xu, Shang Zhu, Jan Janssen, Venkatasubramanian Viswanathan
- **Year**: 2025
- **Venue**: arXiv preprint
- **arXiv**: 2507.14267
- **Summary**: Hierarchical multi-agent framework for DFT simulation combining a central LLM planner with domain-specific agents for atomistic structure generation, systematic DFT convergence testing, HPC scheduling, and error handling. Achieves average errors below 1% on the Sol27LC lattice-constant benchmark compared to human DFT experts. Demonstrates L3-level automation (autonomous exploration of defined design space), substantially reducing the expertise barrier for high-throughput materials research.

### 3.2 MCP-SIM: A Self-Correcting Multi-Agent LLM Framework for Language-Based Physics Simulation and Explanation
- **Authors**: Donggeun Park, Hyeonbin Moon, Seunghwa Ryu
- **Year**: 2026
- **Venue**: npj Artificial Intelligence
- **arXiv**: (preprint June 2025)
- **Summary**: Multi-agent framework that transforms underspecified prompts into validated simulations via structured plan-act-reflect-revise cycles. Central shared memory system enables agent collaboration. Achieved 100% success on a 12-task benchmark across diverse physics domains, significantly outperforming baseline LLMs. Converges in fewer than 5 iterations even for complex problems. Produces interpretable language-localized reports explaining governing equations, boundary conditions, solver strategies, and assumptions.

### 3.3 ChronoLLM: Customizing Language Models for Physics-Based Simulation Code Generation
- **Authors**: Jingquan Wang, Andrew Negrut, Harry Zhang, Khailanii Slaton, Shu Wang, Radu Serban, Jinlong Wu, Dan Negrut
- **Year**: 2025
- **Venue**: arXiv preprint
- **arXiv**: 2508.13975
- **Summary**: Domain-specific LLM customized for PyChrono, an open-source multibody dynamics engine. Refined both open and closed-source LLMs through systematic fine-tuning, generating simulation scripts from natural language descriptions of mechatronic systems--ranging from simple pendulums to complex vehicle-terrain interactions. Generated code provides strong starting points for users; models can also answer API-specific questions and suggest modeling strategies. Approach is generalizable to other simulation tools.

### 3.4 Leveraging Large Language Models for Code Translation and Software Development in Scientific Computing (CodeScribe)
- **Authors**: Akash Dhruv, Anshu Dubey
- **Year**: 2024
- **Venue**: arXiv preprint / ACM
- **arXiv**: 2410.24119
- **Summary**: Developed CodeScribe, a tool combining prompt engineering with human supervision for code conversion in scientific computing. Applied to a legacy Fortran codebase for particle interaction simulations at the Large Hadron Collider. Addresses three tasks: Fortran-to-C++ translation, Fortran-C API creation for legacy system integration, and code organization/algorithm design. Demonstrates productivity gains while highlighting challenges requiring human verification.

### 3.5 Adding New Capability in Existing Scientific Application with LLM Assistance
- **Authors**: Anshu Dubey, Akash Dhruv
- **Year**: 2025
- **Venue**: 1st International Workshop on Foundational LLM Advances for HPC in Asia
- **arXiv**: 2511.00087
- **Summary**: Addresses the underexplored problem of writing code from scratch for new algorithms (without prior training examples) using LLM assistance. Enhances Code-Scribe to enable new code generation capabilities for scientific applications, going beyond translation to novel algorithm implementation. Focuses on generating entirely new physics capabilities for existing simulation frameworks.

### 3.6 LLM-Assisted Translation of Legacy FORTRAN Codes to C++: A Cross-Platform Study
- **Authors**: Nishath Rajiv Ranasinghe, Shawn M. Jones, Michal Kucer, Ayan Biswas, Daniel O'Malley, Alexander Buschmann Most, Selma Liliane Wanna, Ajay Sreekumar
- **Year**: 2025
- **Venue**: arXiv preprint
- **arXiv**: 2504.15424
- **Summary**: Systematic evaluation of LLM-based Fortran-to-C++ translation across multiple computational environments. Measures compilation success rates, semantic similarity between machine-translated and human-translated versions, and consistency of numerical outputs. Works toward developing automated workflows using open-source language models for HPC code migration.

### 3.7 HPC-Coder-V2: Studying Code LLMs Across Low-Resource Parallel Languages
- **Authors**: Aman Chaturvedi, Daniel Nichols, Siddharth Singh, Abhinav Bhatele
- **Year**: 2024
- **Venue**: arXiv preprint
- **arXiv**: 2412.15178
- **Summary**: In-depth study of fine-tuning specialized HPC LLMs for parallel code generation. Develops the best-performing open-source code LLM for parallel code generation (HPC-Coder-V2), fine-tuned on HPC-INSTRUCT (120k instruction-response pairs across C, Fortran, CUDA, MPI, OpenMP, Kokkos). Achieves pass@1 of 31.17 on ParEval parallel code generation tasks, outperforming all open-source models under 30B parameters.

### 3.8 MatTools: Benchmarking Large Language Models for Materials Science Tools
- **Authors**: Siyu Liu, Bo Hu, Beilin Ye, Jiamin Xu, David J. Srolovitz, Tongqi Wen
- **Year**: 2025
- **Venue**: arXiv preprint
- **arXiv**: 2505.10852
- **Summary**: Comprehensive benchmark for evaluating LLMs on materials science computational tasks. Includes a QA benchmark (69,225 pairs from pymatgen codebase/documentation) and a real-world tool-usage benchmark (49 tasks, 138 subtasks requiring functional Python code for materials property calculations). Three key findings: generalist models outperform domain specialists; AI-generated documentation helps AI; simpler prompting strategies work better.

### 3.9 SciAgents: Automating Scientific Discovery Through Multi-Agent Intelligent Graph Reasoning
- **Authors**: Alireza Ghafarollahi, Markus J. Buehler
- **Year**: 2024 (published 2025)
- **Venue**: Advanced Materials, 2025
- **arXiv**: 2409.05556
- **Summary**: Leverages large-scale ontological knowledge graphs, LLMs, and multi-agent systems with in-situ learning for automated scientific discovery. Applied to biologically inspired materials, reveals hidden interdisciplinary relationships. Autonomously generates and refines research hypotheses, elucidating mechanisms, design principles, and unexpected material properties. Implemented with AG2 (formerly AutoGen). Represents the paradigm of using knowledge graphs + LLMs for structured scientific reasoning.

---

## Cross-Cutting Themes

1. **Equation-as-code paradigm**: Multiple works (LLM-SR, CodePDE, AutoNumerics) treat scientific equations as programs, enabling LLMs to leverage code generation capabilities for scientific discovery.

2. **Multi-agent architectures dominate**: ATHENA, Lang-PINN, AutoNumerics, MCP-SIM, DREAMS all use specialized agent teams (planner, coder, critic, debugger) coordinated by a central dispatcher.

3. **Self-correction is essential**: Across PDE solving (CodePDE: 4 rounds), physics simulation (MCP-SIM: <5 iterations), and numerical methods (AutoNumerics: coarse-to-fine), iterative refinement is critical.

4. **MCP as scientific infrastructure**: The Model Context Protocol is emerging as a standard for connecting LLM agents to HPC resources, simulation tools, and scientific databases (LBNL/NERSC, MATLAB).

5. **Legacy code migration**: Active work on Fortran-to-C++ translation for HPC (CodeScribe, Ranasinghe et al.), but human oversight remains necessary for correctness.

6. **Tool evolution over tool libraries**: TTE (Test-Time Tool Evolution) represents a shift from static tool sets to dynamically synthesized, problem-specific computational tools.

---

## Key Venues

| Venue | Papers |
|-------|--------|
| ICLR 2025 (Oral) | LLM-SR |
| ICML 2025 (Oral) | LLM-SRBench |
| NeurIPS 2025 Workshop | ATHENA |
| AAAI 2026 | Jupiter |
| EACL 2026 Findings | SymCode |
| TMLR | CodePDE |
| npj Artificial Intelligence | MCP-SIM |
| Advanced Materials | SciAgents |
| arXiv (2025-2026) | PiT-PO, OpInf-LLM, PDE-SHARP, Lang-PINN, AutoNumerics, TTE, HPC-Coder-V2, MatTools, CodeScribe, Fortran-C++ |

---

*Generated 2026-03-10. 26 papers/systems surveyed across 3 dimensions.*
