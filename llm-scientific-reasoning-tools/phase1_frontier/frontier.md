# Phase 1: Frontier — LLM-Assisted Scientific Reasoning & Knowledge Tools

**Date**: 2026-03-10 | **Coverage**: 2024-2026 | **Unique papers**: ~70 (after deduplication across 3 search streams)

## Research Topic

Using LLMs to transform complex scientific knowledge (mathematical equations, physics theories) into human-understandable formats (knowledge trees, graphs, natural language, code) — extending coding assistant paradigms for scientific reasoning, learning, and software maintenance.

---

## 9 Thematic Clusters Identified

### Cluster 1: AI for Science Agents & Autonomous Discovery (11 papers)
**Key insight**: Multi-agent LLM architectures are becoming the dominant paradigm for autonomous scientific discovery.

| Paper | Venue | Key Contribution |
|-------|-------|-----------------|
| From Automation to Autonomy (Zheng+) | EMNLP 2025 | Three-tier framework: Tool→Analyst→Scientist |
| Towards Scientific Intelligence (Ren+) | arXiv 2503.24047 | Survey of LLM-based scientific agents |
| Agentic AI for Scientific Discovery (Gridach+) | ICLR 2025 | Categorizes agentic systems across domains |
| From AI for Science to Agentic Science (Wei+) | arXiv 2508.14111 | Introduces "Agentic Science" paradigm |
| AlphaEvolve (Novikov+ DeepMind) | arXiv 2506.13131 | Evolutionary coding agent; improved Strassen algorithm |
| FunSearch (Romera-Paredes+ DeepMind) | Nature 2024 | LLM+evaluator+evolution for math discovery |
| DREAMS (Wang+) | arXiv 2507.14267 | Multi-agent DFT simulation, <1% error |
| ChemGraph (Pham+) | Comm. Chemistry | Graph NN + LLM for computational chemistry |
| AgenticSciML (Jiang+ Karniadakis) | arXiv 2511.07262 | 10+ specialized agents, 4 OOM error reduction |
| SciAgents (Ghafarollahi+ Buehler) | Adv. Materials 2025 | KG + LLM multi-agent for materials discovery |
| 34 LLM Applications in MatSci/Chem (Zimmermann+) | arXiv 2505.03049 | Hackathon review spanning 7 research areas |

### Cluster 2: Autoformalization — Math & Physics (10 papers)
**Key insight**: Autoformalization is expanding beyond pure math into physics (PDE, quantum mechanics, high-energy physics).

| Paper | Venue | Key Contribution |
|-------|-------|-----------------|
| Autoformalization Survey (Weng+) | arXiv 2505.23486 | Comprehensive survey: data→model→assessment |
| PDE-Controller (Soroco+) | ICML 2025 | NL→formal PDE specs (>64%), executable code (>82%) |
| MerLean (Ren+) | arXiv 2602.16554 | Agentic LaTeX→Lean4 for quantum computation |
| FormL4/PDA (Lu+) | ICLR 2025 | Process-driven autoformalization with compiler feedback |
| ProofBridge (Jana+) | arXiv 2510.15681 | Joint embedding for NL→Lean4 proof translation |
| IndiMathBench (Microsoft) | arXiv 2512.00997 | 312 olympiad problems, human-verified Lean4 |
| Lean4Physics (Li+) | ICLR 2026 | First formal physics benchmark in Lean4 |
| PhysLean/HepLean (Tooby-Smith) | Comp. Phys. Comm. 2025 | Maxwell, QHO, Wick's theorem in Lean4 |
| Formal Math Reasoning (Yang+) | arXiv 2412.16075 | Position paper: proof assistants as complementary path |
| QuaSAR (Ranaldi+) | ACL 2025 | Quasi-symbolic CoT: +8% on adversarial benchmarks |

### Cluster 3: Equation-to-Code & Symbolic Regression (8 papers)
**Key insight**: Equations-as-programs is a proven paradigm — LLMs can bridge NL scientific knowledge and formal math via code.

| Paper | Venue | Key Contribution |
|-------|-------|-----------------|
| LLM-SR (Shojaee+) | ICLR 2025 Oral | Equations as executable programs + evolutionary search |
| LLM-SRBench (Shojaee+) | ICML 2025 Oral | 239 problems; best=31.5% symbolic accuracy |
| PiT-PO (Wang+) | arXiv 2602.10576 | RL-based equation discovery with physics constraints |
| SymCode (Bagheri Nezhad+) | EACL 2026 | Math→SymPy code, +13.6pp over baselines |
| KeplerAgent (Yang+) | arXiv 2602.12259 | Physics-guided agent mimicking scientific method |
| DotaMath (Li+) | arXiv 2407.04078 | Decompose math→code subtasks with self-correction |
| Open Proof Corpus (Dekoninck+) | arXiv 2506.21621 | 5000+ human-evaluated LLM proofs including IMO |
| TTE/SciEvo (Lu+) | arXiv 2601.07641 | Test-time tool evolution for scientific reasoning |

### Cluster 4: PDE Solving as Code Generation (7 papers)
**Key insight**: LLMs can generate PDE solvers competitive with expert-written code, but self-debugging (4+ rounds) is essential.

| Paper | Venue | Key Contribution |
|-------|-------|-----------------|
| CodePDE (Li+) | TMLR | First systematic eval of LLM PDE solving as code gen |
| AutoNumerics (Du+) | arXiv 2602.17607 | Multi-agent pipeline, 24 PDE problems |
| PDE-SHARP (Fazliani+) | arXiv 2511.00183 | 3-stage framework, <13 solver evals, 4x accuracy |
| Lang-PINN (He+) | arXiv 2510.05158 | NL→PINN, 3-5 OOM MSE reduction |
| OpInf-LLM (Wang+) | arXiv 2602.01493 | Operator inference + LLM for parametric PDEs |
| ATHENA (Toscano+) | NeurIPS MTI-LLM 2025 | Hierarchical evolutionary numerical algorithms |
| MCP-SIM (Park+) | npj AI 2026 | 100% success on 12 physics simulation tasks |

### Cluster 5: Mathematical Knowledge Graphs & Ontologies (7 papers)
**Key insight**: Ontology-guided LLM extraction outperforms free-form extraction for mathematical knowledge.

| Paper | Venue | Key Contribution |
|-------|-------|-----------------|
| AutoMathKG (Bian+) | arXiv 2505.13406 | Auto-updating math KG from ProofWiki/arXiv/textbooks |
| Math KG for Manufacturing (Cha+) | arXiv 2601.05298 | Equation ontology for additive manufacturing |
| Enhancing Math KGs (MDPI) | Modelling 2025 | LaTeX→KG with hypothesis/proof ontology |
| PyIRK for Control (Fiedler+) | arXiv 2511.02759 | Interactive semantic layer for engineering knowledge |
| LLMs4OL Challenge | ISWC 2024-2025 | Benchmark for LLM ontology learning |
| Scholarly Ontology Generation (Aggarwal+) | IP&M | 17 LLMs evaluated on IEEE Thesaurus |
| LinkQ Visualization (Li+) | arXiv 2505.21512 | KG exploration with 5 visual mechanisms |

### Cluster 6: AI-Generated Textbooks & Interactive Learning (9 papers)
**Key insight**: Nonlinear interfaces (knowledge trees, mind maps, canvases) substantially outperform linear text for learning.

| Paper | Venue | Key Contribution |
|-------|-------|-----------------|
| Learn Your Way/LearnLM (Google) | arXiv 2509.13348 | +11% retention via multi-representation AI textbook |
| KMP-Bench (Shi+) | arXiv 2603.02775 | Solving ≠ teaching: pedagogical intelligence benchmark |
| Inquizzitor (Cohn+) | AAAI 2026 | Adaptive scaffolding grounded in cognitive science |
| PedagogicalRL-Thinking (Lee+) | arXiv 2601.14560 | RL for pedagogical reasoning traces |
| LLM Tutors for Outcomes (Scarlatos+) | AIED 2025 | Optimizing for actual student learning |
| Concept Map Generation (Zhai) | arXiv 2509.14554 | Systematic review of 28 studies on LLM concept maps |
| AI Concept Maps (Tytenko) | iTextbooks 2025 | Hierarchical drill-down from e-books |
| Mindalogue (Zhang+) | CHI 2025 | Nonlinear "nodes+canvas" interface for AI learning |
| MindMap KG Prompting (Wen+) | ACL 2024 | KG-grounded "mind map" for transparent reasoning |

### Cluster 7: Scientific Coding Benchmarks (5 papers)
**Key insight**: LLMs solve <10% of research-grade scientific coding problems in realistic settings.

| Paper | Venue | Key Contribution |
|-------|-------|-----------------|
| SciCode (Tian+) | NeurIPS 2024 D&B | 338 subproblems, 16 subfields; best=4.6% realistic |
| FEM-Bench (Mohammadzadeh+) | arXiv 2512.20732 | FEM code gen; best=73.8% joint success |
| HeuriGym (Chen+) | ICLR 2026 | Heuristic algorithm gen; QYI=0.6 vs expert 1.0 |
| PHYBench (Qiu+) | arXiv 2504.16074 | 500 physics problems; best=36.9% vs human 61.9% |
| MatTools (Liu+) | arXiv 2505.10852 | Materials science tool benchmark, 69K QA pairs |

### Cluster 8: IDE/CLI & Workflow Extensions (7 papers/tools)
**Key insight**: MCP is emerging as the standard for connecting LLM agents to scientific computing infrastructure.

| Paper/Tool | Source | Key Contribution |
|------------|--------|-----------------|
| MCP for Science (Pan+ LBNL) | arXiv 2508.18489 | MCP servers for Globus, Galaxy, NERSC |
| MATLAB MCP Server (MathWorks) | GitHub | Official MCP for MATLAB + hardware access |
| GitHub Copilot CLI | GitHub 2026 | Agentic terminal environment, multi-model |
| Claude/Codex on Agent HQ | GitHub 2026 | Cross-client agentic workflows |
| InfraNodus VSCode | GitHub | Knowledge graph visualization in IDE |
| Jupiter (Li+) | AAAI 2026 | MCTS over Jupyter notebook workflows |
| Scientific Workflows (Yildiz+) | arXiv 2412.10606 | LLMs struggle with domain-specific workflows |

### Cluster 9: Scientific Software & Legacy Code (6 papers)
**Key insight**: LLMs provide strong starting points but human verification remains essential for scientific code.

| Paper | Venue | Key Contribution |
|-------|-------|-----------------|
| DREAMS (Wang+) | arXiv 2507.14267 | Multi-agent DFT automation |
| ChronoLLM (Wang+) | arXiv 2508.13975 | Domain-specific LLM for PyChrono simulation |
| CodeScribe (Dhruv+) | arXiv 2410.24119 | Fortran→C++ for LHC particle simulation |
| Fortran→C++ Study (Ranasinghe+) | arXiv 2504.15424 | Systematic cross-platform translation eval |
| HPC-Coder-V2 (Chaturvedi+) | arXiv 2412.15178 | Best open-source parallel code LLM |
| Code-Scribe Extensions (Dubey+) | Workshop 2025 | New algorithm implementation (not just translation) |

---

## Key Frontier Trends (2025-2026)

1. **Equations-as-programs**: Code is the universal bridge between mathematical notation and computation (LLM-SR, SymCode, CodePDE)
2. **Multi-agent dominance**: 6+ agent systems are standard for complex scientific tasks (DREAMS, AgenticSciML, MCP-SIM, ATHENA)
3. **Autoformalization beyond math**: Lean4 now covers physics (Lean4Physics), quantum computing (MerLean), PDEs (PDE-Controller)
4. **Nonlinear knowledge interfaces**: Trees, graphs, canvases beat linear chat (Learn Your Way, Mindalogue, concept maps)
5. **MCP as scientific infrastructure**: Standard protocol connecting LLMs to HPC, simulation tools, instruments (LBNL, MATLAB)
6. **Pedagogical ≠ reasoning**: Teaching requires distinct training beyond problem-solving (KMP-Bench, PedagogicalRL-Thinking)
7. **Self-correction is mandatory**: 4+ debug rounds for PDE code, <5 iterations for physics simulation
8. **Benchmarks expose gaps**: <10% solve rate on realistic scientific coding (SciCode), 31.5% on equation discovery (LLM-SRBench)
9. **Tool evolution**: Static tool libraries → dynamic tool synthesis at inference time (TTE/SciEvo)

---

## Statistics

| Cluster | Papers | Top Venues |
|---------|--------|------------|
| AI4Science Agents | 11 | Nature, EMNLP, ICLR, Adv. Materials |
| Autoformalization | 10 | ICML, ICLR 2025/2026, ACL, Comp. Phys. Comm. |
| Equation-to-Code | 8 | ICLR Oral, ICML Oral, EACL |
| PDE as Code Gen | 7 | TMLR, NeurIPS Workshop, npj AI |
| Math KG & Ontology | 7 | IP&M, ISWC, MDPI |
| AI Textbooks | 9 | CHI, AAAI, AIED, ACL |
| Sci Coding Benchmarks | 5 | NeurIPS, ICLR |
| IDE/CLI & Workflow | 7 | AAAI, GitHub |
| Sci Software | 6 | arXiv, Workshop |
| **Total (deduplicated)** | **~70** | |

*Generated 2026-03-10. Merged from 3 parallel search streams.*
