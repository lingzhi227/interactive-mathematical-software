# Phase 3 Deep Dive: Knowledge Representation and Interactive Learning with LLMs

## Papers Covered
- Paper 7: Towards an AI-Augmented Textbook (Learn Your Way / LearnLM)
- Paper 8: Mindalogue: LLM-Powered Nonlinear Interaction for Effective Learning and Task Exploration
- Paper 9: AutoMathKG: The Automated Mathematical Knowledge Graph Based on LLM and Vector Database

---

## Paper 7: Towards an AI-Augmented Textbook (Learn Your Way / LearnLM)

**Citation**: LearnLM Team, Google. "Towards an AI-Augmented Textbook." arXiv:2509.13348, 2025.
**Venue**: Google Research (pre-print)
**Blog**: https://research.google/blog/learn-your-way-reimagining-textbooks-with-generative-ai/

### Problem

Traditional textbooks are a one-size-fits-all medium. Any new representation or adaptation requires arduous human effort, making it impossible to personalize textbooks at scale. Learners have diverse grade levels, interests, and preferred modalities, but a single static textbook cannot accommodate these differences. The core question: can generative AI transform textbooks into personalized, multi-representation learning experiences while maintaining content integrity?

### Key Contributions

- **Multi-representation generation pipeline**: Transforms a single textbook chapter into five distinct formats (immersive text, slides with narration, audio-graphic lesson, mind maps, quizzes) using Gemini 2.5 Pro without fine-tuning (except for visual illustrations).
- **Two-axis personalization**: Adapts content along (1) grade level (via Flesch-Kincaid Grade re-leveling) and (2) learner interests (e.g., sports, music, food), where interest-relevant segments are selectively replaced while preserving ~80% of original structure.
- **Pedagogical validation**: Expert evaluation across 10 rubric dimensions (accuracy, coverage, engagement, cognitive load, metacognition, etc.) with all components scoring >0.90.
- **RCT-based efficacy evidence**: Randomized controlled trial with 60 students (ages 15-18) showing statistically significant improvements in both immediate (p=0.03) and 3-day retention assessments (p=0.03).

### Methodology

**Two-stage pipeline:**

**Stage 1 -- Text Personalization:**
- Grade-level adaptation uses Flesch-Kincaid Grade readability metric. Three levels tested: 7th grade, 10th grade, undergraduate.
- Interest personalization uses a "focused" approach: the system identifies amenable text segments, replaces only those portions with interest-mapped analogies (e.g., Newton's Third Law rewritten for basketball or art), and highlights personalized text visually.

**Stage 2 -- Content Transformations (Multiple Representations):**

1. **Immersive Text (Core View)**: Personalized comprehensive text with embedded enhancements:
   - *Timeline component*: Identifies sequences (historical, experimental, algorithmic), generates visual timelines with interactive drag-and-drop practice.
   - *Memory aids*: Generates mnemonics with semantic association to material.
   - *Visual illustrations*: Fine-tuned model for pedagogical images (standard image generators trained for realism fail at simple/educational images).
   - *Embedded questions*: Multiple-choice, grounded to specific source segments, with immediate feedback.

2. **Slides and Narration**: Slide sequences with interest-capturing questions, engagement activities, and optional narration designed to complement (not repeat) slide text.

3. **Audio-Graphic Lesson**: Simulated teacher-student conversation using independent Gemini personas. The student character may reveal misconceptions before receiving instruction. Coupled with dynamic graphical representation of key concepts and relationships (dual coding theory).

4. **Mind Maps**: Hierarchical organization with expandable/collapsible nodes at multiple granularity levels. Leaf nodes annotated with text and images from source material. Serves as post-session organizational tool or quick-reference reminder.

5. **Quizzes**: Section-level, 5-10 multiple-choice questions at various difficulty levels. Provides numerical score plus targeted feedback with "Glows" (strengths) and "Grows" (areas for improvement).

**Model**: Gemini 2.5 Pro (via LearnLM pedagogy-infused family) for all components; fine-tuned model only for visual illustrations.

### Architecture/Pipeline

```
Source PDF (textbook chapter)
    |
    v
[Stage 1: Personalization]
    |-- Grade-level re-leveling (Flesch-Kincaid)
    |-- Interest-based segment replacement
    v
Personalized Source Text
    |
    v
[Stage 2: Multi-Representation Generation]
    |-- Immersive Text (+ timelines, memory aids, illustrations, embedded questions)
    |-- Slides + Narration
    |-- Audio-Graphic Lesson (teacher-student dialogue + concept graphics)
    |-- Mind Map (hierarchical, expandable nodes)
    |-- Section Quizzes (MCQ + Glows/Grows feedback)
    v
Learn Your Way Interactive Interface
```

### Experiments

**Pedagogical Expert Evaluation:**
- Source materials: 10 OpenStax PDFs (world history, biology, physics, economics, astronomy, sociology, psychology)
- 3 grade levels x 3 interests = 9 combinations, 3 random assignments per PDF
- 3 independent raters per component
- 10-dimension rubric (accuracy, coverage, emphasis, engagement, cognitive load, active learning, metacognition, motivation, adaptability, clarity of learning intentions)
- Rating scale: Agree (1.0), Neutral (0.5), Disagree (0.0)
- **Result**: All components scored >0.90 across all pedagogical axes. Visual illustration was the lowest performer. Slides alone scored lowest on engagement; slides + narration scored significantly higher.

**Randomized Controlled Trial:**
- 60 students (ages 15-18) from Chicago area schools
- Selection based on reading comprehension (scores 4-9 out of 10, mean 6.4, SD 2.3)
- Material: "Brain Development for Adolescents" chapter (LibreTexts)
- Conditions: Learn Your Way (experimental) vs. Adobe Acrobat Reader (control)
- Protocol: 5-min intro, 20-40 min learning, 15-min immediate assessment, 10-min survey, 20-min interview, 3-day follow-up retention test (58/60 completed)
- Assessment: 15 questions across Bloom's taxonomy (short answer, MCQ, matching)

**Key Results:**
| Metric | Learn Your Way | Digital Reader | p-value |
|--------|---------------|----------------|---------|
| Immediate assessment | Higher | Baseline | p=0.03 |
| 3-day retention | 78% | 67% (+11%) | p=0.03 |
| Would use again | 93% | 67% | -- |
| Felt more comfortable | 100% | -- | -- |

- All experience dimensions (engagement, understanding, motivation) significantly favored Learn Your Way.
- All participants used quizzes; majority used at least one additional content transformation.

### Limitations

- **Component attribution**: Cannot isolate which component drives learning gains (quizzes? mind maps? personalization?). The system is evaluated holistically.
- **Narrow scope**: Single chapter evaluation only. Broader topic coverage untested.
- **Cost of detailed experiments**: Systematic ablation studies acknowledged as expensive.
- **No long-term study**: Only 3-day retention measured; semester-long effects unknown.
- **Implicit personalization missing**: Currently relies on explicit learner input (grade, interests) rather than adaptive sensing of learner behavior.

### Connections to Interactive Scientific Learning

This paper is highly relevant to the vision of transforming scientific knowledge into human-understandable formats:

- **Mind maps as knowledge trees**: Directly implements hierarchical, expandable knowledge structures where mathematical/scientific concepts are organized in parent-child relationships with annotations.
- **Multiple representation principle**: Demonstrates that the same scientific content can be rendered as text, audio dialogue, visual slides, mind maps, and interactive assessments -- and that this multiplicity improves learning outcomes.
- **Dual coding for scientific concepts**: Audio-graphic lessons combine auditory explanation with dynamic concept-relationship diagrams, directly relevant to visualizing scientific knowledge structures.
- **Timelines as structured representations**: Converts sequential scientific processes (experimental procedures, algorithmic steps) into interactive visual timelines.
- **Personalization of scientific analogies**: Shows how abstract physics concepts (Newton's Third Law) can be mapped to familiar domains (sports, art) to activate prior knowledge -- a form of knowledge translation.
- **Evidence-based validation**: Provides the strongest empirical evidence (RCT) among the papers surveyed that AI-generated multi-representation learning materials improve both comprehension and retention.

---

## Paper 8: Mindalogue: LLM-Powered Nonlinear Interaction for Effective Learning and Task Exploration

**Citation**: Rui Zhang, Ziyao Zhang, Fengliang Zhu, Jiajie Zhou, Anyi Rao. "Mindalogue: LLM-Powered Nonlinear Interaction for Effective Learning and Task Exploration." CHI 2025, arXiv:2410.10570.
**Venue**: ACM CHI Conference on Human Factors in Computing Systems, April 26-May 1, 2025, Yokohama, Japan
**Affiliations**: Tsinghua University, The New School, Beijing Institute of Technology, HKUST

### Problem

Current LLM interfaces (ChatGPT, Claude, Gemini) use linear, sequential text-based interaction: users pose a question, receive a response, and subsequent questions depend on previous responses. This linear model fails for complex tasks involving multi-layered information, multiple subtasks, and non-sequential exploration (brainstorming, structured knowledge learning, large project analysis). Users are forced to repeatedly scroll, compare, copy, and re-contextualize information across conversation turns, creating cognitive overload and reducing interaction efficiency. Useful information becomes scattered across long conversations with overlaps and conflicts.

### Key Contributions

- **Formative study identifying design requirements**: 11-participant study generating 618 initial codes, refined into 4 design considerations (DC1-DC4) that establish the need for nonlinear interaction, deeper answers, graphical representation, and structured information.
- **"Nodes + Canvas" interaction model**: A non-linear interaction paradigm where LLM responses are decomposed into independent, manipulable nodes on a 2D canvas with a 4-layer mind map hierarchy, replacing the linear chat scroll.
- **Three AI-powered deep exploration functions**: Each node supports Explanation (detailed concept expansion), Examples (3 specific cases), and Exploration (user-directed new queries), with parent-node context propagation.
- **Comparative evaluation**: 16-participant study across 4 task categories showing Mindalogue outperforms linear LLM interfaces on 4 of 5 dimensions (convenience, controllability, cognitive load reduction, result reliability).

### Methodology

**Phase 1: Formative Study**
- 11 participants (ages 19-29, mean=23.91, SD=2.57; 6 women, 5 men)
- 3 novice, 4 moderate, 4 advanced LLM users
- Remote interviews, 45-60 minutes each, recorded and transcribed
- Reflexive thematic analysis yielded 618 initial codes, merged into themes

**Four Design Considerations Emerged:**
1. DC1 (Linear interaction limitations): Frequent scrolling, step-by-step Q&A dependency, cross-topic questioning difficulty
2. DC2 (Deeper answers): Responses too broad and shallow for complex tasks
3. DC3 (Graphical representation): Text alone cannot convey multi-level structures; mind maps and flowcharts deemed more intuitive for memory retention and operational efficiency
4. DC4 (Structured information): Need for tables, flowcharts, and mind maps to organize long conversations and maintain logical coherence

**Phase 2: System Design (Four Design Goals)**
- DG1: Non-linear interaction for exploration freedom (node-based navigation)
- DG2: Deep information feedback (progressive, layered responses)
- DG3: Graphical representation for comprehension (mind map visualization)
- DG4: Hierarchical structure and information management (logical classification)

### Architecture/Pipeline

**Core Model**: "Nodes + Canvas" powered by ChatGPT-4.0

```
User Query
    |
    v
[LLM Generation with Predefined Task Prompts]
    |
    v
[4-Layer Mind Map Structure]
    |-- Layer 1: Root concept (brief phrase)
    |-- Layer 2: Sub-topics (brief phrases)
    |-- Layer 3: Sub-sub-topics (brief phrases)
    |-- Layer 4: Detailed descriptions (complete sentences)
    v
[Interactive Canvas]
    |-- Nodes: draggable, selectable, color-coded by subtopic
    |-- Per-node AI functions:
    |     |-- Explanation: detailed concept expansion
    |     |-- Examples: 3 specific related cases
    |     |-- Exploration: user-directed new queries (child nodes)
    |-- Context propagation: child nodes inherit parent context
    |-- Redundancy detection: prevents duplicate content
    |-- Spatial layout optimization: logical node positioning
    v
[User Exploration]
    |-- Nonlinear navigation (jump between any nodes)
    |-- Zoom/drag/edit capabilities
    |-- Custom node operations (add, delete, edit, move)
```

**Key Technical Features:**
- Vector graphics rendering for scalable node clarity
- Color coding per subtopic for visual hierarchy
- Parent-child context propagation ensures generated content relates to background
- Redundancy detection prevents duplicate nodes
- Predefined prompts per task type prevent overly broad responses

### Experiments

**Evaluation Study Design:**
- 16 participants (6 male, 10 female; ages 21-30, mean=24.06, SD=2.52)
- 3 monthly, 5 weekly, 8 daily LLM users
- Education: 10 master's, 4 bachelor's, 2 doctoral
- ~120 minutes total per participant
- Compared: Mindalogue (S1) vs. Mindmap Master (S2, linear LLM with mind map output)
- Both powered by ChatGPT-4.0 with identical settings

**Four Task Categories:**
1. Knowledge Exploration (unfamiliar): Machine Learning, Dadaism, Relativity, Industrial Revolution
2. Brainstorming (unfamiliar): Future Fashion, Dream Experiences, Digital Food, Event Planning
3. Product Analysis (familiar): Recommend a mobile app
4. City Description (familiar): Describe a favorite city

Each task: max 15 minutes. Cross-domain assignment to ensure unfamiliarity.

**Quantitative Results:**

| Metric | Mindalogue (S1) | Linear (S2) |
|--------|----------------|-------------|
| Avg time per task | 10.16 min (SD=3.20) | 7.78 min (SD=2.50) |
| Avg actions per task | 11.19 (SD=4.90) | 6.81 (SD=3.04) |
| Unfamiliar task time | 12.13 min | 8.94 min |
| Familiar task time | 8.19 min | 6.63 min |

Interpretation: Mindalogue users spent more time and performed more actions, indicating greater exploration space and engagement (not inefficiency).

**Task Support Ratings (1-7 scale):**
- Knowledge Exploration: S1=5.13 vs S2=4.50
- Brainstorming: S1=5.50 vs S2=5.00
- Product Analysis: S1=6.13 vs S2=5.88 (strongest showing)
- City Description: S1=5.50 vs S2=5.00

**Five-Dimension Comparison (Paired t-tests):**
- Convenience: S1 significantly better
- Usability: S2 slightly better (familiar chat interface easier to learn; S1 confidence p=0.008 favoring S2)
- Controllability: S1 better
- Cognitive Load Reduction: S1 better (psychological difficulty p=0.033 favoring S1)
- Result Reliability: S1 better
- Overall: S1 outperforms on 4/5 dimensions

**Cronbach's Alpha**: S1=0.74, S2=0.84 (both >0.7 threshold)

**Qualitative Findings:**
- Node explanations and case functions helped users "quickly grasp the entire task"
- Tree diagrams allow "clearer comprehension, improving efficiency"
- Structured presentation enhanced perceived reliability, especially for unfamiliar topics
- Users reported S2 content as "too repetitive" and "like chatting with a search engine"
- S1 excelled at: learning new knowledge, brainstorming, constructing frameworks
- Users requested: color/bold marking, auto sub-node generation, undo features, image generation, citation sources

### Limitations

- **Short session only**: Study confined to single sessions; does not capture long-term engagement spanning months/years typical of actual learning.
- **Individual use only**: Learning is often collaborative; individual preferences may shape visualization perceptions.
- **Learning curve**: Non-linear interaction's high degree of freedom caused some users to "lose direction" in information-dense tasks.
- **Content redundancy**: Node expansion sometimes produced repetitive content, reducing depth.
- **Annotation accuracy**: Occasional errors in complex multi-layered relationships (e.g., incorrect entity associations, misinterpreted node connections).
- **No longitudinal field study**: Authors call for randomized, controlled, longitudinal study for more accurate assessment.
- **Small sample size**: 16 participants limits generalizability.

### Connections to Interactive Scientific Learning

This paper directly addresses the vision of knowledge trees and mind maps for scientific understanding:

- **Mind map as primary interaction paradigm**: The 4-layer hierarchical mind map IS the interface, not an auxiliary feature. Scientific concepts would be naturally organized as expandable knowledge trees.
- **Nonlinear exploration of knowledge**: Users can jump between any concept nodes without sequential constraints -- exactly the kind of free exploration needed when navigating mathematical or physics concept spaces.
- **Progressive disclosure for complexity management**: The 3 brief layers + 1 detailed layer pattern mirrors how scientific knowledge could be presented: high-level overview first, then drill-down to equations and proofs.
- **Contextual deep exploration**: The Explanation/Examples/Exploration triad per node maps directly to how one might explore a scientific concept: understand the definition, see applications, then investigate related concepts.
- **Color-coded knowledge taxonomy**: Visual categorization by topic mirrors how knowledge trees could distinguish between definitions, theorems, proofs, and applications in mathematics.
- **Parent-child context propagation**: When exploring a child concept, the system automatically provides parent context -- essential for maintaining coherence when navigating deep scientific knowledge hierarchies.
- **Redundancy detection**: Prevents duplicate concepts in the knowledge tree, maintaining clean ontological structure.
- **Key limitation for scientific use**: The system currently uses ChatGPT-4.0 without domain-specific knowledge bases; integrating domain knowledge bases is identified as a critical future direction for professional/scientific domains.

---

## Paper 9: AutoMathKG: The Automated Mathematical Knowledge Graph Based on LLM and Vector Database

**Citation**: Rong Bian, Yu Geng, Zijian Yang, Bing Cheng. "AutoMathKG: The automated mathematical knowledge graph based on LLM and vector database." arXiv:2505.13406, 2025.
**Venue**: Pre-print (Academy of Mathematics and Systems Science, Chinese Academy of Sciences)
**Affiliations**: AMSS/CAS, University of Chinese Academy of Sciences

### Problem

Constructing mathematical knowledge graphs from natural language text faces two fundamental challenges: (1) existing approaches are constrained by corpus completeness, often discarding or manually supplementing incomplete knowledge without exploiting the inherent logic of mathematical knowledge; (2) existing KGs cannot automatically integrate knowledge from different sources, requiring substantial human effort for updates. Furthermore, while formal mathematics (Lean 4, Metamath) enables verification, it does not directly promote human understanding the way natural mathematical language does. The goal is to build a KG that captures mathematics in natural language, supports automatic updating, and enables fuzzy semantic search.

### Key Contributions

- **Multi-source mathematical KG**: Integrates knowledge from 4 sources (ProofWiki, textbooks, arXiv papers, TheoremQA) into a directed graph of 13,388 entities with 29,459 edges, representing Definitions, Theorems, and Problems with 9 tactic-labeled edge types.
- **MathVD vector database with dual embedding strategies**: Two SBERT-based embedding approaches (concatenation and weighted-sum) for entity similarity search, outperforming 5 KG embedding baselines (TransE, KG2E, HoLE, R-GCN, BoxE) on reachability queries at q>=5.
- **Automatic knowledge completion via Math LLM**: A gemma-7b-it model with 3 task-specific LoRA adapters (Application, Calculation, Proof) plus retrieval augmentation and self-calibration for filling missing proofs/solutions.
- **Automatic knowledge fusion mechanism**: New entities are embedded, compared against existing KG via cosine similarity in MathVD, and LLM determines whether to merge with an existing entity or add as new -- enabling automatic KG expansion.

### Methodology

**Mathematical Knowledge Model:**

Mathematics is represented as a directed graph G = {V, E} where:
- V = vertices representing mathematical entities of 3 types:
  - **Definition** (Def): Concise statements of mathematical concepts; foundational building blocks
  - **Theorem** (Thm): Logically proven statements including theorems, propositions, corollaries, lemmas; each includes proof process (potentially multiple proofs)
  - **Problem** (Prob): Mathematical problems with solution processes demonstrating application of definitions and theorems
- E = directed edges representing reference relationships. Edge from vi to vj means vj references vi in its content, proof, or solution.

**Nine Tactic Labels for Edges** (inspired by Lean 4):
premise, assumption, lemma, proposition, corollary, calculation, enumeration, definition, conclusion

**Graph allows cycles**: e.g., Pythagorean Theorem and Sum of Squares of Sine and Cosine are mutually referenced.

**Entity Storage**: JSON format with 17 attributes organized in 3 levels:
1. Basic information (id, type, label, title, field, contents, source)
2. Advanced information (bodylist with step-by-step logical segments and action labels, proofs, solutions)
3. Query information (in_refs, in_ref_ids, out_refs, out_ref_ids, references_tactics)

**Information Extraction Pipeline:**

Step 1 -- Rule-based extraction: LaTeX environment names (\begin{theorem}...\end{theorem}) parsed to extract entity type, title, contents, references, source.

Step 2 -- LLM augmentation via ICL: 12 prompt templates applied using Llama-2-7b to complete remaining attributes (title, field, bodylist, references_tactics, refs). Bodylist segments each entity's content into logical steps with tactic labels (e.g., "Let S be a set" -> action: "premise"; "Then (G,*) is called..." -> action: "definition").

### Architecture/Pipeline

```
[Data Sources]
    |-- ProofWiki (9,496 samples: 4,743 Def, 1,811 Thm, 2,942 Other)
    |-- Textbooks (1,605 samples: 361 Def, 1,209 Thm, 35 Other) [8 textbooks]
    |-- ArXiv Papers (538 samples: 134 Def, 399 Thm, 5 Other) [20 papers]
    |-- TheoremQA (1,749 samples: 1,084 Prob, 665 Other)
    v
[Rule-Based Extraction]
    |-- LaTeX environment parsing
    |-- Basic attribute extraction (id, type, contents, refs, source)
    v
[LLM Augmentation (Llama-2-7b, ICL)]
    |-- 12 prompt templates for attribute completion
    |-- Bodylist generation (logical segmentation with tactic labels)
    |-- Reference discovery and tactic labeling
    v
[AutoMathKG Construction]
    |-- 13,388 entities, 29,459 directed edges, 145 simple cycles
    |-- 4,216 head nodes (no incoming), 3,789 leaf nodes (no outgoing)
    v
[MathVD Construction (SBERT: all-MiniLM-L6-v2, 384-dim)]
    |-- MathVD1: Concatenation strategy (all 5 fields -> single text -> embed)
    |-- MathVD2: Weighted-sum strategy (embed fields separately, weights: content=0.5, title=0.3, field=0.1, in_refs=0.05, out_refs=0.05)
    v
[Automatic Update Mechanisms]
    |
    |-- Knowledge Completion (Math LLM):
    |     |-- Base: gemma-7b-it fine-tuned on MathInstruct
    |     |-- 3 LoRA Adapters:
    |     |     |-- Application (orca-math-200k)
    |     |     |-- Calculation (GSM8K-sympy-v2, PoT)
    |     |     |-- Proof (NaturalProofs, CoT)
    |     |-- 2-Stage Retrieval Augmentation:
    |     |     |-- Exact search in AutoMathKG
    |     |     |-- Fuzzy search in MathVD
    |     |-- Self-Calibration: step-by-step verification with feedback loop
    |
    |-- Knowledge Fusion:
          |-- New text -> Rule-based extraction + LLM augmentation -> Input KG
          |-- Embed Input KG entities via SBERT -> Input VD
          |-- For each input entity, retrieve top-5 similar from Existing VD
          |-- LLM determines: merge with candidate or add as new entity
          |-- If multiple matches: LLM performs secondary judgment for best match
```

### Experiments

**Hardware**: 4x RTX 3090 GPUs (96GB VRAM), Python 3.10

**Experiment 1: Reachability Query Performance**
- Metric: Hits@q for k-hop reachability (can entities A and B be connected by <= k edges?)
- Setup: 100 random entities (50 Def, 30 Thm, 20 Prob), q = {1, 5, 10, 15}, k = {1, 2, 3, 4, 5}
- Baselines: TransE, KG2E, HoLE, R-GCN, BoxE (all via PyKEEN, 384-dim)

**Results (5-hop reachability):**

| Model | Hits@1 | Hits@5 | Hits@10 | Hits@15 |
|-------|--------|--------|---------|---------|
| TransE | 0.9610 | 0.7766 | 0.7182 | 0.6797 |
| BoxE | 0.9351 | 0.8338 | 0.7610 | 0.7247 |
| **MathVD1** | 0.8831 | **0.8364** | **0.8182** | **0.7861** |
| **MathVD2** | **0.8974** | **0.8385** | **0.8013** | **0.7786** |

MathVD outperforms all baselines at q>=5. TransE wins at q=1 due to supervised optimization.

**Experiment 2: Math LLM Reasoning**
- Dataset: 234 questions from GHOSTS benchmark, 6 categories
- Rating: 1-5 scale (5=nearly perfect, 1=irrelevant)

| Category | Samples | Rating |
|----------|---------|--------|
| Algebra | 46 | 3.78 (best) |
| Complement | 35 | 3.37 |
| Theorem | 27 | 3.37 |
| Prealgebra | 43 | 3.26 |
| Topology | 39 | 3.05 |
| Probability | 44 | 3.05 |

Assessment: "Commendable for a 7B parameter model" -- correct reasoning with mostly correct answers, strongest in algebra.

**Experiment 3: Internal Retrieval Precision**
- 50 test samples per VD (25 Def, 15 Thm, 10 Prob), top-10 retrieval
- Manual evaluation of relevance

| Type | MathVD1 | MathVD2 |
|------|---------|---------|
| Definition | 96.0% | 95.3% |
| Theorem | 94.0% | 94.0% |
| Problem | 97.0% | 96.0% |

KS test: no significant distribution difference between MathVD1 and MathVD2.

**Experiment 4: External Retrieval (Natural Language Queries)**
- Query "Expectation in Probability and Statistics" retrieves relevant theorems, definitions, and problems about expectations across both VDs.
- MathVD2 returns "Definition: expectation" as top result (most semantically relevant), while MathVD1 returns "Theorem: expectation of geometric distribution" first.

**Ablation Study: Reference Information Impact**
- Removing in_refs/out_refs from embedding: retrieval becomes purely linguistic (matches titles with similar words).
- With refs: retrieves semantically essential entities (e.g., "Boole's inequality" for "probability measure is subadditive" -- a direct consequence despite dissimilar titles).
- Conclusion: Reference information is crucial for capturing mathematical essence beyond surface-level text similarity.

### Limitations

- **Small LLM scale**: Only Llama-2-7b for augmentation and gemma-7b-it for Math LLM; larger models would likely improve quality significantly.
- **Formal corpus only**: Excludes informal sources (math forums, discussions, Stack Exchange) that contain different types of mathematical knowledge and pedagogical explanations.
- **No systematic quality evaluation**: Missing precision/recall for knowledge fusion accuracy, entity deduplication correctness, or graph-level quality metrics.
- **Limited corpus scale**: 13,388 entities from 8 textbooks and 20 papers is small relative to the breadth of mathematics.
- **No user study**: No evaluation of whether the KG actually helps humans learn or navigate mathematical knowledge.
- **SBERT limitations**: 384-dimensional embeddings with a general-purpose sentence transformer may not capture deep mathematical semantics (mathematical notation, symbolic reasoning).
- **Tactic label set**: 9 labels may be insufficient for the full range of mathematical reasoning patterns.

### Connections to Interactive Scientific Learning

This paper is the most directly relevant to building structured knowledge representations of scientific/mathematical content:

- **Mathematical knowledge as a directed graph**: The core model -- definitions, theorems, and problems as nodes with tactic-labeled reference edges -- is precisely a "knowledge tree" for mathematics, though with cycles allowed.
- **Three-entity ontology**: The Definition/Theorem/Problem trichotomy provides a clean schema for organizing mathematical knowledge that could generalize to physics (Law/Derivation/Application) or other sciences.
- **Tactic labels as edge semantics**: The 9 tactic types (premise, assumption, lemma, corollary, etc.) capture HOW knowledge elements relate, not just THAT they relate -- essential for building comprehensible knowledge structures.
- **Bodylist logical segmentation**: Decomposing theorems and proofs into step-by-step segments with action labels (inspired by Lean 4 tactics) bridges the gap between formal and informal mathematical reasoning.
- **Fuzzy semantic search via vector database**: MathVD enables natural-language queries to navigate the knowledge graph, making it accessible to learners who don't know exact terminology -- a key requirement for "human-understandable" knowledge interfaces.
- **Automatic expansion**: The knowledge fusion mechanism means the KG can grow as new papers/textbooks are published, maintaining currency without manual curation.
- **Reference-aware embeddings**: The ablation showing that reference information (graph structure) improves retrieval beyond text similarity alone validates that knowledge graph structure captures mathematical meaning not present in text.
- **Limitation for the vision**: No interactive visualization, no personalization, no pedagogical layer. This is infrastructure (the KG backend) that would need systems like Mindalogue or Learn Your Way as frontends to become a learning tool.

---

## Cross-Paper Analysis: Synthesis and Connections

### Complementary Roles in the Vision

These three papers address different layers of a complete system for interactive scientific learning:

| Layer | Paper | Role |
|-------|-------|------|
| **Knowledge Backend** | AutoMathKG (#9) | Structured knowledge graph with entity types, relationships, and semantic search |
| **Interaction Paradigm** | Mindalogue (#8) | Nonlinear, node-based exploration interface with progressive disclosure |
| **Content Generation** | Learn Your Way (#7) | Multi-representation content transformation with personalization |

A combined system would use AutoMathKG's structured mathematical knowledge graph as the backend, Mindalogue's node+canvas nonlinear interaction as the exploration interface, and Learn Your Way's multi-representation generation for producing personalized explanations at each node.

### Shared Themes

1. **Hierarchical knowledge organization**: All three papers use hierarchical structures (mind maps in #7 and #8, directed knowledge graphs in #9) as the primary organizational principle.

2. **Multiple representations of the same knowledge**: Learn Your Way generates 5 formats from one source; AutoMathKG stores definitions, proofs, and solutions for the same mathematical entity; Mindalogue provides Explanation, Examples, and Exploration views per node.

3. **LLMs as knowledge transformers**: All three use LLMs not to generate novel knowledge but to transform, restructure, augment, or personalize existing knowledge -- keeping human-authored content as the source of truth.

4. **Addressing cognitive load**: All three explicitly cite cognitive load theory. Learn Your Way reduces it via personalization and progressive disclosure; Mindalogue reduces it via nonlinear navigation and color-coded structure; AutoMathKG reduces it via semantic search over structured entities.

5. **The "comprehension gap"**: All three papers are motivated by the gap between how knowledge is stored/published (textbooks, papers, formal systems) and how humans actually learn and explore it.

### Key Gaps Identified Across Papers

1. **No integration of formal verification with natural language**: AutoMathKG's tactic labels are inspired by Lean 4 but the system does not connect to formal proof systems. A complete vision would link natural-language explanations to machine-verified proofs.

2. **No adaptive learning based on knowledge state**: None of the systems model the learner's evolving knowledge state to adapt which nodes/representations to show. Learn Your Way personalizes on static attributes (grade, interests) not dynamic understanding.

3. **Scale and domain coverage**: AutoMathKG has 13K entities; real mathematical knowledge spans millions of concepts. Scalability of these approaches to comprehensive scientific knowledge is untested.

4. **Collaborative knowledge exploration**: All three focus on individual learners. Scientific learning is often collaborative (study groups, lab discussions, peer instruction).

5. **Equation and symbol handling**: None of the papers deeply address how to render, explain, or interact with mathematical equations and scientific notation -- a critical gap for the vision of making "math equations and physics" understandable.

### Evidence Strength Comparison

| Paper | Study Type | N | Key Evidence |
|-------|-----------|---|--------------|
| Learn Your Way (#7) | RCT | 60 | p=0.03 for both immediate and retention; expert ratings >0.90 |
| Mindalogue (#8) | Within-subjects comparison | 16 | 4/5 dimensions favored; cognitive load p=0.033 |
| AutoMathKG (#9) | Benchmark evaluation | -- | Hits@q superior to 5 baselines at q>=5; 94-97% retrieval precision |

Learn Your Way provides the strongest evidence for learning efficacy (randomized controlled trial with retention testing). Mindalogue provides HCI evidence for interaction design. AutoMathKG provides system-level technical evaluation but no user study.
