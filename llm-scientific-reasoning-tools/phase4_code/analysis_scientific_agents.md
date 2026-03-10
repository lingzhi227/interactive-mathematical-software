# Deep Analysis of Scientific Multi-Agent Repositories

**Date:** 2026-03-10
**Repositories analyzed:**
1. MCP-SIM (KAIST) -- Multi-agent physics simulation framework
2. DREAMS/material_agent (BattModels) -- Multi-agent DFT simulation
3. Science MCPs (Globus Labs / LBNL) -- MCP servers for science & HPC

---

## 1. MCP-SIM: Multi-Agent Physics Simulation Framework

**Repository:** https://github.com/KAIST-M4/MCP-SIM
**Authors:** Donggeun Park, Hyunbin Moon, Seunghwa Ryu (KAIST Mechanical Engineering)
**License:** MIT

### 1.1 Repository Structure

The repository has a remarkably flat layout with six Python files at the root level and no subdirectories:

```
MCP-SIM/
  README.md
  LICENSE
  input_clarifier_agent.py       # Stage 1: NL input refinement
  parsing_agent.py                # Stage 2: JSON extraction from text
  code_builder_agent.py           # Stage 3: FEniCS code generation
  simulation_executor_agent.py    # Stage 4: subprocess execution
  error_diagnosis_agent.py        # Stage 5: error analysis and auto-fix
  mechanical_insight_agent.py     # Stage 6: report generation
```

No orchestrator file, test suite, configuration files, or `requirements.txt` are present in the public repository. The system appears to rely on an unpublished orchestrator script or notebook that chains the agents sequentially.

### 1.2 Agent Architecture

MCP-SIM implements **6 specialized agents** in a strict linear pipeline:

| Agent | Class | LLM | Purpose |
|-------|-------|-----|---------|
| Input Clarifier | `InputClarifierAgent` | GPT-4o (temp=0.2) | Converts vague/multilingual NL prompts into precise FEniCS-ready specifications |
| Parser | `ParsingAgent` | GPT-4o (temp=0) | Extracts structured JSON (problem_type, PDE, BCs, mesh, materials) from clarified text |
| Code Builder | `CodeBuilderAgent` | GPT-4o (temp=0) | Generates complete executable FEniCS Python scripts from parsed JSON |
| Simulation Executor | `SimulationExecutorAgent` | None (no LLM) | Saves code to disk, runs via `subprocess.run()`, captures stdout/stderr |
| Error Diagnosis | `ErrorDiagnosisAgent` | GPT-4o (temp=0) | Analyzes errors, returns JSON with `fix_type`, `hint`, `after_code`, `confidence` |
| Mechanical Insight | `MechanicalInsightAgent` | GPT-4o (temp=0) | Generates multilingual educational reports explaining the simulation physics |

**Communication pattern:** Strictly sequential. Each agent receives the output of the previous agent as input. There is no parallel execution or feedback loops between non-adjacent agents (except the error diagnosis loop between stages 3-5).

**Self-correction loop:** The Executor and Error Diagnosis agents form a retry loop. When execution fails, the error diagnosis agent receives the error message, simulation output, and original code, then returns a corrected version. The iteration count is tracked via the `iteration` parameter.

### 1.3 Orchestration

**Framework:** Custom pipeline, not LangGraph/AutoGen/CrewAI. Each agent is a standalone class using LangChain's `PromptTemplate` + `ChatOpenAI` via `RunnableSequence` (prompt | llm chain). The orchestrator (not in repo) presumably calls:

```python
clarified = InputClarifierAgent(key).clarify(raw_input)
parsed = ParsingAgent(key).parse(clarified)
code = CodeBuilderAgent(key).build_code(parsed)
# retry loop:
output = SimulationExecutorAgent().execute_simulation()
if has_error(output):
    fix = ErrorDiagnosisAgent(key).diagnose_and_fix(error, code, output)
    code = fix["after_code"]
    # repeat execution
report = MechanicalInsightAgent(key).generate_report(code, language="Korean")
```

There is **no state graph, no conditional routing, no supervisor agent**. The pipeline is deterministic and linear.

### 1.4 Tool Integration

**Primary simulation tool: FEniCS** (finite element library for PDEs).

The Code Builder agent generates Python scripts targeting FEniCS APIs:
- Supported physics: Navier-Stokes (fluid), heat transfer, linear elasticity, hyperelasticity, phase-field fracture
- Output format: XDMF files with timestep info
- Solver preferences: MUMPS linear solver for nonlinear problems, backward Euler for time-dependent heat
- Mesh handling: Both procedural mesh generation and XML mesh file loading

The Simulation Executor runs the generated script via `subprocess.run(["python", filename])` with comprehensive error pattern matching for FEniCS-specific errors including:
- Standard Python errors (NameError, TypeError, etc.)
- FEniCS-specific: DijitsoError, UFLException, ArityMismatch, form compilation failures
- System-level: Segmentation fault, Killed, MemoryError

### 1.5 Memory/State Management

**Minimal explicit state management.** State is passed as function arguments between agents. However, the system does persist data to disk:
- `parsed_results.jsonl` and `parsed_results.txt`: Accumulated parsing results
- `error_logs.txt`: Timestamped error history with code and output
- `last_prompt.txt`: Appended prompt history for debugging
- `simulation_report.txt`: Final educational report

There is no shared memory store, no vector database, no conversation history tracking across agents.

### 1.6 Error Handling

The Error Diagnosis Agent implements structured self-correction:

1. **Error logging:** Every error is appended to `error_logs.txt` with timestamp, error message, code, and simulation output
2. **LLM diagnosis:** GPT-4o receives the full context and returns a JSON object:
   - `fix_type`: "parsing" or "code" (indicates where the fix should be applied)
   - `hint`: Human-readable explanation of the error
   - `after_code`: Complete corrected Python code
   - `confidence`: Float 0.0-1.0
3. **Fallback:** If LLM diagnosis fails, returns original code with confidence=0.0
4. **Robustness:** Handles markdown code block stripping from LLM responses

The Simulation Executor has a comprehensive regex-based error detector that catches both Python-level and FEniCS-specific compilation errors in stdout/stderr.

### 1.7 LLM Configuration

**Model:** OpenAI GPT-4o exclusively (all 5 LLM-using agents)
**Temperature:** 0.0 for code/parsing/diagnosis agents; 0.2 for input clarifier (allows slight creativity)
**API:** LangChain's `ChatOpenAI` wrapper
**Prompts:** Extensive, domain-specific system prompts embedded as `PromptTemplate` strings. The Code Builder prompt alone is ~60 lines covering FEniCS best practices, solver configuration, mesh handling, and output formatting.

### 1.8 MCP Integration

**None.** Despite the name "MCP-SIM," this repository does not implement or use Model Context Protocol servers. The "MCP" in the name likely stands for something else (possibly "Multi-Component Pipeline" or similar). The agents communicate via direct Python function calls, not via MCP tool interfaces.

### 1.9 Reproducibility

**Weak.** The repository lacks:
- `requirements.txt` or `environment.yml`
- Docker/container configuration
- The orchestrator script
- Example inputs/outputs
- Installation instructions beyond the README overview

FEniCS itself requires a non-trivial installation (typically via Docker or conda). The system hardcodes the OpenAI API key as a constructor parameter.

### 1.10 Code Quality

- **No tests** in the repository
- **Logging:** Each agent has its own logger with dedicated log files via `setup_logger()` (from a `utils.py` not in the repo)
- **Documentation:** Inline comments (some in Korean), descriptive class/method names
- **Modularity:** Good separation of concerns -- each agent is a standalone class in its own file
- **Error handling:** Try/except blocks with logging throughout, but some agents silently return the input on failure (e.g., InputClarifierAgent returns raw_input if clarification fails)

---

## 2. DREAMS/material_agent: Multi-Agent DFT Simulation

**Repository:** https://github.com/BattModels/material_agent
**Authors:** Ziqi Wang, Hongshuo Huang et al. (BattModels group)
**Paper:** arXiv:2507.14267
**License:** Not explicitly stated

### 2.1 Repository Structure

```
material_agent/
  README.md
  environment.yml              # Conda environment specification
  invoke.py                    # Main entry point with example workflows
  config/
    default.yaml               # API keys, model selection, paths
  src/
    graph.py                   # LangGraph state machine (supervisor pattern)
    planNexe2.py               # Plan-and-execute variant (primary)
    planNexeHighPlan.py         # Higher-level planning variant
    agent.py                   # Generic agent factory function
    tools.py                   # ~30 domain tools (DFT, HPC, structure)
    prompt.py                  # Agent system prompts (DFT, HPC, supervisor)
    utils.py                   # Config loading, QE parsing, convergence analysis
    myCANVAS.py                # Persistent shared state (pickle-based)
    rag.py                     # RAG pipeline (Chroma + web content)
    var.py                     # Global variable module
  bin/
    invoke.sh                  # Shell runner
    run_md.sh                  # MD runner
  figures/                     # Architecture diagrams
```

### 2.2 Agent Architecture

DREAMS implements a **3-agent hierarchical supervisor pattern** with two architectural variants:

#### Variant 1: Supervisor Graph (`graph.py`)

| Agent | Role | Tools |
|-------|------|-------|
| **Supervisor** | Routes tasks to appropriate worker agent | No tools; uses structured output for routing |
| **DFT_Agent** | Computational chemistry operations | `find_pseudopotential`, `write_script`, `calculate_lc`, `generate_convergence_test`, `generate_eos_test`, `get_kspacing_ecutwfc` |
| **HPC_Agent** | Job submission and monitoring | `submit_and_monitor_job`, `find_job_list`, `add_resource_suggestion`, `read_energy_from_output` |

Communication uses LangGraph's `StateGraph` with `AgentState` (message list + next agent name). The Supervisor uses `with_structured_output(routeResponse)` to produce routing decisions (DFT_Agent, HPC_Agent, or FINISH).

#### Variant 2: Plan-and-Execute (`planNexe2.py`) -- Primary variant

This is a more sophisticated architecture with three phases:

1. **Planner:** Creates a high-level plan as a list of `myStep` objects, each specifying a step description and the assigned agent
2. **Agent (simTeam subgraph):** Executes individual steps using the supervisor pattern from Variant 1
3. **Replanner:** After each step completion, reassesses remaining work and updates the plan

The `PlanExecute` state tracks: `input`, `plan` (list of steps), `past_steps` (completed work with results), and `response` (final output).

**Both agents share common tools** for canvas inspection and writing (`inspect_my_canvas`, `read_my_canvas`, `write_my_canvas`), plus their domain-specific tools.

### 2.3 Orchestration

**Framework:** LangGraph (from LangChain ecosystem)

Key architectural decisions:
- **Supervisor pattern:** The Supervisor agent is a `ChatPromptTemplate` + `llm.with_structured_output(routeResponse)` chain that determines which worker acts next
- **ReAct agents:** Both DFT and HPC workers are created via `create_react_agent()` from `langgraph.prebuilt`, giving them autonomous tool-use capability
- **Checkpointing:** `MemorySaver()` provides in-memory state persistence across graph executions
- **Recursion limit:** Set to 1000, reflecting the potentially long-running nature of DFT workflows
- **Streaming:** Full streaming support with `graph.stream()` for real-time output
- **Pause/resume:** External control via `status.txt` file monitoring (the graph checks this file and can pause execution)

The graph topology:
```
START --> Supervisor
Supervisor --> DFT_Agent (conditional)
Supervisor --> HPC_Agent (conditional)
Supervisor --> END (FINISH)
DFT_Agent --> Supervisor
HPC_Agent --> Supervisor
```

### 2.4 Tool Integration

**~30 tools** implemented as LangChain `@tool` decorated functions:

#### DFT/Structure Tools:
- `find_pseudopotential(element)`: Searches pseudo_dir for matching UPF files
- `init_structure_data(element, lattice, a, b, c)`: Creates bulk structures via ASE
- `generateSurface_and_getPossibleSite(...)`: Surface generation + adsorption site identification (uses `autocat`)
- `generate_myAdsorbate(...)`: Creates adsorbate molecular structures
- `add_myAdsorbate(...)`: Places adsorbates on surfaces with rotation control
- `write_QE_script_w_ASE(...)`: Generates Quantum ESPRESSO input files (23 parameters!)
- `generate_convergence_test(...)`: Creates k-point/ecutwfc convergence test grids
- `generate_eos_test(...)`: Creates equation of state calculations with 5 volume points
- `calculate_lc(...)`: Fits EOS to determine lattice constants
- `get_kspacing_ecutwfc(...)`: Analyzes convergence tests to select optimal parameters
- `get_convergence_suggestions(...)`: LLM-powered analysis of non-converging calculations
- `calculate_formation_E(...)`: Computes adsorption energies from slab/adsorbate/system energies
- `write_script(...)`, `write_LAMMPS_script(...)`: Raw script writing tools

#### HPC Tools:
- `submit_and_monitor_job(...)`: SLURM job submission via `pysqa.QueueAdapter`
- `submit_single_job(...)`: Single job submission
- `find_job_list(...)`: Queries CANVAS for pending/finished jobs
- `read_energy_from_output(...)`: Parses QE output for total energy
- `add_resource_suggestion(...)`: Stores resource allocation data in SQLite

#### Common Tools:
- `inspect_my_canvas()`, `read_my_canvas(key)`, `write_my_canvas(key, value)`: Shared state access

**Simulation backends:**
- **Quantum ESPRESSO:** Primary DFT engine, driven via ASE's `Espresso` calculator and raw input file generation
- **ASE (Atomic Simulation Environment):** Structure building, EOS fitting, I/O
- **autocat:** Surface structure generation and adsorption site identification
- **pysqa:** SLURM queue adapter for HPC job submission
- **LAMMPS:** Secondary support via `write_LAMMPS_script`

### 2.5 Memory/State Management

DREAMS implements a sophisticated shared state system:

#### CANVAS (Primary shared state)
The `myCANVAS` class provides a persistent key-value store:
- **Backed by pickle:** State persists to `canvas.pickle` after every write
- **Text mirror:** Human-readable `canvas.pickle.txt` generated alongside
- **Protected keys:** `finished_job_list` is read-only (only updated by the HPC agent)
- **Type validation:** Job lists must contain only strings
- **Overwrite protection:** Default prevents accidental overwrites unless explicitly enabled
- **Special keys:**
  - `ready_to_run_job_list`: Jobs waiting for submission
  - `finished_job_list`: Completed jobs
  - `scratch_job_list`: Intermediate/template jobs
  - `jobs_K_and_ecut`: Convergence test metadata
  - `Possible_CO_site_on_Pt_surface`: Adsorption site coordinates

#### LangGraph MemorySaver
In-memory checkpointing for conversation state, enabling the graph to resume from any checkpoint.

#### SQLite Database
`resource_suggestions.db` stores HPC resource allocation history for learning optimal configurations.

#### Dialogue History
When `SAVE_DIALOGUE=True`, full agent dialogue is saved to `{WORKING_DIR}/his.txt`.

### 2.6 Error Handling

Multi-layered error handling:

1. **Tool-level:** Most tools have try/except blocks returning descriptive error messages as strings (e.g., "Invalid input atoms directory: {path}. make sure to supply either absolute path..."). This allows the LLM agent to understand and correct errors.

2. **Convergence diagnosis:** The `get_convergence_suggestions()` tool spawns a separate Claude 3.7 Sonnet instance as a "DFT expert" to analyze non-converging calculations. It reads the input file and output file, then provides parameter-specific suggestions.

3. **Self-correction in prompts:** System prompts instruct agents to "find out possible reasons and resolve it" when results deviate from literature values. For example: "If your result is not within 10 percent of the literature, please find out possible reasons and resolve it."

4. **Job monitoring:** The HPC agent monitors SLURM jobs and can detect failed calculations from output files.

5. **Replanning:** The plan-and-execute architecture allows the system to replan after step failures.

### 2.7 LLM Configuration

**Primary model:** Claude 3.7 Sonnet (`claude-3-7-sonnet-20250219`) via `ChatAnthropic`
**Temperature:** 0.0 (deterministic)
**Alternative backends (commented out in code):**
- Azure OpenAI GPT-4o
- DeepSeek
**API key:** Stored in `config/default.yaml` (note: the actual key is committed to the repo -- a security issue)

The system uses Claude for:
- Supervisor routing decisions
- DFT agent tool selection and execution
- HPC agent job management
- Convergence analysis (inner LLM call within `get_convergence_suggestions`)
- Plan generation and replanning

### 2.8 MCP Integration

**None.** Despite the repository being named in the context of MCP research, DREAMS/material_agent does not use Model Context Protocol. Tools are implemented as native LangChain `@tool` decorated functions, not as MCP server endpoints.

### 2.9 Reproducibility

**Moderate.**
- `environment.yml` provided for conda environment setup
- `config/default.yaml` centralizes configuration
- Hardcoded paths to University of Michigan NFS (`/nfs/turbo/coe-venkvis/...`) limit portability
- Pseudopotential files and Quantum ESPRESSO installation required separately
- SLURM-specific HPC integration (job templates reference specific partitions/resources)
- API key committed to repo (security concern)

### 2.10 Code Quality

- **No tests** in the repository
- **Documentation:** Minimal -- primarily README with setup instructions
- **RAG module:** `rag.py` indexes Lilian Weng's agent blog post for reference (seems unused in main workflow)
- **Code style:** Functional but inconsistent -- variable naming mixes camelCase and snake_case, some hardcoded values
- **Known issues:**
  - Bare `except:` clauses throughout `tools.py`
  - API key in config checked into git
  - Some `time.sleep(60)` calls commented out but left in code
  - Hardcoded lattice constant `a_dict = {'Pt': 3.92}` in `generateSurface_and_getPossibleSite` overrides user input

---

## 3. Science MCPs: MCP Servers for Science & HPC

**Repository:** https://github.com/globus-labs/science-mcps
**Paper:** arXiv:2508.18489 ("Experiences with Model Context Protocol Servers for Science and High Performance Computing")
**Authors:** Haochen Pan, Ryan Chard, Reid Mello, Christopher Grams, et al. (Globus Labs / ANL / LBNL)
**License:** MIT

### 3.1 Repository Structure

```
science-mcps/
  README.md
  LICENSE
  docs/
    develop.md                         # Developer guide
    local-mcps.md                      # Local setup instructions
  mcps/
    globus/
      __init__.py
      auth.py                          # Globus OAuth2 authentication
      compute_server.py                # Remote function execution
      transfer_server.py               # File transfer between endpoints
      search_server.py                 # Document indexing and search
      schemas.py                       # Pydantic data models
      requirements.txt
      Dockerfile
      entrypoint.sh
      README.md
    compute_facilities/
      __init__.py
      facility_server.py               # NERSC + ALCF status monitoring
      schemas.py                       # HPC status data models
      requirements.txt
      Dockerfile
      entrypoint.sh
      README.md
    diaspora/
      diaspora_server.py               # Kafka event streaming
      requirements.txt
      Dockerfile
      entrypoint.sh
      README.md
    garden/
      garden-mcp.py                    # ML model inference on HPC
      requirements.txt
      README.md
  tests/
    __init__.py
    diaspora_server_test.py
    facility_server_test.py
  .github/workflows/
    deploy.yaml                        # AWS Lightsail deployment
    live_tests.yaml                    # Integration test workflow
  .pre-commit-config.yaml
  mypy.ini
  tox.ini
```

### 3.2 Architecture: Not a Multi-Agent System

**Critical distinction:** Science MCPs is fundamentally different from the other two repositories. It is **not** a multi-agent system. Instead, it provides a collection of **independent MCP servers** that make existing scientific infrastructure accessible to LLM agents (like Claude Desktop) via the Model Context Protocol standard.

The architecture follows a "thin wrapper" philosophy: each MCP server is a lightweight adapter over a mature, existing service (Globus, ALCF/NERSC status APIs, Diaspora, Garden). The servers expose their capabilities as MCP tools that any MCP-compatible LLM client can discover and invoke.

### 3.3 MCP Server Inventory

#### Server 1: Globus Compute (`compute_server.py`)
**Purpose:** Remote Python function execution on HPC resources
**Framework:** FastMCP
**Tools (5):**
- `list_my_endpoints()` -- Lists accessible Globus Compute endpoints with role filtering
- `register_python_function()` -- Registers Python functions for remote execution
- `register_shell_command()` -- Wraps shell commands as executable functions
- `submit_task()` -- Submits function execution tasks to endpoints
- `get_task_status()` -- Monitors task progress and retrieves results

**Key implementation detail:** Uses `PureSourceTextInspect` serialization strategy and explicitly disables unsafe deserialization.

#### Server 2: Globus Transfer (`transfer_server.py`)
**Purpose:** File movement between Globus-connected storage endpoints
**Framework:** FastMCP
**Tools (7):**
- `search_endpoints_and_collections()` -- Full-text endpoint search
- `list_my_endpoints_and_collections()` -- Admin-owned endpoints
- `list_endpoints_and_collections_shared_with_me()` -- Shared access endpoints
- `submit_transfer_task()` -- Initiates file transfers between collections
- `get_task_events()` -- Transfer progress monitoring
- `list_directory()` -- Remote directory listing

**Key implementation detail:** Automatic consent handling -- when a 403 ConsentRequired error occurs, the `handle_gare()` function automatically requests the necessary scopes and retries.

#### Server 3: Globus Search (`search_server.py`)
**Purpose:** Scientific data indexing and retrieval
**Framework:** FastMCP
**Tools (11):**
- Index management: `create_index`, `list_my_indices`, `get_index_info`, `delete_index`
- Document operations: `ingest_document`, `ingest_documents`, `get_ingestion_status`, `delete_subject`
- Search: `search_index`, `advanced_search`, `get_subject`

#### Server 4: Compute Facilities (`facility_server.py`)
**Purpose:** HPC system status monitoring for NERSC and ALCF
**Framework:** FastMCP (async)
**Tools (4):**
- `get_nersc_status()` -- All NERSC systems status
- `get_nersc_system_status(system_name)` -- Individual system query (case-insensitive)
- `get_alcf_status()` -- ALCF operational summary
- `get_alcf_jobs(kind, resource, n, skip)` -- Paginated job listings

**Resources (3):** URI-based access patterns (`nersc://status/systems`, `alcf://polaris/status`, `alcf://aurora/status`)

#### Server 5: Diaspora Event Fabric (`diaspora_server.py`)
**Purpose:** Kafka-based scientific event streaming
**Framework:** FastMCP
**Tools (8):**
- Authentication: `diaspora_authenticate`, `complete_diaspora_auth`, `diaspora_confidential_auth`, `logout`
- Operations: `list_topics`, `register_topic`, `unregister_topic`, `produce_one`, `consume_latest`

**Key implementation detail:** Uses MSK IAM authentication with role-based token generation. The `@require_login` decorator protects operational tools.

#### Server 6: Garden (`garden-mcp.py`)
**Purpose:** ML model inference for materials science on HPC
**Framework:** FastMCP
**Tools (5):**
- `get_functions(doi)` -- Lists available functions in a Garden by DOI
- `run_function(doi, func_name, func_args)` -- Executes Garden functions
- `submit_relaxation_job(xyz_file_path, model)` -- Submits structure relaxation using MLIP models (default: mace-mp-0) to the Edith cluster
- `check_job_status(job_id)` -- Job status monitoring
- `get_job_results(job_id, output_file_path)` -- Result retrieval

**Key implementation detail:** Uses a specific Globus Compute endpoint ID for the Edith cluster. The `_mlip_garden` singleton pattern maintains Garden client state across tool calls.

### 3.4 Authentication

Centralized in `auth.py`:
- **Client credentials flow:** Uses `GLOBUS_CLIENT_ID` + `GLOBUS_CLIENT_SECRET` environment variables for service accounts
- **User flow:** Falls back to local-server OAuth2 flow with a default client ID
- **Diaspora-specific:** Separate AWS IAM + Kafka MSK authentication

### 3.5 Data Models

Comprehensive Pydantic models in `schemas.py` files:

**Globus schemas:** `ComputeEndpoint`, `ComputeTask`, `TransferEndpoint`, `TransferEvent`, `TransferFile`, `SearchIndex`, `SearchResult`, `SearchEntry`, `SearchSubject` (and more)

**Facility schemas:** `NERSCSystem`, `NERSCApiResponse`, `ALCFJob`, `ALCFStatusResponse`, `ALCFJobsResponse`, `MaintenanceInfo`, `JobCounts`

All models use descriptive field annotations via Pydantic `Field(description=...)`, which propagates to MCP tool descriptions.

### 3.6 Error Handling

Consistent pattern across all servers:
- All API calls wrapped in try/except for `globus_sdk.GlobusAPIError`
- Errors converted to `ToolError` (from `fastmcp.exceptions`) for proper MCP error reporting
- The transfer server handles consent escalation automatically
- Path traversal guards in facility server resource names
- Input validation via Pydantic models

### 3.7 Deployment & Reproducibility

**Local deployment:**
- Python 3.11 + conda
- Per-server `requirements.txt` files
- Configuration via Claude Desktop's MCP config file (`claude_desktop_config.json`)
- Each server runs via stdio transport locally

**Cloud deployment:**
- **Dockerized:** Each server has a Dockerfile based on `continuumio/miniconda3`
- **AWS Lightsail:** GitHub Actions workflow deploys 5 services (alcf, nersc, diaspora, compute, transfer) to AWS Lightsail containers
- **Port 8000:** All HTTP-served instances use port 8000
- **CI/CD:** Manual workflow dispatch for deployment; separate live test workflow

### 3.8 Testing

**Tests present** (unlike the other two repos):
- `facility_server_test.py`: Comprehensive async test suite covering:
  - NERSC status tools (valid/invalid systems, case sensitivity)
  - ALCF job queries (parameterized across job kinds and resources)
  - ALCF status validation
  - Resource URI reading
- `diaspora_server_test.py`: Tests for Diaspora event fabric
- Tests use `fastmcp.Client` context managers
- Live test workflow in GitHub Actions

**Quality tooling:**
- `mypy.ini`: Type checking configuration
- `.pre-commit-config.yaml`: Pre-commit hooks
- `tox.ini`: Test automation

### 3.9 Code Quality

**High quality** relative to the other two repositories:
- Clean separation of concerns (one server per service)
- Consistent error handling patterns
- Comprehensive Pydantic model validation
- Type hints throughout
- Proper logging setup
- No hardcoded secrets
- Test coverage for critical functionality
- CI/CD pipeline
- Pre-commit hooks and linting

---

## Comparative Analysis

### Architecture Comparison

| Dimension | MCP-SIM | DREAMS/material_agent | Science MCPs |
|-----------|---------|----------------------|--------------|
| **Type** | Linear pipeline | Hierarchical multi-agent | MCP server collection |
| **Agents** | 6 (sequential) | 3 (supervisor + 2 workers) | 0 (infrastructure servers) |
| **Framework** | Custom (LangChain prompts) | LangGraph | FastMCP |
| **Orchestration** | Sequential pipeline | Supervisor routing + plan-and-execute | N/A (client-driven) |
| **LLM** | GPT-4o | Claude 3.7 Sonnet | None (servers are tool providers) |
| **Simulation tools** | FEniCS | Quantum ESPRESSO, ASE, LAMMPS | Globus Compute (remote arbitrary code) |
| **Domain** | Continuum physics | Computational materials science | General scientific computing |

### Tool Integration Depth

| Aspect | MCP-SIM | DREAMS | Science MCPs |
|--------|---------|--------|--------------|
| **Tool count** | 0 (code generation) | ~30 native tools | ~40 MCP tools |
| **Tool type** | LLM generates code | Pre-built Python functions | API adapters |
| **Execution model** | Generate-then-run | Agent selects and calls tools | Client invokes MCP tools |
| **HPC integration** | None | SLURM via pysqa | Globus Compute + facility status |
| **Error recovery** | LLM diagnoses and fixes code | Agent retries + convergence advisor | ToolError propagation |

### State Management

| Aspect | MCP-SIM | DREAMS | Science MCPs |
|--------|---------|--------|--------------|
| **Mechanism** | Function args + files | CANVAS (pickle) + MemorySaver + SQLite | Stateless (per-request) |
| **Persistence** | Error logs, parsed JSONs | Pickle serialization + dialogue history | None (client responsibility) |
| **Shared state** | None (sequential) | CANVAS accessible by all agents | N/A |

### MCP Protocol Usage

| Repo | Uses MCP? | Details |
|------|-----------|---------|
| MCP-SIM | **No** | Despite the name, uses direct Python function calls |
| DREAMS | **No** | Uses LangChain @tool decorators, not MCP |
| Science MCPs | **Yes** | Core purpose -- all servers implement MCP via FastMCP |

### Reproducibility & Engineering Quality

| Dimension | MCP-SIM | DREAMS | Science MCPs |
|-----------|---------|--------|--------------|
| **Tests** | None | None | Yes (pytest + live tests) |
| **Docker** | No | No | Yes (per-server Dockerfiles) |
| **CI/CD** | No | No | Yes (GitHub Actions) |
| **Dependencies declared** | No | Yes (environment.yml) | Yes (per-server requirements.txt) |
| **Secrets management** | Constructor param | Committed to YAML | Environment variables |
| **Type checking** | No | No | Yes (mypy) |
| **Linting** | No | No | Yes (pre-commit) |

---

## Key Insights

### 1. Code Generation vs. Tool Selection

MCP-SIM takes a **code generation** approach: the LLM writes complete FEniCS scripts from scratch. DREAMS takes a **tool selection** approach: the LLM selects from pre-built tools that handle the computational details. The code generation approach is more flexible (any FEniCS problem) but less reliable. The tool selection approach is more constrained but more predictable.

### 2. The Supervisor Pattern vs. Linear Pipeline

DREAMS' supervisor pattern allows dynamic routing -- the system can adaptively decide which agent handles each step. MCP-SIM's linear pipeline is simpler but cannot handle cases where the output of one stage needs to influence an earlier stage (except the error-correction loop).

### 3. Thin MCP Wrappers vs. Monolithic Agents

Science MCPs represents a fundamentally different philosophy: instead of building monolithic agent systems, it builds thin, composable tool servers that any MCP client can use. This separation of concerns (tools vs. agent logic) is more modular and reusable, but requires the client to handle orchestration.

### 4. Shared State as Critical Infrastructure

DREAMS' CANVAS system is the most interesting state management approach. By providing a shared, persistent key-value store accessible by all agents, it enables inter-agent coordination (e.g., DFT agent writes job list, HPC agent reads and submits). The overwrite protection and type validation prevent common multi-agent state corruption issues.

### 5. Domain-Specific Error Recovery

All three systems handle errors differently:
- **MCP-SIM:** LLM analyzes and rewrites failed code (most general, least reliable)
- **DREAMS:** Spawns a specialist LLM instance to diagnose DFT convergence issues, with domain-specific constraints ("never lower accuracy")
- **Science MCPs:** Propagates errors to the client via ToolError (most robust, least autonomous)

### 6. HPC Integration Maturity

DREAMS has the deepest HPC integration (direct SLURM submission via pysqa, convergence testing workflows, resource optimization). Science MCPs provides more general HPC access via Globus Compute (remote function execution on any endpoint) and facility status monitoring. MCP-SIM has no HPC integration.

---

## Recommendations for Building Scientific Agent Systems

Based on this analysis:

1. **Use MCP for tool interfaces** -- Science MCPs demonstrates the value of standardized, discoverable tool interfaces. Future systems like DREAMS should expose their tools as MCP servers rather than hardcoded LangChain tools.

2. **Combine code generation with tool selection** -- MCP-SIM's code generation is powerful for novel problems; DREAMS' tool selection is reliable for known workflows. An ideal system would use pre-built tools for standard operations and fall back to code generation for novel cases.

3. **Invest in shared state** -- DREAMS' CANVAS pattern is essential for multi-step scientific workflows where results from early steps inform later decisions.

4. **Domain-specific error handling** -- The DFT convergence advisor in DREAMS (spawning a specialist LLM) is a pattern worth generalizing: domain-specific error analysis produces much better corrections than generic "fix this code" prompts.

5. **Prioritize reproducibility** -- Science MCPs' Docker + CI/CD approach should be standard. Scientific agent systems that cannot be reliably deployed undermine the trust needed for computational science.
