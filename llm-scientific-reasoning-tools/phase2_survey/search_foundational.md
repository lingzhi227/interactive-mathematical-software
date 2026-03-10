# Phase 2 Survey: Foundational and Earlier Works (2007-2023)

## Search Strategy
Searched for foundational, seminal, and earlier works across six categories that provide essential context for the research area of using LLMs to transform scientific knowledge into human-understandable formats. Focus on works from 2007-2023 that predate the Phase 1 papers (mostly 2024-2026).

**Date of search**: 2026-03-10
**Papers found**: 25

---

## Category 1: LLM for Math Reasoning (Foundational 2019-2023)

### Paper 1
- **title**: Chain-of-Thought Prompting Elicits Reasoning in Large Language Models
- **authors**: Wei, Jason; Wang, Xuezhi; Schuurmans, Dale; Bosma, Maarten; Ichter, Brian; Xia, Fei; Chi, Ed; Le, Quoc; Zhou, Denny
- **first_author**: Wei, Jason
- **year**: 2022
- **venue**: NeurIPS 2022
- **arxiv_id**: 2201.11903
- **peer_reviewed**: true
- **summary**: Introduces chain-of-thought (CoT) prompting, demonstrating that providing a few exemplars with intermediate reasoning steps significantly improves LLM performance on arithmetic, commonsense, and symbolic reasoning tasks. Shows that such reasoning abilities emerge naturally in sufficiently large language models. This is the foundational work that enabled all subsequent structured reasoning approaches for scientific knowledge transformation.

### Paper 2
- **title**: Self-Consistency Improves Chain of Thought Reasoning in Language Models
- **authors**: Wang, Xuezhi; Wei, Jason; Schuurmans, Dale; Le, Quoc; Chi, Ed; Narang, Sharan; Chowdhery, Aakanksha; Zhou, Denny
- **first_author**: Wang, Xuezhi
- **year**: 2023
- **venue**: ICLR 2023
- **arxiv_id**: 2203.11171
- **peer_reviewed**: true
- **summary**: Proposes self-consistency decoding, which samples diverse reasoning paths and selects the most consistent answer by marginalizing out sampled reasoning paths. Achieves significant improvements on GSM8K (+17.9%), SVAMP (+11.0%), and AQuA (+12.2%). Establishes that multiple reasoning paths can be aggregated for more reliable mathematical reasoning.

### Paper 3
- **title**: PAL: Program-aided Language Models
- **authors**: Gao, Luyu; Madaan, Aman; Zhou, Shuyan; Alon, Uri; Liu, Pengfei; Yang, Yiming; Callan, Jamie; Neubig, Graham
- **first_author**: Gao, Luyu
- **year**: 2023
- **venue**: ICML 2023 (PMLR 202)
- **arxiv_id**: 2211.10435
- **peer_reviewed**: true
- **summary**: Presents Program-Aided Language Models (PAL), where LLMs generate programs as intermediate reasoning steps and offload computation to a Python interpreter. Demonstrates that decomposing natural language problems into runnable code steps is more reliable than having LLMs perform arithmetic directly. Evaluated across 12 reasoning tasks from BIG-Bench Hard and other benchmarks. Key precursor to using code as a scientific knowledge representation format.

### Paper 4
- **title**: Program of Thoughts Prompting: Disentangling Computation from Reasoning for Numerical Reasoning Tasks
- **authors**: Chen, Wenhu; Ma, Xueguang; Wang, Xinyi; Cohen, William W.
- **first_author**: Chen, Wenhu
- **year**: 2023
- **venue**: TMLR 2023 (Transactions on Machine Learning Research)
- **arxiv_id**: 2211.12588
- **peer_reviewed**: true
- **summary**: Proposes Program of Thoughts (PoT) prompting, where LLMs express reasoning as Python programs executed by an interpreter, disentangling computation from reasoning. Shows ~12% average performance gain over CoT across math word problem and financial QA datasets. Demonstrates that code-as-reasoning is superior for numerical tasks, directly relevant to translating scientific equations into executable representations.

### Paper 5
- **title**: Solving Quantitative Reasoning Problems with Language Models (Minerva)
- **authors**: Lewkowycz, Aitor; Andreassen, Anders; Dohan, David; Dyer, Ethan; Michalewski, Henryk; Ramasesh, Vinay; Slone, Ambrose; Anil, Cem; Schlag, Imanol; Gutman-Solo, Theo; Wu, Yuhuai; Neyshabur, Behnam; Gur-Ari, Guy; Misra, Vedant
- **first_author**: Lewkowycz, Aitor
- **year**: 2022
- **venue**: NeurIPS 2022
- **arxiv_id**: 2206.14858
- **peer_reviewed**: true
- **summary**: Introduces Minerva, a PaLM-based model further trained on 118GB of scientific papers and math-heavy web pages. Achieves state-of-the-art on STEM reasoning without external tools, solving ~30% of undergraduate-level problems in physics, biology, chemistry, and economics. Demonstrates that training on LaTeX/MathJax mathematical notation enables models to converse using standard mathematical notation, a key capability for scientific knowledge transformation.

### Paper 6
- **title**: Llemma: An Open Language Model For Mathematics
- **authors**: Azerbayev, Zhangir; Schoelkopf, Hailey; Paster, Keiran; Dos Santos, Marco; McAleer, Stephen; Jiang, Albert Q.; Deng, Jia; Biderman, Stella; Welleck, Sean
- **first_author**: Azerbayev, Zhangir
- **year**: 2023
- **venue**: ICLR 2024
- **arxiv_id**: 2310.10631
- **peer_reviewed**: true
- **summary**: Continues pretraining Code Llama on Proof-Pile-2 (scientific papers, math web data, mathematical code) to create Llemma, an open 7B/34B parameter math-focused LLM. Outperforms Minerva at equivalent scale on MATH benchmark and demonstrates tool use and formal theorem proving without finetuning. First open-weight model competitive with proprietary systems for mathematical reasoning, enabling reproducible research on scientific knowledge transformation.

### Paper 7
- **title**: Tree of Thoughts: Deliberate Problem Solving with Large Language Models
- **authors**: Yao, Shunyu; Yu, Dian; Zhao, Jeffrey; Shafran, Izhak; Griffiths, Thomas L.; Cao, Yuan; Narasimhan, Karthik
- **first_author**: Yao, Shunyu
- **year**: 2023
- **venue**: NeurIPS 2023
- **arxiv_id**: 2305.10601
- **peer_reviewed**: true
- **summary**: Introduces Tree of Thoughts (ToT), generalizing CoT to explore multiple reasoning paths with self-evaluation, lookahead, and backtracking. Improves Game of 24 success from 4% (CoT) to 74%. Provides the foundational framework for tree-structured knowledge representation of reasoning, directly relevant to building knowledge trees from scientific theories.

### Paper 8
- **title**: Least-to-Most Prompting Enables Complex Reasoning in Large Language Models
- **authors**: Zhou, Denny; Scharli, Nathanael; Hou, Le; Wei, Jason; Scales, Nathan; Wang, Xuezhi; Schuurmans, Dale; Cui, Claire; Bousquet, Olivier; Le, Quoc; Chi, Ed
- **first_author**: Zhou, Denny
- **year**: 2023
- **venue**: ICLR 2023
- **arxiv_id**: 2205.10625
- **peer_reviewed**: true
- **summary**: Proposes least-to-most prompting, which decomposes complex problems into simpler subproblems solved sequentially. Achieves 99% on SCAN compositional generalization with just 14 exemplars (vs. 16% with CoT). Establishes the principle of hierarchical decomposition for problem solving, directly applicable to decomposing complex scientific equations into understandable sub-components.

---

## Category 2: Mathematical Reasoning Benchmarks and Datasets

### Paper 9
- **title**: Measuring Mathematical Problem Solving With the MATH Dataset
- **authors**: Hendrycks, Dan; Burns, Collin; Kadavath, Saurav; Arora, Akul; Basart, Steven; Tang, Eric; Song, Dawn; Steinhardt, Jacob
- **first_author**: Hendrycks, Dan
- **year**: 2021
- **venue**: NeurIPS 2021 (Datasets and Benchmarks)
- **arxiv_id**: 2103.03874
- **peer_reviewed**: true
- **summary**: Introduces MATH, a dataset of 12,500 challenging competition mathematics problems with step-by-step solutions in LaTeX. Covers algebra, calculus, geometry, number theory, and combinatorics. Large transformer models achieved only 3-7% accuracy at release, establishing a critical benchmark for measuring progress in mathematical reasoning that has driven subsequent research.

### Paper 10
- **title**: Training Verifiers to Solve Math Word Problems (GSM8K)
- **authors**: Cobbe, Karl; Kosaraju, Vineet; Bavarian, Mohammad; Chen, Mark; Jun, Heewoo; Kaiser, Lukasz; Plappert, Matthias; Tworek, Jerry; Hilton, Jacob; Nakano, Reiichiro; Hesse, Christopher; Schulman, John
- **first_author**: Cobbe, Karl
- **year**: 2021
- **venue**: arXiv preprint
- **arxiv_id**: 2110.14168
- **peer_reviewed**: false
- **summary**: Introduces GSM8K, a dataset of 8.5K linguistically diverse grade school math word problems requiring multi-step reasoning. Proposes training verifiers to judge solution correctness and selecting the highest-ranked among multiple candidate solutions. Demonstrates that verification scales more effectively than finetuning, establishing a key paradigm for reliable mathematical reasoning.

### Paper 11
- **title**: Analysing Mathematical Reasoning Abilities of Neural Models
- **authors**: Saxton, David; Grefenstette, Edward; Hill, Felix; Kohli, Pushmeet
- **first_author**: Saxton, David
- **year**: 2019
- **venue**: ICLR 2019
- **arxiv_id**: 1904.01557
- **peer_reviewed**: true
- **summary**: Early DeepMind work presenting a mathematics dataset covering arithmetic, algebra, probability, and calculus with free-form textual input/output. Provides interpolation and extrapolation tests to measure generalization along various difficulty axes. Established key methodology for evaluating neural models' mathematical reasoning, showing models achieve moderate generalization on algebraic tasks.

---

## Category 3: Autoformalization and Formal Mathematics

### Paper 12
- **title**: A Promising Path Towards Autoformalization and General Artificial Intelligence
- **authors**: Szegedy, Christian
- **first_author**: Szegedy, Christian
- **year**: 2020
- **venue**: CICM 2020 (Conference on Intelligent Computer Mathematics), Springer LNCS
- **arxiv_id**: N/A
- **peer_reviewed**: true
- **summary**: Seminal position paper arguing that autoformalization -- automatically translating natural language mathematics into machine-verifiable formal proofs -- is a promising path toward general AI. Outlines a realistic roadmap for bootstrapping from unlabeled data with minimum human interaction. Surveys results supporting feasibility, influencing the entire field of LLM-assisted formalization.

### Paper 13
- **title**: Autoformalization with Large Language Models
- **authors**: Wu, Yuhuai; Jiang, Albert Q.; Li, Wenda; Rabe, Markus; Staats, Charles; Jamnik, Mateja; Szegedy, Christian
- **first_author**: Wu, Yuhuai
- **year**: 2022
- **venue**: NeurIPS 2022
- **arxiv_id**: 2205.12615
- **peer_reviewed**: true
- **summary**: Demonstrates that LLMs can correctly translate 25.3% of mathematical competition problems to formal specifications in Isabelle/HOL. Improves neural theorem prover performance on MiniF2F benchmark from 29.6% to 35.2% by training on autoformalized theorems. First large-scale demonstration that LLMs can bridge informal and formal mathematical representations.

### Paper 14
- **title**: Draft, Sketch, and Prove: Guiding Formal Theorem Provers with Informal Proofs
- **authors**: Jiang, Albert Q.; Welleck, Sean; Zhou, Jin Peng; Li, Wenda; Liu, Jiacheng; Jamnik, Mateja; Lacroix, Timothee; Wu, Yuhuai; Lample, Guillaume
- **first_author**: Jiang, Albert Q.
- **year**: 2023
- **venue**: ICLR 2023 (Oral)
- **arxiv_id**: 2210.12283
- **peer_reviewed**: true
- **summary**: Introduces DSP (Draft, Sketch, Prove), mapping informal proofs to formal proof sketches and using them to guide automated provers. Demonstrates that LLMs can produce well-structured formal sketches following the same reasoning steps as informal proofs. First work to leverage available informal proofs for formal theorem proving, bridging human-readable and machine-verifiable mathematics.

### Paper 15
- **title**: Generative Language Modeling for Automated Theorem Proving (GPT-f)
- **authors**: Polu, Stanislas; Sutskever, Ilya
- **first_author**: Polu, Stanislas
- **year**: 2020
- **venue**: arXiv preprint
- **arxiv_id**: 2009.03393
- **peer_reviewed**: false
- **summary**: Introduces GPT-f, applying transformer language models to automated theorem proving in Metamath. First deep-learning system to contribute proofs accepted into a formal mathematics library (Metamath). Demonstrates that generative language models can produce original mathematical terms, addressing a key limitation of traditional automated provers.

### Paper 16
- **title**: NaturalProofs: Mathematical Theorem Proving in Natural Language
- **authors**: Welleck, Sean; Liu, Jiacheng; Le Bras, Ronan; Hajishirzi, Hannaneh; Choi, Yejin; Cho, Kyunghyun
- **first_author**: Welleck, Sean
- **year**: 2021
- **venue**: NeurIPS 2021 (Datasets and Benchmarks)
- **arxiv_id**: 2104.01112
- **peer_reviewed**: true
- **summary**: Develops a multi-domain corpus of mathematical statements and proofs written in natural mathematical language, unifying broad and deep coverage sources. Enables evaluation of both in-distribution and zero-shot generalization for mathematical reasoning. Provides foundational infrastructure for studying how models can work with human-readable mathematical proofs.

---

## Category 4: Knowledge Representation for Science

### Paper 17
- **title**: Knowledge Graphs
- **authors**: Hogan, Aidan; Blomqvist, Eva; Cochez, Michael; D'amato, Claudia; De Melo, Gerard; Gutierrez, Claudio; et al.
- **first_author**: Hogan, Aidan
- **year**: 2021
- **venue**: ACM Computing Surveys, Volume 54, Issue 4
- **arxiv_id**: 2003.02320
- **peer_reviewed**: true
- **summary**: Comprehensive introduction to knowledge graphs covering graph-based data models, query/validation languages, deductive and inductive knowledge representation techniques, and applications. Provides the canonical reference for knowledge graph concepts including ontology engineering, knowledge extraction, and knowledge fusion. Essential context for understanding how LLMs can generate and populate scientific knowledge graphs.

### Paper 18
- **title**: A Survey on Knowledge Graphs: Representation, Acquisition, and Applications
- **authors**: Ji, Shaoxiong; Pan, Shirui; Cambria, Erik; Marttinen, Pekka; Yu, Philip S.
- **first_author**: Ji, Shaoxiong
- **year**: 2022
- **venue**: IEEE Transactions on Neural Networks and Learning Systems, Vol. 33, No. 2
- **arxiv_id**: 2002.00388
- **peer_reviewed**: true
- **summary**: Surveys knowledge graph embedding, completion, and applications across representation spaces, scoring functions, encoding models, and auxiliary information. Reviews path inference and logical rule reasoning methods for knowledge acquisition. Provides the technical foundation for neural approaches to knowledge graph construction that LLM-based systems build upon.

### Paper 19
- **title**: Ontologies and Languages for Representing Mathematical Knowledge on the Semantic Web
- **authors**: Lange, Christoph
- **first_author**: Lange, Christoph
- **year**: 2013
- **venue**: Semantic Web Journal, Vol. 4, No. 2, pp. 119-158
- **arxiv_id**: N/A
- **peer_reviewed**: true
- **summary**: Reviews ontologies for mathematical problems, proofs, and scientific publications, plus mathematical metadata vocabularies. Shows that MathML and OpenMath (XML-based mathematical exchange languages) can be integrated with RDF for the Web of Data. Provides a roadmap for mathematical knowledge datasets and interlinking -- the pre-LLM approach to machine-readable mathematical knowledge that modern LLM systems aim to supersede or complement.

---

## Category 5: Interactive Scientific Computing Environments

### Paper 20
- **title**: IPython: A System for Interactive Scientific Computing
- **authors**: Perez, Fernando; Granger, Brian E.
- **first_author**: Perez, Fernando
- **year**: 2007
- **venue**: Computing in Science & Engineering, Vol. 9, No. 3, pp. 21-29 (IEEE)
- **arxiv_id**: N/A
- **peer_reviewed**: true
- **summary**: Introduces IPython, the interactive computing environment that later evolved into Jupyter. Provides enhanced interactive features including data visualization support and distributed/parallel computation facilities. The foundational system that established the interactive scientific computing paradigm that modern LLM-assisted tools extend (e.g., AI-powered notebook assistants, code generation in computational notebooks).

### Paper 21
- **title**: Jupyter Notebooks -- A Publishing Format for Reproducible Computational Workflows
- **authors**: Kluyver, Thomas; Ragan-Kelley, Benjamin; Perez, Fernando; Granger, Brian; Bussonnier, Matthias; Frederic, Jonathan; et al.
- **first_author**: Kluyver, Thomas
- **year**: 2016
- **venue**: Positioning and Power in Academic Publishing: Players, Agents and Agendas (IOS Press)
- **arxiv_id**: N/A
- **peer_reviewed**: true
- **summary**: Presents Jupyter Notebooks as a realization of the "computational narrative" concept -- documents embedding code, results, and explanations in readable and executable form. Describes the web-based architecture designed for extensibility, language independence, and open document format. Established the dominant environment for scientific computing that LLM code generation systems now target as their primary output format.

---

## Category 6: Code Generation for Science

### Paper 22
- **title**: Evaluating Large Language Models Trained on Code (Codex)
- **authors**: Chen, Mark; Tworek, Jerry; Jun, Heewoo; Yuan, Qiming; Pinto, Henrique Ponde de Oliveira; Kaplan, Jared; et al.
- **first_author**: Chen, Mark
- **year**: 2021
- **venue**: arXiv preprint
- **arxiv_id**: 2107.03374
- **peer_reviewed**: false
- **summary**: Introduces Codex, a GPT model fine-tuned on GitHub code that powers GitHub Copilot. Solves 28.8% of HumanEval problems (vs. 0% for GPT-3), and 70.2% with 100 samples per problem. Introduces the HumanEval benchmark for functional correctness evaluation. The foundational code generation model that enabled all subsequent work on LLM-assisted scientific code generation.

### Paper 23
- **title**: Competition-Level Code Generation with AlphaCode
- **authors**: Li, Yujia; Choi, David; Chung, Junyoung; Kushman, Nate; Schrittwieser, Julian; et al.
- **first_author**: Li, Yujia
- **year**: 2022
- **venue**: Science, Vol. 378, Issue 6624
- **arxiv_id**: 2203.07814
- **peer_reviewed**: true
- **summary**: DeepMind's system achieving top 54.3% ranking on Codeforces competitive programming. Uses encoder-decoder transformer to generate millions of diverse programs, then filters and clusters to 10 submissions. First AI system to perform competitively in programming competitions, demonstrating that LLMs can handle complex algorithmic reasoning relevant to scientific computing.

### Paper 24
- **title**: StarCoder: May the Source Be with You!
- **authors**: Li, Raymond; et al. (67 authors, BigCode community)
- **first_author**: Li, Raymond
- **year**: 2023
- **venue**: TMLR (Transactions on Machine Learning Research)
- **arxiv_id**: 2305.06161
- **peer_reviewed**: true
- **summary**: Introduces StarCoder/StarCoderBase, 15.5B parameter open-access code LLMs trained on 1 trillion tokens from The Stack (80+ programming languages) with 8K context and infilling capabilities. Outperforms every open code LLM supporting multiple languages and matches OpenAI code-cushman-001. Established the open-source foundation for code generation tools used in scientific computing applications.

### Paper 25
- **title**: A Neural Network Solves, Explains, and Generates University Math Problems by Program Synthesis and Few-Shot Learning at Human Level
- **authors**: Drori, Iddo; Zhang, Sarah; Shuttleworth, Reece; Tang, Leonard; Lu, Albert; Ke, Elizabeth; et al.
- **first_author**: Drori, Iddo
- **year**: 2022
- **venue**: PNAS (Proceedings of the National Academy of Sciences)
- **arxiv_id**: 2112.15594
- **peer_reviewed**: true
- **summary**: Demonstrates that Codex can solve MIT and Columbia university-level math course problems at 81% accuracy via program synthesis and few-shot learning, covering calculus, differential equations, linear algebra, probability, and computational linear algebra. The model not only solves but also explains and generates new problems. First demonstration of LLMs solving real university-level STEM courses through code generation, directly demonstrating the transformation of mathematical knowledge into executable programs.

---

## Summary Statistics
- **Total papers found**: 25
- **Peer-reviewed**: 21 (84%)
- **Year distribution**: 2007 (1), 2013 (1), 2016 (1), 2019 (1), 2020 (2), 2021 (4), 2022 (7), 2023 (8)
- **Venue breakdown**: NeurIPS (6), ICLR (4), ICML (1), TMLR (2), ACL (0), Science (1), PNAS (1), IEEE (2), ACM (1), Springer (1), IOS Press (1), arXiv-only (3), other (2)

## Category Coverage
1. **LLM for Math Reasoning (foundational)**: 8 papers -- CoT, Self-Consistency, PAL, PoT, Minerva, Llemma, ToT, Least-to-Most
2. **Math Reasoning Benchmarks**: 3 papers -- MATH, GSM8K, DeepMind math dataset
3. **Autoformalization and Formal Math**: 5 papers -- Szegedy 2020, Autoformalization with LLMs, DSP, GPT-f, NaturalProofs
4. **Knowledge Representation for Science**: 3 papers -- KG survey (Hogan), KG survey (Ji), Math ontologies (Lange)
5. **Interactive Scientific Computing**: 2 papers -- IPython, Jupyter Notebooks
6. **Code Generation for Science**: 4 papers -- Codex, AlphaCode, StarCoder, Drori et al. university math
