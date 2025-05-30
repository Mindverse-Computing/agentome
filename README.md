
# Agentome: A Multi-Agent AI System with Syncronization Protocol for Exchange (SPX) implementing Google ADK

**Agentome** is a simulation framework for exploring multi-agent AI systems where agent activation and interaction are driven by the spectral dynamics (eigenvalues and eigenvectors) of a time-evolving complex Hermitian graph. It leverages concepts from Google's Agent Development Kit (ADK) for agent structure, tool usage (via MCP), and inter-agent communication (via A2A), and implements a custom Synchronization Protocol for Exchange (SPX) for emergent coordinated behaviors.

The system is designed to model scenarios where underlying systemic harmony, dissonance, or specific spectral patterns can trigger intelligent agent responses, applicable to domains like financial markets, neuroscience, or complex sensor networks.

## Core Concepts

* **Dynamic Hermitian Graph**: The system's foundation is a graph with `N` nodes where edge weights are complex numbers and evolve over time ($H_t$). This represents the changing relationships and influences within the system.
* **Eigenflow**: At each timestep, the eigendecomposition of $H_t$ yields eigenvalues ($\lambda_k^t$) and eigenvectors ($\psi_k^t$). The temporal evolution of this spectral data is termed the "eigenflow."
* **System State Tensor (SST)**: A representation of the system's state derived from the eigenflow. In this project, $\text{SST}_t(i,k) = \psi_k^t(i)$ captures the projection of node `i` onto eigenmode `k` at time `t`.
* **Spectral Agents**: AI agents that perceive and react to the eigenflow or the SST. They are categorized into:
    * **Indicator Agents**: Monitor global spectral properties derived from eigenvalues (e.g., dominant eigenvalue, spectral gap, eigenvalue variance).
    * **Projection Agents**: Analyze the SST directly, looking for patterns like harmonic resonance, phase coherence across nodes for specific modes, or temporal wave patterns in node-mode components.
* **Google ADK Integration**:
    * Agents inherit from `google.adk.agents.Agent`.
    * **MCP (Model Context Protocol)**: Agents use ADK Tools (hosted on an MCP server) to access historical spectral data (eigenflow, SST, SPX pulses) from memory modules.
    * **A2A (Agent-to-Agent Communication)**: Agents emit SPX `SynchPulses` via A2A messages (simulated in the current version) when their activation criteria are met. These pulses can be routed to other agents or a central SPX handler.
* **SPX (Synchronization Protocol for Exchange)**:
    * **`SynchPulse`**: A structured message emitted by an agent upon spectral resonance or indicator threshold-crossing. It contains details about the triggering condition (agent, modes, amplitude, phase).
    * **SPX Emitter/Handler**: Mechanisms for agents to send `SynchPulses` and for other agents (or a router) to receive and process these pulses.

## Project Architecture

1.  **Graph Dynamics**: `HermitianGraph` generates $H_t$ at each timestep.
2.  **Spectral Analysis**: `EigenFlow` computes $(\lambda_k^t, \psi_k^t)$ from $H_t$.
3.  **State Representation**: `SpectralTensor` constructs $\text{SST}_t$ from $\psi_k^t$.
4.  **Memory Logging**: `EigenLog`, `SSTLog`, `PulseLog` store the history of these components.
5.  **ADK Tools & MCP Server**:
    * `adk_tools/*_history_tool.py` define ADK-compatible tools that wrap the memory logs.
    * `services/mcp_tool_server.py` runs a `FastMCP` server to host these tools, making them accessible via HTTP.
6.  **Agent Instantiation & Configuration**:
    * Agents (`agents/indicator_agents/*`, `agents/projection_agents/*`) inherit from `agents/base_spectral_agent.py` (which itself inherits from `google.adk.agents.Agent`).
    * Configurations for each agent instance are loaded from YAML files in `config/agent_configs/`.
    * Agents are initialized with an ADK `Toolset` (via an `MCPClient` pointing to the MCP server) and an A2A send callback.
7.  **Simulation Loop (`main_simulation.py`)**:
    * Orchestrates timesteps.
    * Updates graph, eigenflow, SST, and logs them.
    * Triggers each agent's `process_timestep()` method.
    * Agents use their ADK tools to query historical data from the logs (via the MCP server).
    * Activated agents generate `SynchPulse` objects.
    * These pulses are logged locally and "sent" via a simulated A2A bus to a target (e.g., a central `SPXMessageHandler` or other agents).
8.  **SPX Communication**:
    * `spx/pulse.py` defines the `SynchPulse` structure.
    * `spx/spx_emitter.py` (used by `BaseSpectralAgent`) formats and sends pulses via the A2A callback.
    * `spx/spx_handler.py` provides logic for parsing incoming A2A messages as SPX pulses and routing them to registered callbacks within an agent.

## Directory Structure

```
agentome/
│
├── graph/              # Hermitian graph generator & eigenflow computation
├── sst/                # System State Tensor construction
├── memory/             # Data logging (eigenflow, SST, SPX pulses)
├── adk_tools/          # ADK-compatible tools for accessing memory logs
├── agents/             # Spectral agent implementations (base, indicator, projection)
│   ├── base_spectral_agent.py
│   ├── indicator_agents/
│   └── projection_agents/
├── spx/                # SPX pulse definition, emitter, and handler logic
├── config/             # Configuration files
│   ├── spx_config.py
│   ├── tool_server_config.py
│   └── agent_configs/    # YAML configurations for individual agent instances
├── services/           # Server implementations
│   ├── mcp_tool_server.py # Hosts ADK tools via MCP
│   └── agent_a2a_service.py # Conceptual: Hosts agents with A2A endpoints (FastAPI example)
├── notebooks/          # Jupyter notebooks for demos, visualization, analysis
│   └── demo_adk_spx.ipynb (to be created)
├── main_simulation.py  # Main script to run the simulation
├── requirements.txt    # Python dependencies
└── README.md           # This file
```

## Setup and Installation

1.  **Python**: Ensure you have Python 3.8 or newer installed.
2.  **Virtual Environment** (recommended):
    ```bash
    python -m venv .venv
    source .venv/bin/activate  # On Windows: .venv\Scripts\activate
    ```
3.  **Install Dependencies**:
    ```bash
    pip install -r requirements.txt
    ```

## Running the Simulation

The simulation involves two main components that ideally run as separate processes: the MCP Tool Server and the Main Simulation script. The optional Agent A2A Service could also be run if you are simulating agents that need to be independently addressable for incoming A2A messages not originating from the main simulation's pulse emissions.

1.  **Start the MCP Tool Server**:
    This server makes the ADK tools (which access `EigenLog`, `SSTLog`, `PulseLog`) available to agents via MCP (HTTP).
    Open a terminal in the `agentome` root directory and run:
    ```bash
    python -m agentome.services.mcp_tool_server
    ```
    By default, it will start on `http://localhost:8001`. The simulation's log instances (`eigen_log`, `sst_log`, `pulse_log`) are instantiated globally in `mcp_tool_server.py` and passed to the tools it serves. For the `main_simulation.py` to interact correctly when its agents use tools, these log instances **must be the same ones** the simulation loop populates. This requires that either:
    a.  The logs are persistent (e.g., file-based, DB-based - **not implemented in current version**), OR
    b.  For a simplified single-machine demo where both processes share memory conceptually (not truly for separate processes without IPC), the tools in the server are configured to use the *exact same Python objects*. The current `mcp_tool_server.py` instantiates its own logs. *This needs reconciliation for a true multi-process setup (e.g., using files/DB for logs). For a pure single-script test without a separate server process, `main_simulation.py`'s tool initialization directly accesses its own logs.* The current `main_simulation.py` assumes the server is separate and *itself* populates distinct log instances that the agents (via the external server) would read. This is the part that requires careful setup for data consistency.

    **Simplification for Demo**: If you are running a simplified demo where the `main_simulation.py` handles everything conceptually (and ADK tool calls are mocked or use globally set log instances within the same process *without* a networked MCP server), you might not run `mcp_tool_server.py` separately. However, the `main_simulation.py` is currently written to expect agents to connect to an MCP server specified in `config/tool_server_config.py`. The note about "Conceptual: ADK Tools are now pointing to the simulation's log instances" in `main_simulation.py`'s `run_in_process_mcp_server` function (which isn't actually run as a server task in the final `main_simulation.py`) highlights this conceptual shared memory model for tools.

2.  **Start the Agent A2A Service (Optional)**:
    If you have agents that need to be hosted as persistent services with their own A2A endpoints for receiving messages from *external* sources (not just the simulated A2A bus in `main_simulation.py`), run:
    ```bash
    python -m agentome.services.agent_a2a_service
    ```
    This starts a FastAPI server (default: `http://localhost:8002`) that can receive A2A messages for agents it hosts. These agents will also attempt to connect to the MCP Tool Server for their tools.

3.  **Run the Main Simulation Script**:
    This script orchestrates the graph dynamics, spectral analysis, agent evaluations, and SPX pulse emissions using a simulated A2A bus.
    Open another terminal in the `agentome` root directory and run:
    ```bash
    python agentome/main_simulation.py
    ```
    The script will log its progress, including timestep completions and any SPX pulses emitted by agents. It expects the MCP Tool Server to be running at the configured address (`http://localhost:8001` by default) so that agents can initialize their `Toolset` and use ADK tools.

## Configuration

* **SPX Protocol**: Global SPX settings (e.g., default A2A message type for pulses) are in `config/spx_config.py`.
* **MCP Tool Server**: Network and metadata settings for the `mcp_tool_server.py` are in `config/tool_server_config.py`.
* **Agent Instances**: Specific parameters for each agent instance (e.g., `agent_id`, thresholds, `basis_indices`, target A2A router) are defined in individual YAML files within the `config/agent_configs/` directory. The `agent_configs/__init__.py` provides helpers to load these. Ensure your YAML configurations correctly reference agent class names or follow a naming convention that `main_simulation.py` can infer for instantiation. Add an `agent_class: ClassName` field to your agent YAMLs for explicit mapping.

## Extending the System

* **New Agents**:
    1.  Create a new agent class inheriting from `BaseSpectralAgent` in the `agents/indicator_agents/` or `agents/projection_agents/` directory.
    2.  Implement the `async def evaluate_spectral_condition(...)` method with its unique logic.
    3.  Add its configuration to a new YAML file in `config/agent_configs/`.
    4.  Import and add the new agent class to the `agent_class_map` in `main_simulation.py` and relevant `__init__.py` files.
* **New ADK Tools**:
    1.  Define new tool functions/classes in `adk_tools/` using `google.adk.tools.Tool`.
    2.  Add them to the `all_adk_tools` list in `adk_tools/__init__.py`. The MCP server will automatically pick them up.
* **Graph Dynamics**: Modify `graph/hermitian_graph.py` to implement different time-evolution rules for the system's adjacency matrix.

