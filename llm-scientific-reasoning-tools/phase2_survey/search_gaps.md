# Phase 2 Survey: Gap-Filling Search Results

## Research Topic
Using LLMs to transform scientific knowledge (math equations, physics theories) into human-understandable formats (knowledge trees, graphs, natural language, code) -- extending coding assistants for scientific reasoning.

**Date:** 2026-03-10
**Scope:** 6 gap areas identified from Phase 1, targeting 10-15 papers

---

## Gap 1: LaTeX Parsing and Understanding by LLMs

### Paper 1: Nougat -- Neural Optical Understanding for Academic Documents

- **Authors:** Lukas Blecher, Guillem Cucurull, Thomas Scialom, Robert Stojnic (Meta AI)
- **Date:** August 2023 (arXiv: 2308.13418)
- **Venue:** ECCV 2024 (accepted after arXiv preprint)
- **URL:** https://arxiv.org/abs/2308.13418
- **Code:** https://github.com/facebookresearch/nougat

**Abstract:** Presents a Visual Transformer model that performs OCR on scientific documents, converting PDF pages into structured markup language. The key problem addressed is that scientific knowledge stored in PDFs loses semantic information, particularly for mathematical expressions. Nougat uses a Swin Transformer vision encoder and an mBART-based text decoder to predict markdown (including LaTeX math) directly from raw pixels of scientific PDFs. Trained on papers from arXiv and PubMed Central.

**Relevance:** Foundational work for the first step in any pipeline that transforms scientific knowledge from published documents. Enables extraction of LaTeX equations from PDF papers, which can then be fed to LLMs for further processing (symbolic conversion, code generation, explanation).

---

### Paper 2: ASyMOB -- Algebraic Symbolic Mathematical Operations Benchmark

- **Authors:** Michael Shalyt, Rotem Elimelech, Ido Kaminer
- **Date:** May 2025 (arXiv: 2505.23851)
- **URL:** https://arxiv.org/abs/2505.23851
- **Code:** https://github.com/RamanujanMachine/ASyMOB

**Abstract:** A benchmark with 17,092 unique symbolic math challenges organized by similarity and complexity, designed to test LLMs on core symbolic manipulation skills (integration, differential equations, algebraic simplification). Uses LaTeX-to-SymPy parsing as a key evaluation component -- the final LaTeX answer is extracted via regex and parsed into SymPy through a deterministic function, falling back to an LLM when the parser fails (18.5% of cases). Key finding: LLMs exhibit up to 70.3% performance degradation on perturbed problems, suggesting reliance on memorized patterns. Top models (o4-mini, Gemini 2.5 Flash) score 96-97% on unperturbed problems but still drop 21% on perturbations. Code execution integration improves accuracy by up to 33%.

**Relevance:** Directly tests whether LLMs truly understand symbolic math or merely pattern-match. The LaTeX-to-SymPy pipeline and LLM fallback mechanism are a practical example of hybrid deterministic+LLM equation processing.

---

### Paper 3: TeXpert -- A Multi-Level Benchmark for Evaluating LaTeX Code Generation by LLMs

- **Authors:** Sahil Kale, Vijaykant Nadadur
- **Date:** June 2025 (arXiv: 2506.16990)
- **Venue:** SDProc Workshop @ ACL 2025
- **URL:** https://arxiv.org/abs/2506.16990

**Abstract:** Introduces a benchmark for evaluating LLMs on generating accurate LaTeX code from natural language instructions, covering diverse components commonly found in scientific documents (not just math equations). Finds that models performing well on standard coding benchmarks often struggle significantly with LaTeX generation. Open-source models like DeepSeek v3 compete favorably with closed-source alternatives. Formatting and package errors are disproportionately common, suggesting insufficient LaTeX diversity in training data.

**Relevance:** Evaluates the reverse direction -- from natural language description of scientific content to LaTeX representation. Important for understanding LLM capability in the full bidirectional pipeline between human-understandable formats and formal mathematical notation.

---

### Paper 4 (Tool): Mathpix -- Document Conversion for STEM

- **URL:** https://mathpix.com/
- **Type:** Commercial tool/API (not a research paper)

**Description:** Mathpix provides OCR-powered conversion of PDFs and images containing STEM content to searchable, machine-readable text (LaTeX, DOCX, Markdown). Achieves ~99% accuracy on printed mathematical equations and 75-90% on handwritten equations. Offers a REST API for integration into scientific workflows. In 2025, added improved handwriting recognition, spell-checking, and Code OCR features.

**Relevance:** Production-grade tool that solves the practical problem of extracting equations from scientific documents. Complementary to Nougat (research) -- represents the commercial state of the art for LaTeX extraction.

---

## Gap 2: Scientific Workflow Systems + AI

### Paper 5: From Prompt to Pipeline -- Large Language Models for Scientific Workflow Development in Bioinformatics

- **Authors:** Khairul Alam, Banani Roy
- **Date:** July 2025 (arXiv: 2507.20122)
- **URL:** https://arxiv.org/abs/2507.20122

**Abstract:** Investigates whether modern LLMs can assist in generating accurate, complete, and usable bioinformatics workflows. Evaluates GPT-4o, Gemini 2.5 Flash, and DeepSeek-V3 on generating both Galaxy (graphical) and Nextflow (script-based) workflows for diverse bioinformatics tasks (SNP analysis, RNA-seq, DNA methylation, data retrieval). Tests escalating prompting strategies (basic, role-based, chain-of-thought). Results: Gemini 2.5 Flash produced the most accurate Galaxy workflows; DeepSeek-V3 excelled at Nextflow pipeline generation. Prompting strategy significantly influenced output quality, with role-based and CoT prompts improving correctness and completeness.

**Relevance:** Directly addresses the question of whether LLMs can translate scientific intent into executable computational pipelines. Demonstrates that well-prompted LLMs can generate production-quality workflow code for established scientific workflow systems, reducing barriers for both novice and expert users.

---

## Gap 3: Computational Notebooks + LLMs

### Paper 6 (Tool): Jupyter AI -- Generative AI Extension for JupyterLab

- **Authors:** Project Jupyter (Jason Weill et al.)
- **Date:** v1.0 released 2023; v2.0 for JupyterLab 4 (2024); v3 beta (2025)
- **URL:** https://github.com/jupyterlab/jupyter-ai
- **Blog:** https://blog.jupyter.org/generative-ai-in-jupyter-3f7174824862

**Description:** Official Jupyter subproject that brings generative AI to notebooks via two interfaces: (1) a chat UI in JupyterLab for conversational code assistance, and (2) an `%%ai` magic command that turns notebooks into reproducible generative AI playgrounds. Supports providers including AI21, Anthropic, AWS, Cohere, Gemini, HuggingFace, MistralAI, NVIDIA, and OpenAI. Vendor-neutral design. Capabilities include explaining code, generating code, fixing errors, summarizing content, querying local files, and generating entire notebooks from natural language prompts.

**Relevance:** The de facto standard for integrating LLMs into the computational notebook environment used by millions of scientists. Enables scientists to interact with equations, data, and code through natural language within their existing workflow. The `%%ai` magic command is particularly relevant for scientific reasoning -- it allows inline LLM calls within computational narratives.

---

### Paper 7 (Tool): Google NotebookLM -- Source-Grounded AI Research Assistant

- **URL:** https://notebooklm.google/
- **Type:** Commercial tool (Google Labs)

**Description:** An AI-powered research tool built on Google Gemini that enables users to upload scientific documents and interact with them through natural language. Unique "source-grounded" approach constrains responses to uploaded sources, reducing hallucination. Key features for scientific work include: (1) Audio Overviews (Sep 2024) that convert complex documents into podcast-style discussions; (2) Video Overviews (2025) with AI narration and diagrams; (3) Infographics and Slide Deck generation (Nov 2025). Supports specifying which sources each prompt can use, with responses containing references to specific PDFs.

**Relevance:** Represents a different paradigm from code-centric notebooks -- focuses on synthesizing and explaining scientific knowledge from documents rather than generating executable code. The source-grounding approach is important for scientific accuracy. Audio/video overviews are a novel format for making scientific knowledge more accessible.

---

## Gap 4: Quantum Mechanics / DFT Simulation with AI

### Paper 8: DREAMS -- Density Functional Theory Based Research Engine for Agentic Materials Simulation

- **Authors:** Ziqi Wang, Hongshuo Huang, Hancheng Zhao, Changwen Xu, Shang Zhu, Jan Janssen, Venkatasubramanian Viswanathan
- **Date:** July 2025 (arXiv: 2507.14267)
- **URL:** https://arxiv.org/abs/2507.14267
- **Code:** https://github.com/BattModels/material_agent

**Abstract:** A hierarchical multi-agent framework for DFT simulation that combines a central LLM planner (Claude 3.5/3.7 via LangGraph) with domain-specific LLM agents for: atomistic structure generation, systematic DFT convergence testing, HPC scheduling, and error handling. Uses ASE and Quantum ESPRESSO for calculations. A shared canvas preserves context and prevents hallucination. Validated on Sol27LC lattice-constant benchmark (average errors <1% vs. human DFT experts) and CO/Pt(111) adsorption problem (reproducing expert-level results). Also uses Bayesian ensemble sampling for functional uncertainty quantification. Represents "L3-level automation" -- autonomous exploration of a defined design space.

**Relevance:** Premier example of LLMs orchestrating complex scientific simulations (DFT calculations). Shows how LLMs can translate high-level materials science questions into concrete computational workflows, managing the full pipeline from structure generation to convergence testing to result analysis. Directly relevant to transforming physics theory into executable computation.

---

### Paper 9: Aitomia -- Intelligent Assistant for AI-Driven Atomistic and Quantum Chemical Simulations

- **Authors:** Jinming Hu, Hassan Nawaz, Yuting Rui, Lijie Chi, Arif Ullah, Pavlo O. Dral
- **Date:** May 2025 (arXiv: 2505.08195, v3 July 2025)
- **URL:** https://arxiv.org/abs/2505.08195
- **Platform:** https://aitomistic.com (publicly available since May 11, 2025)

**Abstract:** An AI-powered platform with chatbots and multi-agent architecture for performing atomistic and quantum chemical simulations. Leverages LLMs integrated with the MLatom ecosystem, supporting tasks from ground-state to excited-state calculations including geometry optimizations, thermochemistry, IR and UV/vis spectral calculations. Uses pre-trained universal ML models (AIQM, ANI, OMNI-P, AIMnet, MACE-OFF). Multi-agent implementation enables autonomous execution of complex workflows (e.g., reaction enthalpy computation). Accessible as cloud service or local deployment. First publicly available intelligent assistant for broad-scope atomistic simulations.

**Relevance:** Demonstrates a natural language interface to quantum chemistry calculations -- users describe what they want in plain language, and the AI agent handles setup, execution, and result analysis. A practical realization of transforming scientific intent into computational results.

---

### Paper 10: Developing Large Language Models for Quantum Chemistry Simulation Input Generation

- **Authors:** (Published in Digital Discovery, RSC Publishing)
- **Date:** Preprint September 2024 (ChemRxiv); Published February 2025 (Digital Discovery)
- **URL:** https://pubs.rsc.org/en/content/articlehtml/2025/dd/d4dd00366g
- **ChemRxiv:** https://chemrxiv.org/engage/chemrxiv/article-details/66d20058a4e53c487628f63b

**Abstract:** Investigates the potential of foundational LLMs for generating input files for the quantum chemistry package ORCA. Establishes a general framework adaptable to other domain-specific languages. Explores prompt engineering, retrieval-augmented generation (RAG), and finetuning with synthetic datasets. Key finding: finetuning with as few as 500 synthetic samples significantly improves performance, with synergistic effects when combined with chain-of-thought prompting. The best finetuned models outperform GPT-4o. Synthetic data was generated by brute-force combination of ORCA manual keywords/options.

**Relevance:** Addresses the specific challenge of translating scientific intent into the domain-specific language of quantum chemistry software. The finetuning-on-synthetic-data approach is generalizable to other scientific simulation codes (VASP, Gaussian, etc.). Shows that even small, targeted datasets can dramatically improve LLM performance on specialized scientific code generation.

---

## Gap 5: Cross-Domain Knowledge Transfer Math <-> Physics

### Paper 11: Knowledge or Reasoning? A Close Look at How LLMs Think Across Domains

- **Authors:** Juncheng Wu, Sheng Liu, Haoqin Tu, Hang Yu, Xiaoke Huang, James Zou, Cihang Xie, Yuyin Zhou
- **Date:** June 2025 (arXiv: 2506.02126)
- **URL:** https://arxiv.org/abs/2506.02126

**Abstract:** Investigates how advanced reasoning models (OpenAI-o1/o3, DeepSeek-R1) handle complex tasks by separating step-by-step thinking into two components: knowledge correctness and reasoning quality. Studies cross-domain transfer between mathematical and medical domains. Three key findings: (1) Mathematical reasoning does not naturally transfer to the medical domain via supervised fine-tuning (SFT), due to domain-specific knowledge gaps; (2) SFT improves accuracy but often compromises reasoning quality (information gain drops substantially); (3) Reinforcement learning is more effective in new domains by refining knowledge and eliminating inaccurate information from reasoning chains. The drop from math to physics is qualitative, not just quantitative -- physics demands situational modeling and causal grounding beyond formal manipulation.

**Relevance:** Directly addresses the challenge of cross-domain knowledge transfer. The finding that mathematical reasoning skill is "domain-locked" has important implications for building LLM systems that bridge math and physics -- it suggests that simply training on math problems will not produce good physics reasoning. Different training strategies (RL vs. SFT) are needed for different domains.

---

### Paper 12: UGPhysics -- A Comprehensive Benchmark for Undergraduate Physics Reasoning with Large Language Models

- **Authors:** Xin Xu, Qiyun Xu, Tong Xiao, Tianhao Chen, Yuchen Yan, Jiaxing Zhang, Shizhe Diao, Can Yang, Yang Wang
- **Date:** February 2025 (arXiv: 2502.00334); Accepted ICML 2025
- **URL:** https://arxiv.org/abs/2502.00334
- **Code:** https://github.com/YangLabHKUST/UGPhysics

**Abstract:** A large-scale benchmark with 5,520 undergraduate-level physics problems in English and Chinese, covering 13 subjects, 59 topics, 7 answer types, and 4 physics reasoning skills. All problems screened for data leakage. Includes a novel Model-Assistant Rule-based Judgment (MARJ) evaluation pipeline for physics answer assessment. Tested 31 LLMs; best overall accuracy is 49.8% (OpenAI-o1-mini). Error analysis reveals that, unlike math reasoning errors, physics errors are primarily flawed reasoning, knowledge deficiency, and wrong application of principles -- not calculation errors.

**Relevance:** Quantifies the gap between mathematical ability and physics reasoning in LLMs. The finding that even the best models achieve only ~50% on undergraduate physics, despite strong math performance, highlights the challenge of transforming mathematical formalism into physical understanding. The error taxonomy (flawed reasoning vs. knowledge gaps vs. wrong application) informs where LLM-based scientific reasoning tools need improvement.

---

### Paper 13: PhysReason -- A Comprehensive Benchmark towards Physics-Based Reasoning

- **Authors:** Xinyu Zhang, Yuxuan Dong, Yanrui Wu, Jiaxing Huang, Chengyou Jia, Basura Fernando, Mike Zheng Shou, Lingling Zhang, Jun Liu
- **Date:** February 2025 (arXiv: 2502.12054)
- **URL:** https://arxiv.org/abs/2502.12054

**Abstract:** A 1,200-problem benchmark evaluating physics-based reasoning with 25% knowledge questions and 75% complex reasoning problems across three difficulty tiers. Problems require an average of 8.1 solution steps (hard-level: 15.6 steps). Leading models (Deepseek-R1, o3-mini-high) score below 60% overall, declining from 75% on knowledge questions to 32% on difficult problems. Identifies four performance bottlenecks: applying physics theorems, understanding physical processes, performing calculations, and analyzing physical conditions.

**Relevance:** Complements UGPhysics by providing a step-level analysis of where physics reasoning breaks down. The four identified bottlenecks (theorem application, process understanding, calculation, condition analysis) map directly to stages where LLM-based scientific reasoning tools could intervene -- e.g., knowledge graphs for theorem lookup, simulation for process understanding, code execution for calculation.

---

## Gap 6: Verification Environments for Scientific Code

### Paper 14: Towards Formal Verification of LLM-Generated Code from Natural Language Prompts (Astrogator)

- **Authors:** Aaron Councilman, David Jiahao Fu, Aryan Gupta, Chengxiao Wang, David Grove, Yu-Xiong Wang, Vikram Adve
- **Date:** July 2025 (arXiv: 2507.13290)
- **URL:** https://arxiv.org/abs/2507.13290

**Abstract:** Proposes Astrogator, a system for formal verification of LLM-generated code. Introduces a Formal Query Language that represents user intent in a formally defined but natural-language-like manner, which can verify that LLM-generated code matches specifications. Includes a knowledge base to reduce required system expertise, a calculus for representing program behavior, and a symbolic interpreter for verification. Results on 21 code-generation tasks: verifies correct code in 83% of cases and identifies incorrect code in 92%.

**Relevance:** Addresses a critical gap in using LLMs for scientific code generation -- how to verify correctness. While demonstrated on Ansible, the approach (formal query language + symbolic interpreter) is generalizable to scientific domains where code must correctly implement mathematical equations. The combination of natural-language-like specifications with formal verification is particularly relevant for scientists who can express their intent but need assurance the code is correct.

---

### Paper 15: A Benchmark for Vericoding -- Formally Verified Program Synthesis

- **Authors:** Sergiu Bursuc, Theodore Ehrenborg, Shaowei Lin, Lacramioara Astefanoaei, Ionel Emilian Chiosa, Jure Kukovec, Alok Singh, Oliver Butterley, Adem Bizid, Quinn Dougherty, Miranda Zhao, Max Tan, Max Tegmark
- **Date:** September 2025 (arXiv: 2509.22908)
- **URL:** https://arxiv.org/abs/2509.22908

**Abstract:** Presents the largest benchmark for "vericoding" -- LLM generation of formally verified code from formal specifications. Contains 12,504 formal specifications across three languages: Dafny (3,029), Verus/Rust (2,334), and Lean (7,141). Off-the-shelf LLM success rates: 27% in Lean, 44% in Verus/Rust, 82% in Dafny. Natural-language descriptions do not significantly improve performance. Dafny verification improved from 68% to 96% over the past year. The term "vericoding" is coined as the opposite of "vibecoding" -- generating code with formal correctness guarantees rather than approximate, potentially buggy code.

**Relevance:** Establishes the paradigm of "vericoding" as the gold standard for LLM code generation in safety-critical and scientific contexts. The rapid improvement in Dafny verification (68% to 96% in one year) suggests that formally verified scientific code generation may become practical in the near future. For scientific computing, this could mean LLM-generated implementations of equations that come with mathematical proofs of correctness.

---

## Bonus: Cross-Cutting Paper

### Paper 16: Enhancing Mathematical Knowledge Graphs with Large Language Models

- **Authors:** (Published in MDPI, 2025)
- **Date:** Received February 2025; Published June 2025
- **URL:** https://www.mdpi.com/2673-3951/6/3/53

**Abstract:** Integrates ontology-based knowledge representation with LLMs to automate extraction, organization, and reasoning of mathematical knowledge from LaTeX documents. Key innovations: a lightweight ontology for modeling hypotheses, conclusions, and proofs; algorithms for optimizing assumptions and generating pseudo-demonstrations; an automated extraction pipeline for structured LaTeX documents from arXiv-scale repositories. Uses SPARQL for semantic querying and logical validation.

**Relevance:** Directly addresses the core research topic -- transforming scientific knowledge (in LaTeX) into structured, queryable knowledge graphs. Bridges Gap 1 (LaTeX parsing) and the broader research question of knowledge representation. The ontology-based approach provides a formal framework for organizing mathematical knowledge that could complement LLM-based explanation and code generation.

---

## Summary Table

| # | Paper | Gap Area | Year | Key Contribution |
|---|-------|----------|------|------------------|
| 1 | Nougat | LaTeX/OCR | 2023 | Visual Transformer for PDF-to-markup conversion |
| 2 | ASyMOB | LaTeX/Symbolic | 2025 | Benchmark for symbolic math with LaTeX-SymPy pipeline |
| 3 | TeXpert | LaTeX Generation | 2025 | Benchmark for LLM LaTeX code generation |
| 4 | Mathpix | LaTeX/OCR | Tool | Production equation recognition API (99% accuracy) |
| 5 | Prompt to Pipeline | Workflows + AI | 2025 | LLMs generating Galaxy/Nextflow bioinformatics pipelines |
| 6 | Jupyter AI | Notebooks + LLM | 2023-25 | Official JupyterLab generative AI extension |
| 7 | NotebookLM | Notebooks + LLM | 2024-25 | Source-grounded AI for scientific document synthesis |
| 8 | DREAMS | QM/DFT + AI | 2025 | Multi-agent LLM framework for DFT simulations |
| 9 | Aitomia | QM + AI | 2025 | Intelligent assistant for atomistic/quantum chem simulations |
| 10 | LLM for ORCA | QM + AI | 2024-25 | Finetuned LLMs for quantum chemistry input generation |
| 11 | Knowledge or Reasoning | Cross-domain | 2025 | Math reasoning does not transfer across domains |
| 12 | UGPhysics | Cross-domain | 2025 | Physics reasoning benchmark; best LLM scores 49.8% |
| 13 | PhysReason | Cross-domain | 2025 | Physics reasoning bottleneck analysis |
| 14 | Astrogator | Verification | 2025 | Formal verification of LLM-generated code |
| 15 | Vericoding Benchmark | Verification | 2025 | Largest benchmark for formally verified LLM code synthesis |
| 16 | Math Knowledge Graphs | Cross-cutting | 2025 | LLM + ontology for math knowledge extraction from LaTeX |

---

## Key Themes Across Gaps

### 1. The LaTeX Pipeline is Maturing
The path from scientific documents to machine-readable math is increasingly automated: Nougat/Mathpix handle PDF-to-LaTeX, LaTeX-to-SymPy parsers (with LLM fallback) handle symbolic conversion, and ASyMOB/TeXpert provide evaluation benchmarks. The remaining challenge is ensuring LLMs truly understand the math rather than pattern-matching.

### 2. LLMs Can Generate Scientific Workflows
Both bioinformatics pipelines (Nextflow/Galaxy) and quantum chemistry simulations (DREAMS/Aitomia/ORCA) can now be generated or orchestrated by LLMs. The key enabler is combining LLM language understanding with domain-specific tools and validation. Multi-agent architectures are the dominant pattern for complex scientific workflows.

### 3. Physics Reasoning Remains Hard
Multiple benchmarks (UGPhysics, PhysReason) show that even the best LLMs achieve only ~50% accuracy on undergraduate physics. The gap between mathematical ability and physical reasoning is qualitative, not quantitative -- physics requires situational modeling and causal grounding that math training alone does not provide. Cross-domain transfer is limited.

### 4. Verification is the Missing Piece
The emergence of "vericoding" and formal verification for LLM-generated code addresses a critical need for scientific computing. Current systems can verify 83-92% of code correctness. Rapid progress (68% to 96% in Dafny in one year) suggests formally verified scientific code generation may become practical soon. This is essential for trusting LLM-generated implementations of scientific equations.

### 5. Notebook Environments Are the Natural Interface
Jupyter AI and NotebookLM represent two complementary approaches: code-centric (execute and verify) vs. document-centric (synthesize and explain). For transforming scientific knowledge into human-understandable formats, the notebook paradigm -- combining text, equations, code, and visualizations -- is the natural medium.
