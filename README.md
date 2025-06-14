# ADK A2A Notion-ElevenLabs Integration

This project demonstrates a multi-agent system using Google's Agent Development Kit (ADK) that integrates Notion for information retrieval and ElevenLabs for text-to-speech conversion.

The system demonstrates a fully decoupled Agent-to-Agent (A2A) architecture.

Please use the accompanying the [YouTube Crash Course video](https://www.youtube.com/watch?v=s6-Ofu-uu2k).

> **Note**: This repository contains the starting code to follow along with the tutorial series. For the complete implementation, please check out the `lesson-complete` branch.

## 🔨 Builder Pack

For those looking to dive deeper into ADK development, I've created an optional Builder Pack that includes:

- Complete source code for host agent and worker agents
- Cheat sheets for ADK development and A2A patterns
- .cursor/rules for consistent prompt engineering
- Comprehensive pytest test suite
- Full Lesson Plan

You can get the Builder Pack at [chongdashu.gumroad.com/l/adk-builder-pack-1](https://chongdashu.gumroad.com/l/adk-builder-pack-1).

## ☕ Support the Project

If you find this tutorial series and codebase helpful in your AI agent development journey, consider buying me a coffee! Your support helps me create more educational content on AI and agent development.

<a href="https://buymeacoffee.com/aioriented" target="_blank">
  <img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" >
</a>

## Architecture

The system consists of:

- **ElevenLabsAgent (A2A Service)**: A worker agent for text-to-speech conversion.
- **NotionAgent (A2A Service)**: A worker agent for information retrieval from Notion.
- **HostAgent (A2A Service)**: A coordinator agent that orchestrates the Notion and ElevenLabs agents.
- **Streamlit UIs**: Two separate user interfaces are provided to demonstrate different architectural patterns:
  - **Embedded Runner UI**: The UI runs the HostAgent in the same process.
  - **A2A Client UI**: The UI communicates with the HostAgent as a separate, decoupled service.

Refer to `LESSON.md` for a detailed architectural breakdown and design decisions.

## Prerequisites

- Python 3.13+
- Notion account and API key
- ElevenLabs account and API key
- `uv` package manager (recommended) or pip

## Project Structure

```
adk-a2a-mcp/
├── host_agent/             # Coordinator Agent
│   ├── agent.py            # Agent definition (with delegate_task tool)
│   ├── agent_executor.py   # Handles session management (context_id -> session_id)
│   ├── prompt.py           # Orchestration instructions
│   ├── remote_connections.py # A2A client logic
│   ├── tools.py            # ADK Tools for A2A delegation
│   ├── test_client.py      # Standalone test client
│   └── __main__.py         # A2A service entry point
├── notion_agent/           # Worker Agent
│   ├── agent.py
│   ├── agent_executor.py
│   ├── prompt.py
│   ├── test_client.py
│   └── __main__.py
├── elevenlabs_agent/       # Worker Agent
│   ├── agent.py
│   ├── agent_executor.py
│   ├── prompt.py
│   ├── test_client.py
│   └── __main__.py
├── ui/                     # Streamlit UIs
│   ├── app.py              # Embedded Runner UI
│   └── a2a_app.py          # Decoupled A2A Client UI
├── scripts/
│   └── start_agents.py     # Convenience script to launch all agent services
├── tests/                  # Pytest integration tests
│   ├── test_host_agent.py
│   ├── test_notion_agent.py
│   └── test_elevenlabs_agent.py
├── logs/                   # Agent logs are stored here
├── .env.example            # Example environment variables
├── pyproject.toml          # Project config and dependencies (single source of truth)
└── README.md               # This file
```

## Setup

1.  Clone the repository:

    ```bash
    git clone <repository-url>
    cd adk-a2a-mcp
    ```

2.  Create and activate a virtual environment:

    ```bash
    python -m venv .venv
    source .venv/bin/activate  # On Windows: .venv\Scripts\activate
    ```

3.  Install dependencies from the single source of truth, `pyproject.toml`:

    ```bash
    uv pip install -e ".[dev]"
    ```

4.  Create a `.env` file from the example and add your API keys:

    ```bash
    cp .env.example .env
    # Now edit .env and add your keys
    ```

    The `.env` file should contain:

    ```env
    NOTION_API_KEY="your_notion_api_key_here"
    ELEVENLABS_API_KEY="your_elevenlabs_api_key_here"
    ANTHROPIC_API_KEY="your_anthropic_api_key_here" # Required for some agents

    # A2A Service URLs (defaults)
    HOST_AGENT_A2A_URL="http://localhost:8001"
    NOTION_AGENT_A2A_URL="http://localhost:8002"
    ELEVENLABS_AGENT_A2A_URL="http://localhost:8003"
    ```

## Running the Full System

This is the recommended way to run the entire multi-agent system.

1.  **Start all agent services**:
    A convenience script is provided to launch the Host, Notion, and ElevenLabs agents in the background. Their logs will be saved to the `logs/` directory.

    ```bash
    python scripts/start_agents.py
    ```

    You can check the status of the agents with `ps aux | grep _agent`.

2.  **Run a UI Application**:
    Choose one of the two UIs to interact with the system.

    - **For the Decoupled A2A UI (Recommended for testing the full architecture):**
      ```bash
      streamlit run ui/a2a_app.py --server.port 8080
      ```
    - **For the Embedded Runner UI:**
      ```bash
      streamlit run ui/app.py --server.port 8080
      ```

3.  **Stopping the services**:
    You can stop the background agent processes with:
    ```bash
    pkill -f "_agent"
    ```

## Testing Individual Agents (Standalone)

For development and debugging, it's crucial to test each agent independently. Each agent has a `test_client.py` that communicates directly with its A2A service.

1.  **Start the target agent's service**. For example, to test the Notion agent:

    ```bash
    python -m notion_agent --port 8002
    ```

2.  **In a separate terminal, run its test client**:

    ```bash
    python notion_agent/test_client.py
    ```

    Follow the same pattern for the other agents:

    - `python -m elevenlabs_agent --port 8003` -> `python elevenlabs_agent/test_client.py`
    - `python -m host_agent --port 8001` -> `python host_agent/test_client.py`

## Automated Integration Testing (pytest)

The `tests/` directory contains integration tests that automate the testing process using `pytest`. Each test file (e.g., `tests/test_notion_agent.py`) is responsible for:

1.  Automatically starting the required agent's A2A service in a subprocess.
2.  Executing the logic from the corresponding `test_client.py` against the running service.
3.  Asserting that the communication was successful and the agent behaved as expected.

To run all integration tests:

```bash
pytest -v
```

To run tests for a specific agent:

```bash
pytest -v tests/test_notion_agent.py
```

## Development Workflow

- **Code Formatting**: `black .` && `isort .`
- **Type Checking**: `mypy .`
- **Testing**: `pytest`

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

This software is provided "as is", without warranty of any kind, express or implied.
