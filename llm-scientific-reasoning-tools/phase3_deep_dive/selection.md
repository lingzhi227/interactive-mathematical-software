# Phase 3: Paper Selection for Deep Dive

## Selection Criteria
1. Directly relevant to the user's vision (math→code→explanation pipeline)
2. Priority to peer-reviewed papers at top venues
3. Cover all major clusters: equation-to-code, autoformalization, knowledge representation, AI textbooks, scientific agents, verification
4. Include both foundational and cutting-edge works

## Selected Papers (12)

### Equation-to-Code (3 papers)
1. **LLM-SR** (Shojaee+ ICLR 2025 Oral) — Core paradigm: equations as programs
2. **SymCode** (Bagheri Nezhad+ EACL 2026) — Math→SymPy verified code
3. **CodePDE** (Li+ TMLR) — PDE solving as code generation

### Autoformalization (3 papers)
4. **PDE-Controller** (Soroco+ ICML 2025) — NL→formal PDE→executable code
5. **MerLean** (Ren+ 2026) — LaTeX→Lean4 for quantum computation
6. **Lean4Physics** (Li+ ICLR 2026) — First physics benchmark in Lean4

### Knowledge Representation & Interactive Learning (3 papers)
7. **Learn Your Way/LearnLM** (Google 2025) — AI-augmented textbook with mind maps
8. **Mindalogue** (Zhang+ CHI 2025) — Nonlinear interaction for learning
9. **AutoMathKG** (Bian+ 2025) — Automated mathematical knowledge graph

### Scientific Agents & Verification (3 papers)
10. **MCP-SIM** (Park+ npj AI 2026) — Multi-agent physics simulation + explanation
11. **MCP for Science** (Pan+ LBNL 2025) — MCP servers for HPC/science
12. **Vericoding Benchmark** (Bursuc+ 2025) — Formally verified code synthesis

## Rationale
These 12 papers span the complete pipeline from the user's vision:
- Document→LaTeX extraction (Nougat as prerequisite)
- LaTeX→symbolic (SymCode, LLM-SR)
- Symbolic→formal (PDE-Controller, MerLean, Lean4Physics)
- Formal→code (CodePDE, MCP-SIM)
- Code→explanation (LearnLM, Mindalogue)
- Knowledge organization (AutoMathKG)
- Infrastructure (MCP for Science)
- Verification (Vericoding)
