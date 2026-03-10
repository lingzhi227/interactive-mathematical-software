# Phase 4: Code & Tools Survey
## LLMs for Transforming Scientific Knowledge into Human-Understandable Formats

**Survey date:** 2026-03-10
**Scope:** Open-source repositories, tools, and frameworks at the intersection of LLMs, scientific reasoning, symbolic math, knowledge representation, and code generation for science.

---

## Table of Contents
1. [Symbolic Regression & Equation Discovery](#1-symbolic-regression--equation-discovery)
2. [Scientific Benchmarks](#2-scientific-benchmarks)
3. [Formal Verification & Autoformalization](#3-formal-verification--autoformalization)
4. [Scientific Simulation & PDE Solvers](#4-scientific-simulation--pde-solvers)
5. [Document Parsing & Knowledge Extraction](#5-document-parsing--knowledge-extraction)
6. [Scientific Computing Platforms & MCP Servers](#6-scientific-computing-platforms--mcp-servers)
7. [Knowledge Graphs & Visualization](#7-knowledge-graphs--visualization)
8. [Foundational Symbolic Math](#8-foundational-symbolic-math)
9. [Materials Science & DFT Agents](#9-materials-science--dft-agents)
10. [Curated Surveys & Meta-Resources](#10-curated-surveys--meta-resources)

---

## 1. Symbolic Regression & Equation Discovery

### 1.1 LLM-SR
| Field | Value |
|-------|-------|
| **URL** | https://github.com/deep-symbolic-mathematics/LLM-SR |
| **Stars** | ~215 |
| **Language** | Python |
| **Last Updated** | ~2025 (created Apr 2024) |
| **License** | MIT |
| **Venue** | ICLR 2025 Oral |

**Description:** Official implementation of "LLM-SR: Scientific Equation Discovery via Programming with Large Language Models." Combines LLMs' scientific knowledge and code generation with evolutionary search to discover interpretable equations from data. Represents equations as program skeletons, enabling flexible hypothesis generation guided by domain-specific priors. Benchmarked on physics, biology, and materials science problems.

**Relevance:** Core example of using LLMs to translate scientific knowledge (physics priors) into symbolic equations via code generation. Demonstrates the LLM-as-scientist paradigm for equation discovery.

---

### 1.2 LLM-SRBench
| Field | Value |
|-------|-------|
| **URL** | https://github.com/deep-symbolic-mathematics/llm-srbench |
| **Stars** | ~94 |
| **Language** | Python |
| **Last Updated** | Jun 2025 |
| **License** | MIT |
| **Venue** | ICML 2025 Oral |

**Description:** Comprehensive benchmark with 239 challenging problems across four scientific domains for evaluating LLM-based scientific equation discovery. Includes LSR-Transform (transforms common physical models into less common representations) and LSR-Synth (synthetic discovery-driven problems). Best system achieves only 31.5% symbolic accuracy, highlighting remaining challenges.

**Relevance:** Standardized evaluation framework for measuring how well LLMs can transform data into scientific equations. Companion to LLM-SR.

---

### 1.3 PySR
| Field | Value |
|-------|-------|
| **URL** | https://github.com/MilesCranmer/PySR |
| **Stars** | ~3,400 |
| **Language** | Python (Julia backend via SymbolicRegression.jl) |
| **Last Updated** | Active (2,823+ commits) |
| **License** | Apache-2.0 |

**Description:** High-performance symbolic regression library with a scikit-learn-style Python interface backed by an optimized Julia search engine. Finds interpretable symbolic expressions that fit data. Widely used in astrophysics, materials science, and other domains. Published in Advances in Neural Information Processing Systems.

**Relevance:** Foundational tool for symbolic regression that LLM-based systems (like KeplerAgent) build upon. Bridges machine learning and scientific equation discovery with human-interpretable outputs.

---

### 1.4 KeplerAgent
| Field | Value |
|-------|-------|
| **URL** | https://arxiv.org/abs/2602.12259 (paper; code repository not yet publicly released) |
| **Stars** | N/A |
| **Language** | Python (expected) |
| **Last Updated** | Feb 2026 (paper) |
| **License** | N/A |

**Description:** Physics-guided LLM agent for equation discovery from the paper "Think like a Scientist." Uses LLMs as agents that orchestrate physics-based tools (symmetry estimation, functional term identification) to configure symbolic regression engines like PySINDy and PySR. Achieves higher symbolic accuracy and greater robustness to noisy data than both LLM-only and traditional baselines.

**Relevance:** Directly addresses using LLMs to transform observational data into scientific equations by emulating the multi-step workflow of human scientists. Combines physics reasoning with tool use.

---

## 2. Scientific Benchmarks

### 2.1 SciCode
| Field | Value |
|-------|-------|
| **URL** | https://github.com/scicode-bench/SciCode |
| **Stars** | ~179 |
| **Language** | Python |
| **Last Updated** | Feb 2025 |
| **License** | Apache-2.0 |
| **Venue** | NeurIPS 2024 (Datasets & Benchmarks) |

**Description:** Research coding benchmark curated by scientists, covering 16 subdomains from 5 domains (Physics, Math, Material Science, Biology, Chemistry). Contains 338 subproblems from 80 main problems, each requiring knowledge recall, reasoning, and code synthesis. Best model (o1-preview) solves only 7.7% in the most realistic setting.

**Relevance:** Directly benchmarks LLMs' ability to transform scientific problems into working code. Measures the gap between current LLM capabilities and real scientific computing tasks.

---

## 3. Formal Verification & Autoformalization

### 3.1 PhysLean (formerly HepLean)
| Field | Value |
|-------|-------|
| **URL** | https://github.com/HEPLean/PhysLean |
| **Stars** | ~507 |
| **Language** | Lean 4 (99.3%) |
| **Last Updated** | Active (2,531+ commits) |
| **License** | Apache-2.0 |

**Description:** A project to digitalize results from physics into the Lean 4 proof assistant. Covers Maxwell's equations, quantum harmonic oscillator, statistical mechanics, condensed matter, special relativity, the Standard Model Higgs boson, Pati-Salam model, and more. Published in Computer Physics Communications (2025).

**Relevance:** Premier example of transforming physics knowledge into machine-verifiable formal code. Enables rigorous, computer-checked representations of physical theories -- a key modality for "human-understandable" scientific knowledge.

---

### 3.2 MerLean
| Field | Value |
|-------|-------|
| **URL** | https://arxiv.org/abs/2602.16554 / https://arthurmerlean.com/ / https://doxtor6.github.io/MerLean-examples/ |
| **Stars** | N/A (code examples available, full framework not yet open-sourced) |
| **Language** | Lean 4, Python (Claude Code-based agent) |
| **Last Updated** | Feb 2026 (paper) |
| **License** | N/A |

**Description:** Fully automated agentic framework for autoformalization in quantum computation. Extracts mathematical statements from LaTeX, formalizes them into verified Lean 4 code on Mathlib, and translates results back to LaTeX for semantic review. Evaluated on three quantum computing papers, producing 2,050 Lean declarations from 114 statements. Uses Claude Code as the frontier LLM agent with multi-turn compiler feedback.

**Relevance:** End-to-end pipeline for transforming scientific papers (LaTeX) into formally verified code (Lean 4) and back to natural language. Exemplifies bidirectional translation between human-readable and machine-verifiable scientific knowledge.

---

### 3.3 Lean LSP MCP
| Field | Value |
|-------|-------|
| **URL** | https://github.com/oOo0oOo/lean-lsp-mcp |
| **Stars** | ~312 |
| **Language** | Python |
| **Last Updated** | Active (486 commits) |
| **License** | MIT |

**Description:** MCP server enabling agentic interaction with the Lean theorem prover via the Language Server Protocol. Provides tools for diagnostics, goal states, term info, hover docs, and external search (LeanSearch, Loogle, Lean Finder, Lean Hammer). Works out-of-the-box with Claude Code, Cursor, and other MCP clients.

**Relevance:** Infrastructure for LLM agents to interact with formal mathematical verification. Enables AI-assisted formalization of scientific knowledge in Lean 4.

---

## 4. Scientific Simulation & PDE Solvers

### 4.1 MCP-SIM
| Field | Value |
|-------|-------|
| **URL** | https://github.com/KAIST-M4/MCP-SIM |
| **Stars** | ~9 |
| **Language** | Python |
| **Last Updated** | 2025 |
| **License** | MIT |
| **Venue** | npj Artificial Intelligence (2025) |

**Description:** Self-correcting multi-agent LLM framework for language-based physics simulation and explanation. Transforms underspecified natural language prompts into validated FEniCS simulations with explanatory reports. Five specialized agents (Input Clarifier, Code Builder, Simulation Executor, Error Diagnosis, Mechanical Insight) collaborate via shared memory. Achieves 100% success on a 12-task benchmark across linear elasticity, heat conduction, fluid flow, thermo-mechanical coupling, piezoelectric, and fracture mechanics.

**Relevance:** Directly converts natural language physics descriptions into executable simulation code and human-readable explanations. Multi-agent architecture for scientific computing.

---

### 4.2 CodePDE
| Field | Value |
|-------|-------|
| **URL** | https://github.com/LithiumDA/CodePDE |
| **Stars** | ~71 |
| **Language** | Python |
| **Last Updated** | May 2025 (created) |
| **License** | Not specified |

**Description:** Inference framework for LLM-driven PDE solver generation. Frames PDE solving as a code generation task with three modes: repeated sampling (from scratch), refinement (improve existing solvers), and funsearch (evolutionary search). Supports finite difference, spectral methods, and novel approaches. Achieves superhuman performance on representative PDE problems. Includes code repair, iterative refinement, and test-time scaling.

**Relevance:** Transforms mathematical PDE specifications into executable solver code using LLMs. Demonstrates that LLMs can generate scientific computing code that exceeds human-written solvers.

---

## 5. Document Parsing & Knowledge Extraction

### 5.1 Nougat
| Field | Value |
|-------|-------|
| **URL** | https://github.com/facebookresearch/nougat |
| **Stars** | ~9,900 |
| **Language** | Python |
| **Last Updated** | Aug 2023 |
| **License** | MIT (code), CC-BY-NC (model weights) |

**Description:** Neural Optical Understanding for Academic Documents. Visual Transformer model for OCR of scientific documents, converting PDFs into structured markup (LaTeX/Markdown) with accurate math equation parsing. Installable via pip (`nougat-ocr`). Processes single files or directories of PDFs.

**Relevance:** Critical infrastructure for extracting mathematical knowledge from scientific papers. Enables downstream LLM processing by converting visual equations into machine-readable LaTeX -- a key step in the knowledge transformation pipeline.

---

### 5.2 arXiv-LaTeX-MCP
| Field | Value |
|-------|-------|
| **URL** | https://github.com/takashiishida/arxiv-latex-mcp |
| **Stars** | ~107 |
| **Language** | Python |
| **Last Updated** | Feb 2026 |
| **License** | MIT |

**Description:** MCP server that fetches and processes arXiv LaTeX sources for precise interpretation of mathematical expressions in scientific papers. Uses `arxiv-to-prompt` to provide LaTeX source code rather than PDFs, which better preserves mathematical notation for LLM interpretation. Compatible with Claude Desktop, Claude Code, Cursor, and other MCP clients.

**Relevance:** Directly addresses the challenge of making scientific papers (especially equations) accessible to LLMs. Bridges the gap between published research and LLM-readable mathematical formats.

---

## 6. Scientific Computing Platforms & MCP Servers

### 6.1 MATLAB MCP Core Server
| Field | Value |
|-------|-------|
| **URL** | https://github.com/matlab/matlab-mcp-core-server |
| **Stars** | ~224 |
| **Language** | Go (97.9%), MATLAB |
| **Last Updated** | Mar 2026 (v0.6.0) |
| **License** | BSD-3-Clause |

**Description:** Official MCP server from MathWorks enabling AI applications to start MATLAB, write and run MATLAB code, detect toolboxes, and check code for issues. Supports Claude Code, VS Code, and other MCP-compatible agents. Free and open-source, requiring only a local MATLAB installation.

**Relevance:** Enables LLM agents to execute scientific computing workflows in MATLAB. Extends coding assistants for engineering and scientific domains where MATLAB is the standard tool.

---

### 6.2 Jupyter AI
| Field | Value |
|-------|-------|
| **URL** | https://github.com/jupyterlab/jupyter-ai |
| **Stars** | ~4,100 |
| **Language** | Python (65.7%), Shell (34.3%) |
| **Last Updated** | Nov 2025 (v2.31.7) |
| **License** | BSD-3-Clause |

**Description:** Generative AI extension for JupyterLab providing an `%%ai` magic command and native chat UI. Supports 1000+ LLMs out-of-the-box via LiteLLM (Anthropic, OpenAI, AWS, Cohere, Gemini, Hugging Face, NVIDIA, etc.). Turns Jupyter notebooks into reproducible AI playgrounds for scientific computing.

**Relevance:** Primary interface for integrating LLMs into the scientific computing workflow. Enables researchers to use AI assistants for code generation, data analysis, and scientific reasoning directly within their notebook environment.

---

### 6.3 MCP.Science
| Field | Value |
|-------|-------|
| **URL** | https://github.com/pathintegral-institute/mcp.science |
| **Stars** | ~116 |
| **Language** | Python (98.1%) |
| **Last Updated** | Jul 2025 (v0.2.0) |
| **License** | MIT |

**Description:** Collection of open-source MCP servers for scientific research. Includes specialized servers for materials science (Materials Project database), Python code execution, web content fetching, and computational chemistry (GPAW/DFT calculations). Enables AI models to interact with scientific data, tools, and computational resources.

**Relevance:** Purpose-built MCP infrastructure for scientific computing. Allows LLM agents to directly query scientific databases, run computations, and process research data.

---

## 7. Knowledge Graphs & Visualization

### 7.1 InfraNodus
| Field | Value |
|-------|-------|
| **URL** | https://github.com/noduslabs/infranodus (legacy open-source) / https://infranodus.com (current) |
| **Stars** | ~12 (open-source version) |
| **Language** | JavaScript (Node.js), Neo4J |
| **Last Updated** | 2020 (open-source); Jan 2026 (commercial version) |
| **License** | Custom (open-source version) |

**Description:** Text-to-knowledge-graph tool that represents words as nodes and co-occurrences as edges, then applies graph theory algorithms for topic modeling, identifying influential concepts, and revealing discourse gaps. The commercial version (infranodus.com) adds AI integration (Claude, GPT, Gemini, Grok) with GraphRAG for augmented knowledge exploration. Also available as Obsidian plugin and VS Code extension.

**Relevance:** Demonstrates knowledge graph visualization of textual/scientific content. Relevant for transforming unstructured scientific text into structured knowledge graph representations.

---

## 8. Foundational Symbolic Math

### 8.1 SymPy
| Field | Value |
|-------|-------|
| **URL** | https://github.com/sympy/sympy |
| **Stars** | ~14,500 |
| **Language** | Python (98.7%) |
| **Last Updated** | Active (61,861+ commits; v1.14.0 Apr 2025) |
| **License** | BSD-3-Clause |

**Description:** Pure Python computer algebra system covering symbolic arithmetic, calculus, algebra, discrete mathematics, quantum physics, and more. The foundational library for symbolic mathematics in the Python ecosystem. Used as a backend by many LLM-based scientific tools for equation manipulation, simplification, and verification.

**Relevance:** Foundational infrastructure that LLMs use to manipulate, simplify, and verify mathematical expressions. Essential building block for any system that transforms scientific knowledge into symbolic form.

---

## 9. Materials Science & DFT Agents

### 9.1 DREAMS (material_agent)
| Field | Value |
|-------|-------|
| **URL** | https://github.com/BattModels/material_agent |
| **Stars** | ~22 |
| **Language** | Python (99.8%) |
| **Last Updated** | Active (89 commits) |
| **License** | Not specified |

**Description:** Density Functional Theory Based Research Engine for Agentic Materials Simulation. Hierarchical multi-agent framework combining a central LLM planner with domain-specific agents for structure generation, DFT convergence testing, HPC scheduling, and error handling. Validated on Sol27LC lattice-constant benchmark (<1% error vs. human DFT experts) and the CO/Pt(111) adsorption puzzle.

**Relevance:** Demonstrates LLM agents transforming materials science problems into executable DFT simulations. Converts research objectives into computational workflows autonomously, bridging scientific reasoning and code execution.

---

## 10. Curated Surveys & Meta-Resources

### 10.1 Awesome-LLM-Scientific-Discovery
| Field | Value |
|-------|-------|
| **URL** | https://github.com/HKUST-KnowComp/Awesome-LLM-Scientific-Discovery |
| **Stars** | ~308 |
| **Language** | Markdown (curated paper list) |
| **Last Updated** | Active (2025) |
| **License** | Available (not specified) |
| **Venue** | EMNLP 2025 |

**Description:** Companion repository for the survey "From Automation to Autonomy: A Survey on Large Language Models in Scientific Discovery." Organizes the field through a three-level autonomy framework: LLM as Tool, LLM as Analyst, and LLM as Scientist. Covers literature review, hypothesis generation, experimentation, and data analysis across scientific domains.

**Relevance:** Comprehensive meta-resource mapping the landscape of LLMs in scientific discovery. Provides taxonomy and references for all major approaches to using LLMs for scientific reasoning and knowledge transformation.

---

## Summary Table

| # | Repository | Stars | Language | License | Key Category |
|---|-----------|-------|----------|---------|-------------|
| 1 | [LLM-SR](https://github.com/deep-symbolic-mathematics/LLM-SR) | ~215 | Python | MIT | Equation Discovery |
| 2 | [LLM-SRBench](https://github.com/deep-symbolic-mathematics/llm-srbench) | ~94 | Python | MIT | Equation Discovery Benchmark |
| 3 | [PySR](https://github.com/MilesCranmer/PySR) | ~3,400 | Python/Julia | Apache-2.0 | Symbolic Regression |
| 4 | KeplerAgent | N/A | Python | N/A | Physics-Guided Discovery |
| 5 | [SciCode](https://github.com/scicode-bench/SciCode) | ~179 | Python | Apache-2.0 | Scientific Coding Benchmark |
| 6 | [PhysLean](https://github.com/HEPLean/PhysLean) | ~507 | Lean 4 | Apache-2.0 | Physics Formalization |
| 7 | MerLean | N/A | Lean 4/Python | N/A | Quantum Autoformalization |
| 8 | [Lean LSP MCP](https://github.com/oOo0oOo/lean-lsp-mcp) | ~312 | Python | MIT | Lean Prover MCP |
| 9 | [MCP-SIM](https://github.com/KAIST-M4/MCP-SIM) | ~9 | Python | MIT | Physics Simulation |
| 10 | [CodePDE](https://github.com/LithiumDA/CodePDE) | ~71 | Python | N/S | PDE Solver Generation |
| 11 | [Nougat](https://github.com/facebookresearch/nougat) | ~9,900 | Python | MIT/CC-BY-NC | PDF-to-LaTeX OCR |
| 12 | [arXiv-LaTeX-MCP](https://github.com/takashiishida/arxiv-latex-mcp) | ~107 | Python | MIT | Paper Math Extraction |
| 13 | [MATLAB MCP](https://github.com/matlab/matlab-mcp-core-server) | ~224 | Go/MATLAB | BSD-3-Clause | Scientific Computing MCP |
| 14 | [Jupyter AI](https://github.com/jupyterlab/jupyter-ai) | ~4,100 | Python | BSD-3-Clause | LLM Notebook Extension |
| 15 | [MCP.Science](https://github.com/pathintegral-institute/mcp.science) | ~116 | Python | MIT | Scientific MCP Servers |
| 16 | [InfraNodus](https://github.com/noduslabs/infranodus) | ~12 | JavaScript | Custom | Knowledge Graphs |
| 17 | [SymPy](https://github.com/sympy/sympy) | ~14,500 | Python | BSD-3-Clause | Symbolic Math |
| 18 | [DREAMS](https://github.com/BattModels/material_agent) | ~22 | Python | N/S | DFT Agent |
| 19 | [Awesome-LLM-Sci-Discovery](https://github.com/HKUST-KnowComp/Awesome-LLM-Scientific-Discovery) | ~308 | Markdown | N/S | Survey/Meta-Resource |

---

## Key Themes Identified

### Theme 1: LLMs as Equation Discovery Engines
LLM-SR, LLM-SRBench, PySR, and KeplerAgent collectively show a maturing pipeline: LLMs generate hypotheses as code/symbolic expressions, evolutionary search refines them, and physics priors constrain the search space. The best systems combine LLM knowledge with traditional symbolic regression backends.

### Theme 2: Autoformalization -- Bridging Natural Language and Formal Proofs
PhysLean, MerLean, and Lean LSP MCP represent the formalization direction: translating physics from papers/textbooks into machine-verifiable Lean 4 code. MerLean's bidirectional pipeline (LaTeX -> Lean -> LaTeX) is particularly notable for closing the loop between human-readable and machine-verifiable representations.

### Theme 3: MCP as Scientific Computing Glue
The Model Context Protocol is emerging as the standard interface for connecting LLMs to scientific tools. MATLAB MCP, Lean LSP MCP, MCP.Science, arXiv-LaTeX-MCP, and MCP-SIM all use this pattern. This enables composable, tool-augmented scientific reasoning.

### Theme 4: Multi-Agent Architectures for Complex Science
MCP-SIM, DREAMS, and KeplerAgent all use multi-agent architectures where specialized agents handle different aspects of the scientific workflow (input clarification, code generation, error diagnosis, execution, explanation). This mirrors how scientific teams work.

### Theme 5: Document-to-Knowledge Pipelines
Nougat and arXiv-LaTeX-MCP address the ingestion side: converting scientific PDFs and papers into machine-readable formats that preserve mathematical notation. These are prerequisites for any downstream LLM reasoning over scientific content.

---

## Additional Resources Found via Broad Searches

- **Papers with Code -- Equation Discovery task:** https://paperswithcode.com/task/equation-discovery (aggregates benchmarks and leaderboards for symbolic regression)
- **StepFun-Formalizer:** https://github.com/stepfun-ai/StepFun-Formalizer (LLM family for natural language to Lean 4 autoformalization)
- **LeanEuclid:** https://github.com/loganrjmurphy/LeanEuclid (benchmark for autoformalization in Euclidean geometry, Lean 4)
- **PDA (Process-Driven Autoformalization):** https://github.com/rookie-joe/PDA (autoformalization in Lean 4 with FormL4 dataset)
- **Awesome-Scientific-Language-Models:** https://github.com/yuzhimanhua/Awesome-Scientific-Language-Models (EMNLP 2024 survey companion)

---

*Generated 2026-03-10 as part of deep research survey Phase 4 (Code & Tools).*
