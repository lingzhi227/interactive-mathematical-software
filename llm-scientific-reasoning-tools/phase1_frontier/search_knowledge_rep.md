# Phase 1 (Frontier) Survey: LLMs for Transforming Scientific Knowledge into Human-Understandable Formats

**Search Date**: 2026-03-10
**Focus Period**: 2024--2026
**Scope**: LLM-constructed knowledge graphs for math/science, mathematical reasoning & explanation, AI-generated textbooks & interactive learning

---

## Dimension 1: Mathematical Knowledge Graphs & Ontologies

Papers on LLM-constructed knowledge graphs, mathematical ontology, and structured knowledge representation for STEM.

---

### 1.1 Mathematical Knowledge Graph-Driven Framework for Equation-Based Predictive and Reliable Additive Manufacturing

- **Authors**: Yeongbin Cha, Namjung Kim
- **Year**: 2026
- **Venue**: arXiv preprint
- **arXiv ID**: 2601.05298
- **Summary**: Proposes an integrated framework combining LLMs with a specialized mathematical knowledge graph for additive manufacturing. The system transforms unstructured scientific literature into machine-interpretable representations through formal ontology encoding of equations, variables, and their relationships. Ontology-guided LLM extraction consistently outperforms free-form LLM extraction across all metrics, with improved precision reducing spurious relations. The knowledge graph subgraphs include equation entities, variable descriptions, ontology relations, physical assumptions, validity regimes, and hierarchy-level summaries.

### 1.2 AutoMathKG: The Automated Mathematical Knowledge Graph Based on LLM and Vector Database

- **Authors**: Rong Bian, Yu Geng, Zijian Yang, Bing Cheng
- **Year**: 2025
- **Venue**: arXiv preprint
- **arXiv ID**: 2505.13406
- **Summary**: Presents a high-quality, wide-coverage mathematical knowledge graph capable of automatic updates. Structures mathematical knowledge as a directed graph of definitions, theorems, and problems with reference relationships. Integrates information from ProofWiki, textbooks, arXiv papers, and TheoremQA. Uses LLMs for in-context learning enhancement and a vector database (MathVD) with SBERT embeddings for entity similarity search and intelligent merging of new knowledge.

### 1.3 Enhancing Mathematical Knowledge Graphs with Large Language Models

- **Authors**: (Multiple authors; published in MDPI Modelling)
- **Year**: 2025
- **Venue**: Modelling (MDPI), Vol. 6(3), Article 53
- **DOI**: 10.3390/modelling6030053
- **Summary**: Integrates ontology-based knowledge representation with LLMs to automate extraction, organization, and reasoning of mathematical knowledge from LaTeX documents. Key innovations include a lightweight ontology for modeling hypotheses, conclusions, and proofs, and algorithms for optimizing assumptions and generating pseudo-demonstrations. The automated extraction pipeline processes structured LaTeX documents from arXiv-scale repositories. A web interface supports visualization and interaction with the knowledge graph.

### 1.4 LLM-Supported Formal Knowledge Representation for Enhancing Control Engineering Content with an Interactive Semantic Layer

- **Authors**: Julius Fiedler, Carsten Knoll, Klaus Robenack (TU Dresden)
- **Year**: 2025
- **Venue**: arXiv preprint
- **arXiv ID**: 2511.02759
- **Summary**: Demonstrates how LLMs can assist in transforming natural-language descriptions and mathematical definitions (as LaTeX source code) into a formalized knowledge graph using the Imperative Representation of Knowledge (PyIRK) framework. The application generates an "interactive semantic layer" enriching source documents to facilitate knowledge transfer in the control engineering domain, contributing to accessible, collaborative, and verifiable knowledge bases.

### 1.5 Large Language Models for Scholarly Ontology Generation: An Extensive Analysis in the Engineering Field

- **Authors**: Tanay Aggarwal, Angelo Salatino, Francesco Osborne, Enrico Motta
- **Year**: 2024 (submitted Dec 2024; accepted to Information Processing & Management)
- **Venue**: Information Processing & Management (camera-ready)
- **arXiv ID**: 2412.08258
- **Summary**: Evaluates 17 LLMs on automated ontology generation for scholarly knowledge using the IEEE Thesaurus benchmark. Tests four relationship types: broader, narrower, same-as, and other. Top performers include Mixtral-8x7B (F1=0.847), Dolphin-Mistral-7B (F1=0.920), and Claude 3 Sonnet (F1=0.967). Demonstrates that smaller, optimized models can match larger proprietary systems with less compute.

### 1.6 LLMs4OL: Large Language Models for Ontology Learning Challenge (2024 & 2025)

- **Authors**: Hamed Babaei Giglou, Jennifer D'Souza, et al.
- **Year**: 2024--2025
- **Venue**: ISWC 2024 (23rd) and ISWC 2025 (24th) Workshop Proceedings
- **arXiv ID**: 2409.10146 (2024 overview)
- **Summary**: A benchmark challenge series evaluating LLMs on ontology learning tasks: term typing, taxonomy discovery, and non-taxonomic relation extraction. The 2025 edition expanded to four tasks including text-to-ontology extraction. Key finding: hybrid pipelines integrating commercial LLMs with domain-tuned embeddings and fine-tuning achieved strongest performance. Prompt-based strategies with modern LLMs enable efficient, scalable, and domain-independent ontology construction.

### 1.7 The Role of Visualization in LLM-Assisted Knowledge Graph Systems: Effects on User Trust, Exploration, and Workflows

- **Authors**: Harry Li, Gabriel Appleby, Kenneth Alperin, Steven R Gomez, Ashley Suh
- **Year**: 2025
- **Venue**: arXiv preprint
- **arXiv ID**: 2505.21512
- **Summary**: Develops LinkQ, a KG exploration tool that uses LLMs to transform natural language queries into structured database queries with five visual mechanisms (state diagram, query editor with LLM explanation, entity-relation table, query structure graph, interactive graph visualization). A study with 14 KG practitioners found that users tended to overtrust outputs despite transparency-oriented visualizations, highlighting risks in LLM-assisted data analysis.

---

## Dimension 2: LLM Mathematical Reasoning & Explanation

Papers on chain-of-thought for math, knowledge decomposition, equation explanation, and pedagogical approaches.

---

### 2.1 LLM-SR: Scientific Equation Discovery via Programming with Large Language Models

- **Authors**: Parshin Shojaee, Kazem Meidani, Shashank Gupta, Amir Barati Farimani, Chandan K Reddy
- **Year**: 2024 (submitted Apr 2024; revised Mar 2025)
- **Venue**: ICLR 2025 (Oral)
- **arXiv ID**: 2404.18400
- **Summary**: Combines LLMs' scientific knowledge and code generation capabilities with evolutionary search for scientific equation discovery. Equations are represented as executable program skeletons that allow flexible hypothesis generation guided by domain-specific priors. Significantly outperforms state-of-the-art symbolic regression baselines, particularly in out-of-domain test settings. A landmark result demonstrating LLMs can bridge natural language scientific knowledge and formal mathematical expressions via code.

### 2.2 LLM-SRBench: A New Benchmark for Scientific Equation Discovery with Large Language Models

- **Authors**: Parshin Shojaee, Ngoc-Hieu Nguyen, Kazem Meidani, Amir Barati Farimani, Khoa D Doan, Chandan K Reddy
- **Year**: 2025
- **Venue**: ICML 2025 (Oral)
- **arXiv ID**: 2504.10415
- **Summary**: Introduces a comprehensive benchmark with 239 problems across four scientific domains (chemistry, biology, physics, materials science) designed to prevent trivial memorization of known equations. Features LSR-Transform (reformulating known equations into unfamiliar representations) and LSR-Synth (synthetic discovery-driven problems). The best-performing system achieves only 31.5% symbolic accuracy, revealing substantial gaps in current LLM-based equation discovery.

### 2.3 LLM-Based Scientific Equation Discovery via Physics-Informed Token-Regularized Policy Optimization (PiT-PO)

- **Authors**: Boxiao Wang, Kai Li, Tianyi Liu, Chen Li, Junzhe Wang, Yifan Zhang, Jian Cheng
- **Year**: 2026
- **Venue**: arXiv preprint
- **arXiv ID**: 2602.10576
- **Summary**: Transforms LLMs from static equation proposers into adaptive generators via reinforcement learning with physics constraints. A dual-constraint mechanism enforces both hierarchical physical validity and token-level penalties to eliminate redundant mathematical structures. Discovers novel turbulence models for fluid dynamics problems. Enables smaller language models to outperform larger proprietary systems.

### 2.4 A Self-Correcting Multi-Agent LLM Framework for Language-Based Physics Simulation and Explanation (MCP-SIM)

- **Authors**: Donggeun Park, Hyeonbin Moon, Seunghwa Ryu
- **Year**: 2025
- **Venue**: npj Artificial Intelligence (published); also Research Square preprint
- **Summary**: A multi-agent framework (MCP-SIM) that transforms underspecified natural language prompts into validated physics simulations and explanatory reports. Six specialized agents handle input clarification, code generation, simulation execution, error diagnosis, input rewriting, and mechanical insight generation through iterative plan-act-reflect-revise cycles. Achieves 100% success on a twelve-task benchmark spanning elasticity, heat transfer, fluid dynamics, and multiphysics. Generates multilingual educational reports explaining governing equations, boundary conditions, and solver strategies.

### 2.5 A Survey on Large Language Models for Mathematical Reasoning

- **Authors**: Peng-Yuan Wang, Tian-Shuo Liu, Chenyang Wang, et al.
- **Year**: 2025
- **Venue**: arXiv preprint
- **arXiv ID**: 2506.08446
- **Summary**: Comprehensive survey examining how LLMs develop mathematical reasoning through two cognitive phases: comprehension (pretraining strategies) and answer generation (from direct prediction to step-by-step CoT reasoning). Covers enhancement techniques from training-free prompting to supervised fine-tuning and RL. Identifies persistent gaps in model capacity, computational efficiency, and generalization, with future directions including formal reasoning frameworks and meta-generalization.

### 2.6 Towards Reasoning Era: A Survey of Long Chain-of-Thought for Reasoning Large Language Models

- **Authors**: Qiguang Chen, Libo Qin, Jinhao Liu, et al.
- **Year**: 2025
- **Venue**: arXiv preprint
- **arXiv ID**: 2503.09567
- **Summary**: Comprehensive survey on extended reasoning in LLMs (OpenAI-O1, DeepSeek-R1). Identifies three essential long-CoT characteristics: deep reasoning, extensive exploration, and feasible reflection. Investigates the "overthinking" phenomenon where test-time scaling can harm performance on simpler tasks. Proposes that an optimal scaled length distribution differs across domains.

### 2.7 Formal Mathematical Reasoning: A New Frontier in AI

- **Authors**: Kaiyu Yang, Gabriel Poesia, Jingxuan He, Wenda Li, Kristin Lauter, Swarat Chaudhuri, Dawn Song
- **Year**: 2024
- **Venue**: arXiv preprint
- **arXiv ID**: 2412.16075
- **Summary**: Position paper advocating for formal mathematical reasoning grounded in proof assistants as a complementary pathway to text-based approaches. Highlights that formal systems can verify correctness and provide automatic feedback, with emerging applications extending to verifiable code and hardware design generation. Synthesizes existing developments, identifies open problems, and establishes milestones for measuring progress.

### 2.8 Improving Chain-of-Thought Reasoning via Quasi-Symbolic Abstractions (QuaSAR)

- **Authors**: Leonardo Ranaldi, Marco Valentino, Andre Freitas
- **Year**: 2025
- **Venue**: ACL 2025
- **arXiv ID**: 2502.12616
- **Summary**: Introduces QuaSAR, a framework that enhances CoT reasoning by combining symbolic and natural language elements without requiring full formal translation. The approach operates in four steps: abstraction (symbolic predicates/variables), formalisation (mixed symbolic-NL reformulation), explanation (quasi-symbolic reasoning chains), and answering. Improves CoT accuracy by up to 8% on adversarial benchmarks (MMLU-Redux, GSM-Symbolic) while maintaining human readability.

### 2.9 Hierarchical Deconstruction of LLM Reasoning: A Graph-Based Framework for Analyzing Knowledge Utilization

- **Authors**: Miyoung Ko, Sue Hyun Park, Joonsuk Park, Minjoon Seo
- **Year**: 2024
- **Venue**: EMNLP 2024
- **arXiv ID**: 2406.19502
- **Summary**: Deconstructs complex questions into a graph where each node represents a question with predecessors of required background knowledge at three depths: (i) recalling conceptual knowledge, (ii) applying procedural knowledge, and (iii) analyzing strategic knowledge. Introduces the DEPTHQA dataset and quantifies "forward discrepancy" (failure on complex despite solving simpler sub-problems) and "backward discrepancy" (solving complex but failing on simpler). Demonstrates that guiding models from simpler to complex questions via multi-turn interactions improves performance.

### 2.10 Exploring the Limits of Fine-grained LLM-based Physics Inference via Premise Removal Interventions

- **Authors**: Jordan Meadows, Tamsin Emily James, Andre Freitas
- **Year**: 2024
- **Venue**: Findings of EMNLP 2024
- **arXiv ID**: 2404.18384
- **Summary**: Presents a manually curated dataset of 1,200 derivation steps over 218 graduate-level physics derivations spanning electromagnetism, classical, quantum, and statistical mechanics. Expands individual derivation steps into finer-grained steps to improve explainability. Finds through premise removal interventions that LLMs' mathematical reasoning is not physics-informed in this setting -- physical context is predominantly ignored in favor of reverse-engineering solutions.

### 2.11 DotaMath: Decomposition of Thought with Code Assistance and Self-correction for Mathematical Reasoning

- **Authors**: Chengpeng Li, Guanting Dong, Mingfeng Xue, Ru Peng, Xiang Wang, Dayiheng Liu
- **Year**: 2024
- **Venue**: arXiv preprint (work in progress)
- **arXiv ID**: 2407.04078
- **Summary**: Decomposes complex mathematical problems into manageable logical sub-tasks, leveraging code execution for each subtask with feedback from code interpreters and self-reflection for error correction. Creates the DotaMathQA dataset (574K query-response pairs) via interactive tool-use trajectory annotation on GSM8K/MATH. DotaMath-deepseek-7B achieves 64.8% on MATH and 86.7% on GSM8K.

### 2.12 The Open Proof Corpus: A Large-Scale Study of LLM-Generated Mathematical Proofs

- **Authors**: Jasper Dekoninck, Ivo Petrov, Kristian Minchev, Mislav Balunovic, Martin Vechev, et al.
- **Year**: 2025
- **Venue**: arXiv preprint
- **arXiv ID**: 2506.21621
- **Summary**: Introduces a dataset of over 5,000 human-evaluated proofs produced by state-of-the-art LLMs, including correct solutions to USAMO and IMO problems. Investigates the gap between natural language and formal proofs, differences between final-answer correctness and complete proof validity, and how selecting multiple attempts improves quality. Fine-tuning an 8B model on the dataset achieves performance comparable to Gemini-2.5-Pro for evaluating proof correctness.

### 2.13 FEM-Bench: A Structured Scientific Reasoning Benchmark for Evaluating Code-Generating LLMs

- **Authors**: Saeed Mohammadzadeh, Erfan Hamdi, Joel Shor, Emma Lejeune
- **Year**: 2025
- **Venue**: arXiv preprint
- **arXiv ID**: 2512.20732
- **Summary**: A computational mechanics benchmark testing LLMs on finite element method (FEM) code generation with 33 graduate-level tasks involving geometry, spatial relationships, and material behavior with strict physical/numerical constraints. Gemini 3 Pro achieved best results (30/33 tasks at least once), while GPT-5 led in unit test generation (73.8% average). Reveals substantial variability across models.

### 2.14 ChronoLLM: Customizing Language Models for Physics-Based Simulation Code Generation

- **Authors**: Jingquan Wang, Andrew Negrut, Harry Zhang, et al.
- **Year**: 2025
- **Venue**: arXiv preprint
- **arXiv ID**: 2508.13975
- **Summary**: Customizes multiple LLM types to generate code scripts for PyChrono, an open-source multibody dynamics engine. Generated scripts function as "strong starting points" for simulations ranging from basic pendulum models to complex vehicle-terrain interactions. The customized LLM also answers API-specific questions and recommends modeling strategies.

---

## Dimension 3: AI-Generated Textbooks & Interactive Learning

Papers on Google LearnLM/Learn Your Way, AI-augmented textbooks, personalized STEM education, and mind maps from AI.

---

### 3.1 Towards an AI-Augmented Textbook (Google LearnLM / Learn Your Way)

- **Authors**: LearnLM Team at Google (Gal Elidan, Yael Haramaty, Yossi Matias, and 30+ co-authors)
- **Year**: 2025
- **Venue**: arXiv preprint
- **arXiv ID**: 2509.13348
- **Summary**: Proposes using generative AI to transform textbooks through multiple representations and personalization while preserving content fidelity. The Learn Your Way system personalizes by grade level and interests, generating immersive text, narrated slides, audio lessons, mind maps, and quizzes. Built on LearnLM models integrated into Gemini 2.5 Pro. Expert evaluation: average 0.85+ across pedagogical criteria. Randomized controlled trial (60 students, ages 15-18): 9 percentage points higher on immediate assessment, 11% higher on 3-5 day retention test. 100% reported increased comfort; 93% would use again.

### 3.2 From Solver to Tutor: Evaluating the Pedagogical Intelligence of LLMs with KMP-Bench

- **Authors**: Weikang Shi, Houxing Ren, Junting Pan, Aojun Zhou, Ke Wang, et al.
- **Year**: 2026
- **Venue**: arXiv preprint
- **arXiv ID**: 2603.02775
- **Summary**: Introduces KMP-Bench for evaluating LLMs as K-8 math tutors with two modules: KMP-Dialogue (holistic teaching against 6 pedagogical principles) and KMP-Skills (collaborative problem-solving, error correction, problem generation). Includes KMP-Pile, a 150K-item dialogue dataset. Leading LLMs perform well on tasks with objective answers but struggle with nuanced pedagogical application. Models trained on pedagogically-informed data show substantial gains.

### 3.3 A Theory of Adaptive Scaffolding for LLM-Based Pedagogical Agents (Inquizzitor)

- **Authors**: Clayton Cohn, Surya Rayala, Namrata Srivastava, et al.
- **Year**: 2025
- **Venue**: AAAI 2026 (Main Technical Track)
- **arXiv ID**: 2508.01503
- **Summary**: Combines Evidence-Centered Design with Social Cognitive Theory for adaptive scaffolding in STEM+C tutoring. Introduces Inquizzitor, an agent for formative assessment merging human-AI hybrid intelligence with feedback grounded in cognitive science. Demonstrates that LLMs can move beyond general-purpose tools toward principled educational applications integrating rigorous pedagogical theory.

### 3.4 Training LLM-based Tutors to Improve Student Learning Outcomes in Dialogues

- **Authors**: Alexander Scarlatos, Naiming Liu, Jaewook Lee, Richard Baraniuk, Andrew Lan
- **Year**: 2025
- **Venue**: AIED 2025 (Springer LNCS vol. 15877)
- **arXiv ID**: 2503.06424
- **Summary**: Optimizes LLM tutors for actual student learning outcomes rather than just pedagogical guidelines. Generates candidate tutor utterances, scores them using an LLM-based student model and GPT-4o pedagogical rubrics, and trains Llama 3.1 8B via direct preference optimization. Tutor utterances lead to significantly higher correct student responses while maintaining GPT-4o-level pedagogical quality.

### 3.5 Rewarding How Models Think Pedagogically: Integrating Pedagogical Reasoning and Thinking Rewards for LLMs in Education (PedagogicalRL-Thinking)

- **Authors**: Unggi Lee, Jiyeong Bae, Jaehyeon Park, et al.
- **Year**: 2026
- **Venue**: arXiv preprint
- **arXiv ID**: 2601.14560
- **Summary**: Introduces PedagogicalRL-Thinking, extending pedagogical alignment to reasoning LLMs through: (1) Pedagogical Reasoning Prompting using educational theory rather than generic instructions, and (2) Thinking Reward that evaluates pedagogical quality of reasoning traces. Domain-specific, theory-grounded prompting outperforms generic prompting. Models trained on math tutoring show improved performance on unseen educational benchmarks.

### 3.6 Generative Large Language Models for Knowledge Representation: A Systematic Review of Concept Map Generation

- **Authors**: Xiaoming Zhai
- **Year**: 2025
- **Venue**: arXiv preprint
- **arXiv ID**: 2509.14554
- **Summary**: Systematic review of 28 studies on LLM-automated concept map creation for education. Identifies six methodological categories: human-in-the-loop, weakly supervised, fine-tuned domain-specific, prompt-engineered pretrained, hybrid knowledge-base integration, and modular symbolic-statistical frameworks. Finds that LLM-generated maps hold promise for scalable, adaptive knowledge visualization but face challenges in validity, interpretability, multilingual support, and classroom deployment.

### 3.7 AI-Driven Interactive Hierarchical Concept Maps for Digital Learning Environments and Intelligent Textbooks

- **Authors**: Sergiy Tytenko
- **Year**: 2025
- **Venue**: iTextbooks 2025 (6th Workshop on Intelligent Textbooks), CEUR-WS Vol-4010, pp. 3-16
- **Summary**: Demonstrates concept map construction from e-books using LLMs through section segmentation, key concept extraction, and relationship identification. Evaluated using GPT-4o on Python programming lectures with strong performance. Addresses challenges through hierarchical drill-down navigation, embedded pedagogical content, and validation based on real student feedback.

### 3.8 Mindalogue: LLM-Powered Nonlinear Interaction for Effective Learning and Task Exploration

- **Authors**: Rui Zhang, Ziyao Zhang, Fengliang Zhu, Jiajie Zhou, Anyi Rao
- **Year**: 2024
- **Venue**: CHI 2025 (ACM Conference on Human Factors in Computing Systems)
- **arXiv ID**: 2410.10570
- **Summary**: Addresses limitations of linear interaction with generative AI by introducing a "nodes + canvas" nonlinear interface. Features single-node explain/examples/explore operations enabling multi-level AI exploration. Evaluation with 16 participants showed significantly reduced task steps and improved comprehension of complex information compared to traditional chat-based interfaces.

### 3.9 MindMap: Knowledge Graph Prompting Sparks Graph of Thoughts in Large Language Models

- **Authors**: Wen et al.
- **Year**: 2024
- **Venue**: ACL 2024
- **arXiv ID**: 2308.09729
- **Summary**: A plug-and-play prompting approach that enables LLMs to comprehend knowledge graph inputs and build their own "mind map" supporting evidence-grounded generation. Leverages KGs to enhance LLMs' inference and transparency, revealing reasoning pathways based on ontology of knowledge. Addresses LLM limitations in incorporating new knowledge, reducing hallucinations, and improving decision-making transparency.

---

## Cross-Cutting Themes & Key Takeaways

1. **Knowledge graphs as the bridge**: Multiple papers (2601.05298, 2505.13406, MDPI 2025) demonstrate that ontology-guided LLM extraction of mathematical knowledge significantly outperforms unconstrained extraction, with knowledge graphs serving as the critical structured intermediary between unstructured scientific text and machine-interpretable formats.

2. **Code as the universal representation**: LLM-SR (ICLR 2025 Oral), FEM-Bench, and ChronoLLM show that executable code is emerging as the preferred output format for translating scientific equations into actionable knowledge -- equations as programs allow both human understanding and machine verification.

3. **Multi-agent systems for scientific explanation**: MCP-SIM demonstrates that orchestrating multiple specialized LLM agents can transform natural language prompts into validated simulations with accompanying educational explanations, achieving 100% success across diverse physics domains.

4. **Pedagogical alignment is distinct from reasoning ability**: KMP-Bench (2603.02775) and PedagogicalRL-Thinking (2601.14560) reveal that LLMs strong at solving problems are not automatically good at teaching -- specialized training with pedagogical theory is required.

5. **Nonlinear interfaces for knowledge exploration**: Mindalogue (CHI 2025), concept maps (2509.14554), and Google's Learn Your Way (2509.13348) all converge on the same insight: linear text is insufficient for knowledge transfer, and structured visual representations (trees, maps, canvases) substantially improve learning outcomes.

6. **The memorization vs. discovery gap**: LLM-SRBench (ICML 2025 Oral) shows that current LLMs achieve only 31.5% symbolic accuracy on equations that resist memorization, establishing a sobering baseline for genuine scientific equation discovery.

---

## Summary Statistics

| Dimension | Papers Found | Key Venues |
|-----------|-------------|------------|
| Math KG & Ontologies | 7 | IP&M, ISWC, MDPI Modelling, arXiv |
| Math Reasoning & Explanation | 14 | ICLR 2025 (Oral), ICML 2025 (Oral), ACL 2025, EMNLP 2024, npj AI, arXiv |
| AI Textbooks & Interactive Learning | 9 | CHI 2025, AAAI 2026, AIED 2025, ACL 2024, iTextbooks 2025, arXiv |
| **Total** | **30** | |
