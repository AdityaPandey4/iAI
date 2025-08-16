
---

# L3AGI Framework with XAgent Integration

This repository contains a modified version of the L3AGI (Team of AI Agents) framework. The original Langchain REACT agent has been replaced with the more powerful, multi-agent **[XAgent framework by OpenBMB](https://github.com/OpenBMB/XAgent)**. This integration enhances the agent's reasoning, planning, and tool-execution capabilities.

The integration also includes support for using Google's **Gemini** models via the **LiteLLM** proxy, providing a flexible and powerful alternative to OpenAI's models.

## Architecture Overview

This integrated system is composed of three core services that must run simultaneously:

1.  **L3AGI Application Server:** The main backend application that handles user interactions, manages agent configurations, and now contains our custom `XAgentProcessor` to orchestrate the agent's logic.
2.  **XAgent ToolServer:** A dedicated, sandboxed microservice required by XAgent. It securely executes all tools (e.g., web searches, code interpretation, file I/O) on behalf of the agent. **The L3AGI application will not work without this service.**
3.  **LiteLLM Proxy:** A lightweight proxy server that translates OpenAI-formatted API requests from XAgent into Google Gemini API requests. This allows us to use Gemini models seamlessly without modifying XAgent's core code.

## Prerequisites

Before you begin, ensure you have the following installed on your local machine:

*   **Git:** For cloning the repositories.
*   **Python 3.11+:** For running the application and dependencies.
*   **Docker and Docker Compose:** For running the XAgent ToolServer.
*   **Poetry:** For managing Python dependencies in the L3AGI project.
*   **API Keys:**
    *   **Google Gemini API Key:** Obtain from [Google AI Studio](https://aistudio.google.com/app/apikey).
    *   **OpenAI API Key (Optional):** If you wish to test with OpenAI models.

## 🛠️ Setup Instructions

Follow these steps carefully to set up the complete development environment.

### Step 1: Clone Repositories

First, clone both the modified L3AGI repository and the original XAgent repository into a single workspace directory.

```bash
# Create a workspace directory
mkdir my-xagent-project
cd my-xagent-project

# Clone your forked L3AGI repository
git clone https://github.com/AdityaPandey4/iAI.git
```

### Step 2: Configure Environment Variables

Create a `.env` file in the L3AGI server directory to store your API keys.

1.  Navigate to the L3AGI server directory:
    ```bash
    cd l3vels-team-of-ai-agents/apps/server/
    ```

2.  Create a copy of the example environment file:
    ```bash
    cp .env.example .env
    ```

3.  Edit the new `.env` file and add your API keys. It should look like this:

    ```env
    # .env

    # Add your Gemini API Key here
    GEMINI_API_KEY="your_google_ai_studio_api_key"

    # Add your OpenAI API Key if you plan to use it
    OPENAI_API_KEY="your_openai_api_key"

    # Other L3AGI settings (database, etc.) can be left as default for local setup
    # ...
    ```

### Step 3: Install XAgent as a Library

To allow L3AGI to import the XAgent code, you must install it in "editable" mode in your Python environment.

1.  Activate your Python virtual environment.
2.  Navigate to the cloned XAgent directory:
    ```bash
    cd ../../../XAgent/  # From the l3agi/apps/server directory
    ```
3.  Install XAgent:
    ```bash
    pip install -e .
    ```

### Step 4: Resolve L3AGI Python Dependencies

The L3AGI project uses Poetry to manage its dependencies. You need to generate an up-to-date `poetry.lock` file.

1.  Navigate back to the L3AGI server directory:
    ```bash
    cd ../l3vels-team-of-ai-agents/apps/server/
    ```
2.  Run `poetry lock` to resolve dependencies:
    ```bash
    poetry lock
    ```
3.  Install the dependencies:
    ```bash
    poetry install --no-interaction --no-ansi
    ```

## 🚀 Running the Application

You will need **three separate terminals** to run all the required services.

### Terminal 1: Start the XAgent ToolServer

This service runs in Docker and handles tool execution.

1.  Navigate to the XAgent directory:
    ```bash
    cd /path/to/your/workspace/XAgent/
    ```
2.  Start the ToolServer using Docker Compose:
    ```bash
    docker-compose up --build
    ```
3.  **Verification:** Wait for the logs to stabilize. You should see messages indicating that the `ToolServerManager` is running on port `8080` and that a `ToolServerNode` has successfully registered.

### Terminal 2: Start the LiteLLM Proxy

This service translates API calls to Gemini.

1.  Navigate to your main workspace directory.
2.  Set your Gemini API key in this terminal session:
    ```bash
    export GEMINI_API_KEY="your_google_ai_studio_api_key"
    ```
3.  Start the LiteLLM proxy, specifying the Gemini model you want to use:
    ```bash
    litellm --model gemini/gemini-1.5-pro-latest
    ```
4.  **Verification:** The proxy should start and listen on port `8000`.

### Terminal 3: Start the L3AGI Application Server

This is the main backend for our project.

1.  Navigate to the L3AGI server directory:
    ```bash
    cd /path/to/your/workspace/l3vels-team-of-ai-agents/apps/server/
    ```
2.  Start the Uvicorn server on a non-conflicting port (e.g., `7000`):
    ```bash
    uvicorn main:app --reload --port 7000
    ```
3.  **Verification:** The application server should start and listen on port `7000`.

## 🧪 Final Testing

1.  **Start the L3AGI Frontend:** If not already running, navigate to `l3vels-team-of-ai-agents/apps/ui/` and follow its instructions to start the UI (usually `npm install && npm run dev`).
2.  **Access the UI:** Open your browser and go to the L3AGI UI (e.g., `http://localhost:5173`).
3.  **Configure Your Agent:**
    *   Create a new agent or edit an existing one.
    *   In the **Model Name** field, you **must** enter the exact model string you configured LiteLLM to use: `gemini/gemini-1.5-pro-latest`.
    *   Save the agent.
4.  **Start a Chat:**
    *   Initiate a conversation with your newly configured agent.
    *   Send a complex prompt that requires tool use, for example:
      > "What is the latest news about Google's Gemini model? Please search the web and provide a summary."
5.  **Verify Success:**
    *   Watch the logs in all three terminals. You should see activity in the L3AGI server, the LiteLLM proxy, and the XAgent ToolServer.
    *   After some processing time, a complete, well-reasoned answer should appear in the chat window.

**Note** I was not able to complete test the integration of the frameworks as the poetry installation of the L3AGI was hardstuck, but this is not the end of my skills and knowledge, please do check out my other projects (on github) on LLMs and Agents and my past experiences on specifically on LLMs and Agents, I would love to connect and provide more information on my prior skills and experience.