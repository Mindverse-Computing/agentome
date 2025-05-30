# Agentome: A Spectral Multi-Agent AI System with SPX over Google ADK

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
