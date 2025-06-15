# InstaVibe: AI-Powered Social Event Planning

## Project Overview

InstaVibe is a popular social event platform. While successful, the process of planning group activities can be a significant hurdle for some users. It involves understanding friends' preferences, searching for suitable events or venues, and coordinating the details, which can be time-consuming and complex.

This project introduces AI-powered intelligent agents to revolutionize social planning on InstaVibe. These agents are designed to:
*   Understand user and friend preferences through 'social listening'.
*   Proactively suggest tailored and relevant activities.
*   Handle the heavy lifting of planning and coordination.

The goal is to make social planning on InstaVibe a seamless, delightful, and engaging experience for all users. This repository contains the codebase for building and deploying this multi-agent system.

## Architecture

InstaVibe's AI-powered social planning features are built upon a robust architecture leveraging Google Cloud Platform services, specialized agent frameworks, and advanced language models.

### Google Cloud Platform (GCP) Services

*   **Vertex AI:**
    *   **Gemini Models:** Powers the reasoning and decision-making capabilities of the agents.
    *   **Vertex AI Agent Engine:** A managed service for deploying, hosting, and scaling the orchestrator agent.
*   **Cloud Run:** A serverless platform used to:
    *   Host the main InstaVibe web application.
    *   Deploy individual A2A-enabled agents (Planner, Social Profiling, Platform Interaction) as independent microservices.
    *   Run the MCP Tool Server, making InstaVibe's internal APIs available to agents.
*   **Spanner:** A fully managed, globally distributed, and strongly consistent relational database, used here with its Graph Database capabilities to:
    *   Model and store complex social relationships (users, friendships, event attendance, posts).
    *   Enable efficient querying of these relationships for social profiling.
*   **Artifact Registry:** A fully managed service for storing, managing, and securing container images.
*   **Cloud Build:** Executes builds on Google Cloud, used to automatically build Docker container images from agent and application source code.
*   **Cloud Storage:** Used by services like Cloud Build for storing build artifacts and by Agent Engine for its operational needs.

### Core Agent Frameworks & Protocols

*   **Google's Agent Development Kit (ADK):** The primary framework for:
    *   Defining the core logic, behavior, and instruction sets for individual intelligent agents.
    *   Managing agent lifecycles, state, and memory.
    *   Integrating tools (like Google Search or custom-built tools).
    *   Orchestrating multi-agent workflows.
*   **Agent-to-Agent (A2A) Communication Protocol:** An open standard enabling:
    *   Direct, standardized communication and collaboration between different AI agents.
    *   Agents to discover each other's capabilities (via Agent Cards) and delegate tasks.
*   **Model Context Protocol (MCP):** An open standard that allows agents to:
    *   Connect with and utilize external tools, data sources, and systems in a standardized way.
    *   The Platform Interaction Agent uses an MCP client to communicate with an MCP server, which exposes tools to interact with the InstaVibe platform's APIs.

### Language Models (LLMs)

*   **Google's Gemini Models (e.g., gemini-2.0-flash):** These models are chosen for:
    *   Advanced Reasoning & Instruction Following.
    *   Tool Use (Function Calling): Enabling agents to determine when and how to use tools provided via ADK.
    *   Efficiency (Flash Models): Offering a balance of performance and cost-effectiveness for interactive agent tasks.

## Features

InstaVibe, enhanced by AI-driven agents, offers a range of features to simplify social planning:

*   **AI-Powered Event & Activity Suggestions:** Intelligent agents proactively suggest events and activities tailored to user and group preferences.
*   **Social Listening:** Agents analyze user connections, interactions, and preferences (with user consent) to understand interests and suggest suitable activities.
*   **Automated Planning:** Agents handle aspects of planning, such as finding venues, checking availability, and even drafting event invitations.
*   **Introvert Ally Feature:** A specialized interface to help users, particularly those who find social planning challenging, to organize and participate in social events.
*   **Multi-Agent Collaboration:** A system of specialized agents (Planner, Social Profiling, Platform Interaction) work together, orchestrated by a central agent, to fulfill complex user requests.
*   **Dynamic Tool Usage:** Agents can use tools like Google Search (for real-time information) and custom tools (via MCP for platform interaction) to perform tasks.
*   **Graph Database for Social Connections:** Leverages Spanner's graph database capabilities to model and query complex social relationships, enhancing the relevance of suggestions.
*   **Interactive Planning Process:** Users can interact with agents to refine plans and provide preferences.

## Getting Started

This section guides you through setting up the InstaVibe project in your Google Cloud environment.

### Prerequisites

*   A Google Cloud Project.
*   Access to Google Cloud Shell:
    *   Activate Cloud Shell from the Google Cloud console.
    *   Open the Cloud Shell Code Editor.
    *   Sign in with Cloud Code and select your Google Cloud Project.
    *   Ensure you are authenticated (`gcloud auth list`) and your project is set (`gcloud config set project YOUR_PROJECT_ID`).

### Cloning the Repository

Clone the `instavibe-bootstrap` project from GitHub into your Cloud Shell environment:

```bash
git clone https://github.com/weimeilin79/instavibe-bootstrap.git
cd instavibe-bootstrap
chmod +x init.sh
chmod +x set_env.sh
```

### Initialization

Run the initialization script. This script will prompt you for your Google Cloud Project ID.

```bash
./init.sh
```

After the script completes, ensure your gcloud config is set to the correct project:

```bash
gcloud config set project $(cat ~/project_id.txt) --quiet
```

### Enabling Google Cloud APIs

Enable the necessary Google Cloud APIs for the project:

```bash
gcloud services enable \
    run.googleapis.com \
    cloudfunctions.googleapis.com \
    cloudbuild.googleapis.com \
    artifactregistry.googleapis.com \
    spanner.googleapis.com \
    apikeys.googleapis.com \
    iam.googleapis.com \
    compute.googleapis.com \
    aiplatform.googleapis.com \
    cloudresourcemanager.googleapis.com \
    maps-backend.googleapis.com
```

### Environment Variables

Set up the required environment variables. The `set_env.sh` script can help, but ensure these are correctly configured for your session:

```bash
export PROJECT_ID=$(gcloud config get project)
export PROJECT_NUMBER=$(gcloud projects describe ${PROJECT_ID} --format="value(projectNumber)")
export SERVICE_ACCOUNT_NAME=$(gcloud compute project-info describe --format="value(defaultServiceAccount)")
export SPANNER_INSTANCE_ID="instavibe-graph-instance"
export SPANNER_DATABASE_ID="graphdb"
export GOOGLE_CLOUD_PROJECT=$(gcloud config get project)
export GOOGLE_GENAI_USE_VERTEXAI=TRUE
export GOOGLE_CLOUD_LOCATION="us-central1" # Or your preferred region if changed
# Note: The REPO_NAME for Artifact Registry will be set in a later step.
```
It's recommended to run `. ./set_env.sh` after cloning and initializing to ensure all variables are sourced.

### IAM Permissions

Grant the necessary IAM roles to your default compute service account:

```bash
# Spanner Admin & Database User
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT_NAME" \
  --role="roles/spanner.admin"
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT_NAME" \
  --role="roles/spanner.databaseUser"

# Artifact Registry Admin
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT_NAME" \
  --role="roles/artifactregistry.admin"

# Cloud Build Editor
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT_NAME" \
  --role="roles/cloudbuild.builds.editor"

# Cloud Run Admin
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT_NAME" \
  --role="roles/run.admin"

# IAM Service Account User
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT_NAME" \
  --role="roles/iam.serviceAccountUser"

# Vertex AI User
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT_NAME" \
  --role="roles/aiplatform.user"

# Logging Writer & Viewer
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT_NAME" \
  --role="roles/logging.logWriter"
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT_NAME" \
  --role="roles/logging.viewer"
```
Validate the permissions in the [IAM console](https://console.cloud.google.com/iam-admin).

### Artifact Registry

Create an Artifact Registry repository to store Docker images for the agents, MCP server, and the InstaVibe application.

```bash
export REPO_NAME="introveally-repo" # Or your preferred repo name
gcloud artifacts repositories create $REPO_NAME \
  --repository-format=docker \
  --location=us-central1 \
  --description="Docker repository for InstaVibe workshop"
```

### Google Maps Platform API Key

The InstaVibe application uses Google Maps to display event locations. You need to create an API key and restrict it.

1.  **Create API Key:**
    *   Go to **APIs & Services > Credentials** in the Google Cloud Console.
    *   Click **+ CREATE CREDENTIALS** and select **API key**.
    *   Note down the newly created API key.
2.  **Restrict API Key:**
    *   Click on the new API key (e.g., "API key 1") and select **Edit API key**.
    *   Under **Restrict and rename API key**:
        *   Set the **Name** to `Maps Platform API Key`. (This exact name is important for scripts in the codelab).
        *   Under **Application restrictions**, ensure **None** is selected.
        *   Under **API restrictions**, select **Restrict key**.
        *   From the **Select APIs** dropdown, search for and select **Maps JavaScript API**.
        *   Click **OK**.
    *   Click **SAVE**.

### Spanner Graph Database Setup

InstaVibe uses a Graph Database to store and query social connections. This is implemented using Google Cloud Spanner.

1.  **Provision Spanner Instance and Database:**
    Ensure your environment variables (`SPANNER_INSTANCE_ID`, `SPANNER_DATABASE_ID`, `PROJECT_ID`, `SERVICE_ACCOUNT_NAME`) are set (refer to the "Environment Variables" section or run `. ./set_env.sh`).

    ```bash
    # Create Spanner Instance
    gcloud spanner instances create $SPANNER_INSTANCE_ID \
      --config=regional-us-central1 \
      --description="GraphDB Instance InstaVibe" \
      --processing-units=100 \
      --edition=ENTERPRISE

    # Create Spanner Database
    gcloud spanner databases create $SPANNER_DATABASE_ID \
      --instance=$SPANNER_INSTANCE_ID \
      --database-dialect=GOOGLE_STANDARD_SQL
    ```

2.  **Grant Spanner Access:**
    Grant the default service account read/write access to the Spanner database.

    ```bash
    echo "Granting Spanner read/write access to ${SERVICE_ACCOUNT_NAME} for database ${SPANNER_DATABASE_ID}..."
    gcloud spanner databases add-iam-policy-binding ${SPANNER_DATABASE_ID} \
      --instance=${SPANNER_INSTANCE_ID} \
      --member="serviceAccount:${SERVICE_ACCOUNT_NAME}" \
      --role="roles/spanner.databaseUser" \
      --project=${PROJECT_ID}
    ```

3.  **Setup Schema and Load Data:**
    Set up a Python virtual environment, install dependencies, and run the `setup.py` script to define the schema and load initial data into Spanner.

    ```bash
    # Ensure you are in the root of the cloned repository: cd ~/instavibe-bootstrap
    python -m venv env
    source env/bin/activate
    pip install -r requirements.txt # Installs dependencies from the root requirements.txt
    cd instavibe
    python setup.py # Sets up schema and loads data for the instavibe app
    cd .. # Return to the root directory
    # It's good practice to deactivate the virtual environment if you're done with it
    # deactivate
    ```

4.  **Verify (Optional):**
    You can verify the setup in the Google Cloud Console:
    *   Navigate to **Spanner**.
    *   Click on your instance (`instavibe-graph-instance`).
    *   Click on your database (`graphdb`).
    *   Go to **Spanner Studio** and run sample graph queries provided in the codelab (Section 4) to see the data.
        *   Example: Find all Person nodes and their Friendships:
            ```sql
            Graph SocialGraph
            MATCH result_paths = ((p:Person)-[f:Friendship]-(friend:Person))
            RETURN SAFE_TO_JSON(result_paths) AS result_paths
            ```

### Deploying the InstaVibe Web Application

Once the database is set up, deploy the core InstaVibe web application to Cloud Run.

1.  **Retrieve Google Maps API Key:**
    The application needs the Google Maps API key you created. The following script retrieves the key string using its display name. Ensure your environment variables are set (`. ./set_env.sh`).

    ```bash
    export KEY_DISPLAY_NAME="Maps Platform API Key" # Must match the name you gave the key

    GOOGLE_MAPS_KEY_ID=$(gcloud services api-keys list \
      --project="${PROJECT_ID}" \
      --filter="displayName='${KEY_DISPLAY_NAME}'" \
      --format="value(uid)" \
      --limit=1)

    GOOGLE_MAPS_API_KEY=$(gcloud services api-keys get-key-string "${GOOGLE_MAPS_KEY_ID}" \
        --project="${PROJECT_ID}" \
        --format="value(keyString)")

    echo "${GOOGLE_MAPS_API_KEY}" > ~/mapkey.txt # Save it for convenience
    echo "Retrieved GOOGLE_MAPS_API_KEY and saved to ~/mapkey.txt"
    # Ensure GOOGLE_MAPS_API_KEY is exported if you need it in the current session for deployment
    export GOOGLE_MAPS_API_KEY=$(cat ~/mapkey.txt)
    ```
    Carefully check the output to ensure the `GOOGLE_MAPS_API_KEY` matches the key you created.

2.  **Build and Push Container Image:**
    Build the container image for the InstaVibe web application and push it to Artifact Registry.
    Ensure your environment variables are set (`. ./set_env.sh`). The `REPO_NAME` should match what you created earlier (e.g., `introveally-repo`).

    ```bash
    cd ~/instavibe-bootstrap/instavibe/
    export IMAGE_TAG="latest"
    # export APP_FOLDER_NAME="instavibe" # Not strictly needed for this script block
    export IMAGE_NAME="instavibe-webapp"
    # Ensure REGION is set, e.g., export REGION="us-central1"
    export IMAGE_PATH="${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${IMAGE_TAG}"
    # export SERVICE_NAME="instavibe" # Will be used in deploy step

    gcloud builds submit . \
      --tag=${IMAGE_PATH} \
      --project=${PROJECT_ID}
    cd ~/instavibe-bootstrap # Return to root
    ```

3.  **Deploy to Cloud Run:**
    Deploy the built image to Cloud Run.
    Ensure your environment variables are set (`. ./set_env.sh`), including `SPANNER_INSTANCE_ID`, `SPANNER_DATABASE_ID`, `GOOGLE_MAPS_API_KEY`, `PROJECT_ID`, and `REGION`.

    ```bash
    # cd ~/instavibe-bootstrap/instavibe/ # Not necessary if IMAGE_PATH is correct
    export IMAGE_TAG="latest"
    export IMAGE_NAME="instavibe-webapp"
    # Ensure REGION is set
    export IMAGE_PATH="${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${IMAGE_TAG}"
    export SERVICE_NAME="instavibe"

    gcloud run deploy ${SERVICE_NAME} \
      --image=${IMAGE_PATH} \
      --platform=managed \
      --region=${REGION} \
      --allow-unauthenticated \
      --set-env-vars="SPANNER_INSTANCE_ID=${SPANNER_INSTANCE_ID}" \
      --set-env-vars="SPANNER_DATABASE_ID=${SPANNER_DATABASE_ID}" \
      --set-env-vars="APP_HOST=0.0.0.0" \
      --set-env-vars="APP_PORT=8080" \
      --set-env-vars="GOOGLE_CLOUD_LOCATION=${REGION}" \
      --set-env-vars="GOOGLE_CLOUD_PROJECT=${PROJECT_ID}" \
      --set-env-vars="GOOGLE_MAPS_API_KEY=${GOOGLE_MAPS_API_KEY}" \
      --project=${PROJECT_ID} \
      --min-instances=1 # As per codelab
    ```

4.  **Access the Application:**
    Once deployment is complete, Cloud Run will provide a public URL for your InstaVibe application. You can also find this URL in the [Cloud Run section](https://console.cloud.google.com/run) of the Google Cloud Console. Open this URL in your browser to explore the platform.

## Working with Agents

This section delves into the different AI agents used in the InstaVibe system, their functionalities, and how they are developed and tested using Google's Agent Development Kit (ADK) and other protocols.

### Event Planner Agent (ADK)

The Event Planner agent is the first intelligent agent developed for InstaVibe. Its core purpose is to generate creative and tailored event suggestions based on user requests.

**Key Concepts & Functionality:**

*   **ADK Framework:** Built using Google's Agent Development Kit (ADK), a modular framework for developing and deploying AI agents. ADK simplifies agent creation, lifecycle management, and tool integration.
*   **Agent Definition (`agents/planner/agent.py`):**
    *   The agent is defined with a name, the underlying LLM (e.g., `gemini-2.0-flash`), a description, and detailed instructions.
    *   **Instructions:** Guide the LLM on how to behave, what inputs to expect (e.g., start/end dates, location, interests), constraints (e.g., budget, creativity), and the desired output format.
    *   **Tools:** Initially equipped with `google_search` to fetch real-time information about events and venues.
*   **ADK Dev UI:**
    *   An interactive web UI for testing agents locally.
    *   Launched using `adk web` from the `~/instavibe-bootstrap/agents` directory (after setting environment variables and activating the Python environment).
    *   Allows selecting an agent (e.g., `planner`) and chatting with it to see responses.
    *   Example prompt: `Search and plan something in Seattle for me this weekend`
*   **Evaluation (Eval Tab in Dev UI):**
    *   ADK provides features for agent evaluation, crucial for generative and non-deterministic AI.
    *   Predefined test files (e.g., `plan_eval.evalset.json` for the planner) contain inputs and criteria.
    *   Allows running evaluations for specific scenarios (e.g., different cities) to test agent performance systematically.
*   **Structured Output (JSON):**
    *   The agent's instructions are refined to request output *exclusively* in a single JSON object.
    *   This structured format (with keys like `fun_plans`, `plan_description`, `locations_and_activities`) is essential for programmatic use by the InstaVibe application.
    *   Tested by re-launching the ADK Dev UI and providing a prompt like: `Plan an event Boston this weekend with art and coffee`
*   **Programmatic Invocation (`agents/planner/planner_client.py`):**
    *   ADK agents can be run programmatically using the `Runner`.
    *   The `Runner` manages the agent's session, event flow, state, and tool calls.
    *   **Key ADK Components:**
        *   `Session`: Container for a single chat thread, holding history, state, and metadata.
        *   `State`: Agent's short-term working memory within a session.
        *   `Memory`: Potential for long-term recall (in-memory services used in the example).
        *   `Event`: Immutable record of each interaction (user message, agent response, tool use, etc.).
    *   The client script (`python -m planner.planner_client`) demonstrates how to:
        *   Create a session.
        *   Define a user query.
        *   Configure and use the `Runner` to execute the agent asynchronously.
        *   Print each event generated during execution, providing a detailed trace.

**To test the Event Planner Agent locally:**

1.  Ensure environment variables are set:
    ```bash
    # From ~/instavibe-bootstrap directory
    . ./set_env.sh
    source ./env/bin/activate # Activate Python virtual environment
    cd ./agents
    # Ensure .env file in agents/planner/ has GOOGLE_CLOUD_PROJECT set
    sed -i "s|^\(O\?GOOGLE_CLOUD_PROJECT\)=.*|GOOGLE_CLOUD_PROJECT=${PROJECT_ID}|" ~/instavibe-bootstrap/agents/planner/.env
    ```
2.  Launch the ADK Dev UI:
    ```bash
    adk web
    ```
3.  Access the UI via Cloud Shell's Web Preview (usually port 8000). Select the `planner` agent.
4.  To run the client script:
    ```bash
    # Ensure environment variables are set as above
    # From ~/instavibe-bootstrap/agents directory
    python -m planner.planner_client
    ```

### Platform Interaction Agent (MCP)

To enable agents to interact with external systems like the InstaVibe platform APIs, the Model Context Protocol (MCP) is used.

**1. InstaVibe MCP Server (`tools/instavibe/`)**

*   **Purpose:** Exposes InstaVibe's internal APIs (`/api/posts` and `/api/events`) as tools that MCP-compatible agents can use.
*   **Model Context Protocol (MCP):** An open standard for standardizing how AI applications connect with external tools, data sources, and systems. It uses a client-server architecture.
*   **Wrapper Functions (`tools/instavibe/instavibe.py`):**
    *   Python functions (`create_post`, `create_event`) are created to wrap the HTTP POST requests to the InstaVibe application's API endpoints.
    *   These functions handle the request logic and JSON payloads.
*   **MCP Server Implementation (`tools/instavibe/mcp_server.py`):**
    *   Built using HTTP and Server-Sent Events (SSE) for communication.
    *   Implements two key MCP functionalities:
        *   `list_tools`: Allows clients to discover available tools (e.g., `instavibe_create_post_tool`, `instavibe_create_event_tool`) and their schemas.
        *   `call_tool`: Handles execution requests for a specific tool, invoking the corresponding Python wrapper function.
*   **Deployment:**
    1.  The MCP server is packaged as a Docker container.
        ```bash
        # From ~/instavibe-bootstrap/tools/instavibe directory
        # Ensure environment variables are set: . ../../set_env.sh
        export IMAGE_TAG="latest"
        export MCP_IMAGE_NAME="mcp-tool-server"
        export IMAGE_PATH="${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${MCP_IMAGE_NAME}:${IMAGE_TAG}"
        # INSTAVIBE_BASE_URL should point to your deployed instavibe app's /api endpoint
        export INSTAVIBE_BASE_URL=$(gcloud run services list --platform=managed --region=${REGION} --format='value(URL)' | grep instavibe)/api

        gcloud builds submit . \
          --tag=${IMAGE_PATH} \
          --project=${PROJECT_ID}
        ```
    2.  Deployed as a service on Google Cloud Run.
        ```bash
        # From ~/instavibe-bootstrap/tools/instavibe directory
        # Ensure environment variables from previous step are set
        export SERVICE_NAME="mcp-tool-server"
        gcloud run deploy ${SERVICE_NAME} \
          --image=${IMAGE_PATH} \
          --platform=managed \
          --region=${REGION} \
          --allow-unauthenticated \
          --set-env-vars="INSTAVIBE_BASE_URL=${INSTAVIBE_BASE_URL}" \
          --set-env-vars="APP_HOST=0.0.0.0" \
          --set-env-vars="APP_PORT=8080" \
          # Add other relevant env vars like GOOGLE_GENAI_USE_VERTEXAI, GOOGLE_CLOUD_LOCATION, GOOGLE_CLOUD_PROJECT (if needed by this specific server)
          --project=${PROJECT_ID} \
          --min-instances=1
        ```
    3.  The deployed MCP server's URL (ending in `/sse`) is captured for the client agent.
        ```bash
        export MCP_SERVER_URL=$(gcloud run services list --platform=managed --region=${REGION} --format='value(URL)' | grep mcp-tool-server)/sse
        echo "MCP Server URL: ${MCP_SERVER_URL}"
        ```

**2. Platform Interaction Agent as MCP Client (`agents/platform_mcp_client/`)**

*   **Purpose:** This ADK agent acts as an MCP client to communicate with the `mcp-tool-server`.
*   **Agent Definition (`agents/platform_mcp_client/agent.py`):**
    *   The agent is configured with instructions to help users create posts or register events on InstaVibe.
    *   **Dynamic Tool Fetching:** Instead of defining tools statically, it uses `MCPToolset.from_server` to connect to the `MCP_SERVER_URL` and retrieve the list of available tools (e.g., `instavibe_create_post_tool`, `instavibe_create_event_tool`) exposed by the MCP server.
    *   These fetched tools are then provided to the ADK agent.
*   **Testing with ADK Dev UI:**
    1.  Ensure environment variables are set, including `MCP_SERVER_URL`.
        ```bash
        # From ~/instavibe-bootstrap directory
        . ./set_env.sh
        source ./env/bin/activate # Activate Python virtual environment
        export MCP_SERVER_URL=$(gcloud run services list --platform=managed --region=${REGION} --format='value(URL)' | grep mcp-tool-server)/sse
        cd ./agents
        # Ensure .env file in agents/platform_mcp_client/ has GOOGLE_CLOUD_PROJECT and MCP_SERVER_URL
        sed -i "s|^\(O\?GOOGLE_CLOUD_PROJECT\)=.*|GOOGLE_CLOUD_PROJECT=${PROJECT_ID}|" ~/instavibe-bootstrap/agents/platform_mcp_client/.env
        sed -i "s|^\(O\?MCP_SERVER_URL\)=.*|MCP_SERVER_URL=${MCP_SERVER_URL}|" ~/instavibe-bootstrap/agents/platform_mcp_client/.env
        ```
    2.  Launch the ADK Dev UI: `adk web`
    3.  Select the `platform_mcp_client` agent.
    4.  **Test `create_post`:**
        *   Prompt: `Create a post saying "Y'all I just got the cutest lil void baby 😭✨ Naming him Abyss bc he's deep, mysterious, and lowkey chaotic 🔥🖤 #VoidCat #NewRoomie" I'm Julia`
        *   Verify: The new post appears on the deployed InstaVibe application.
    5.  **Test `create_event`:**
        *   Prompt with event details (name, description, date, locations, attendees) as shown in the codelab (Section 8).
        *   Example structure for event details:
            ```json
            {
              "event_name": "Mexico City Culinary & Art Day",
              "description": "A vibrant day...",
              "event_date": "2025-05-17T12:00:00-06:00",
              "locations": [
                { "name": "El Tizoncito", "description": "...", "latitude": 19.412179, "longitude": -99.171308, "address": "..." },
                { "name": "Museo Soumaya", "description": "...", "latitude": 19.440056, "longitude": -99.204281, "address": "..." }
              ],
              "attendee_names": ["Hannah", "George", "Julia"]
            }
            ```
        *   You'd phrase the request naturally, like: `Hey, can you set up an event for Hannah and George and me, and I'm Julia? Let's call it 'Mexico City Culinary & Art Day'. Here are more info [paste JSON here]`
        *   Verify: The new event appears in the "Events" section of the InstaVibe application.

### Social Profiling Agent (ADK Workflow)

To provide personalized planning, the Social Profiling agent analyzes users' social circles and activities using data from the Spanner Graph Database.

**Key Concepts & Functionality:**

*   **Purpose:** Gathers context about friends' activities and interests to enable more tailored event suggestions.
*   **Graph Database Tools (`agents/social/instavibe.py`):**
    *   Python functions are added to query the Spanner graph database:
        *   `get_person_attended_events(person_id)`: Fetches events a person attended.
        *   `get_person_id_by_name(name)`: Retrieves a person's ID by their name.
        *   `get_person_posts(person_id)`: Fetches posts written by a person.
        *   `get_person_friends(person_id)`: Fetches a person's friends.
    *   These functions use Spanner's graph SQL capabilities (`Graph SocialGraph MATCH ...`) and standard SQL.
*   **ADK Workflow Agents (`agents/social/agent.py`):**
    *   Complex tasks like analyzing multiple friends' profiles and summarizing findings are handled using ADK's multi-agent capabilities, specifically **Workflow Agents**.
    *   A **Loop Agent** (`root_agent`) is used to iterate through the process of analyzing profiles and updating a summary.
    *   **Sub-Agents:**
        *   `profile_agent` (LlmAgent): Answers questions about a single person's social profile using the graph database tools. It takes a person's name, fetches their ID, and then gathers their posts, friends, and attended events.
        *   `summary_agent` (LlmAgent): Generates a comprehensive social summary as a single paragraph. If multiple profiles are analyzed, it identifies common ground. Its output is stored in the agent's `State` under the key `summary`.
        *   `check_agent` (LlmAgent): Checks if all requested profiles have been summarized. Outputs `completed` or `pending` to the agent's `State` under the key `summary_status`.
        *   `CheckCondition` (Custom BaseAgent): A programmatic check that inspects `summary_status` in the `State`. If "completed", it signals the Loop Agent to stop by setting `actions=EventActions(escalate=True)`.
*   **State and Callbacks:**
    *   **State:** ADK's `State` is used to persist information (like `summary` and `summary_status`) across iterations of the Loop Agent.
    *   **`after_agent_callback` (`modify_output_after_agent`):** Since the Loop Agent doesn't automatically return the final result from its state, a callback function is used. This function runs when the loop finishes (due to `CheckCondition` escalating). It retrieves the final `summary` from the `State` and formats it as the Loop Agent's final output message.
*   **Root Loop Agent Definition:**
    *   The `root_agent` (LoopAgent) is configured with the sequence of sub-agents: `profile_agent` -> `summary_agent` -> `check_agent` -> `CheckCondition`.
    *   It has a `max_iterations` limit and uses the `after_agent_callback` to produce the final output.
*   **Testing with ADK Dev UI:**
    1.  Ensure environment variables are set, and Spanner is populated.
        ```bash
        # From ~/instavibe-bootstrap directory
        . ./set_env.sh
        source ./env/bin/activate # Activate Python virtual environment
        cd ./agents
        # Ensure .env file in agents/social/ has GOOGLE_CLOUD_PROJECT
        sed -i "s|^\(O\?GOOGLE_CLOUD_PROJECT\)=.*|GOOGLE_CLOUD_PROJECT=${PROJECT_ID}|" ~/instavibe-bootstrap/agents/social/.env
        ```
    2.  Launch the ADK Dev UI: `adk web`
    3.  Select the `Social Agent` (which is the `root_agent` from `agents/social/agent.py`).
    4.  Prompt: `Tell me about Mike and Bob` (assuming Mike and Bob are in the sample data).
    5.  **Verification:**
        *   Observe the chat output for the summary.
        *   Navigate to the **Events** tab in the ADK Dev UI. This provides a detailed trace of the workflow, showing invocations of each sub-agent, tool calls made by `profile_agent` to query Spanner, and state updates. This trace is crucial for understanding and debugging the multi-agent interaction.

### Agent-to-Agent (A2A) Communication

To enable agents to operate as independent services and collaborate, the Agent-to-Agent (A2A) communication protocol is used. This allows for a distributed multi-agent system.

**Key Concepts & Functionality:**

*   **A2A Protocol:** An open standard for interoperable communication between AI agents, distinct from MCP (which is for agent-to-tool interaction). A2A allows agents to:
    *   **Discover:** Find other agents and learn their capabilities via **Agent Cards**.
    *   **Communicate:** Exchange messages and data.
    *   **Collaborate:** Delegate tasks and coordinate actions.
*   **A2A Server:** Each ADK agent (Planner, Platform Interaction, Social) that needs to be accessible remotely is wrapped with an A2A Server component. This server:
    *   Exposes an **Agent Card**: A standardized JSON description of the agent's capabilities, name, URL, version, supported input/output modes, and skills. Served via an HTTP endpoint (e.g., `/agent-card`).
    *   Listens for tasks from other agents (A2A clients).
    *   Manages task execution using the underlying ADK agent logic.
*   **`AgentWithTaskManager`:** A wrapper class (e.g., `PlannerAgent` in `agents/planner/planner_agent.py`) is created for each ADK agent to bridge the A2A server's task handling with the ADK `Runner` and agent logic.
*   **A2A Server Script (`a2a_server.py` for each agent):**
    *   Defines the `AgentCard` with specific details for that agent (name, description, skills, examples).
    *   Configures and starts the `A2AServer`, linking it to the respective `AgentWithTaskManager` implementation.
    *   The server listens on a specific host and port (e.g., `0.0.0.0:8080` when deployed in Cloud Run).

**Enabling A2A for InstaVibe Agents:**

The Planner, Platform Interaction (MCP), and Social Profiling agents are each converted into A2A-enabled services deployed on Cloud Run.

**1. Planner Agent (A2A Enabled - `agents/planner/`)**
    *   `planner_agent.py` updated with `PlannerAgent(AgentWithTaskManager)`.
    *   `a2a_server.py` defines the Planner's Agent Card and starts the A2A server.
    *   **Build & Deploy:**
        ```bash
        # From ~/instavibe-bootstrap/agents directory
        # Ensure environment variables are set: . ../set_env.sh
        # Activate Python env: source ../env/bin/activate
        export AGENT_NAME="planner"
        export IMAGE_NAME="planner-agent"
        export IMAGE_PATH="${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:latest"
        # The PUBLIC_URL for the Agent Card will be determined after deployment.
        # Example placeholder: export PUBLIC_URL="https://your-planner-agent-url.a.run.app"

        gcloud builds submit . \
          --config=cloudbuild.yaml \
          --project=${PROJECT_ID} \
          --region=${REGION} \
          --substitutions=_AGENT_NAME=${AGENT_NAME},_IMAGE_PATH=${IMAGE_PATH}

        # Note: SERVICE_NAME should match the _AGENT_NAME used in the build for consistency, or be explicitly defined.
        gcloud run deploy planner-agent \
          --image=${IMAGE_PATH} \
          --platform=managed \
          --region=${REGION} \
          --set-env-vars="A2A_HOST=0.0.0.0,A2A_PORT=8080,GOOGLE_GENAI_USE_VERTEXAI=TRUE,GOOGLE_CLOUD_LOCATION=${REGION},GOOGLE_CLOUD_PROJECT=${PROJECT_ID}" \
          --allow-unauthenticated \
          --project=${PROJECT_ID} \
          --min-instances=1

        # Update PUBLIC_URL with the actual deployed service URL for the Agent Card
        export PLANNER_AGENT_URL=$(gcloud run services describe planner-agent --platform=managed --region=${REGION} --format='value(status.url)')
        gcloud run services update planner-agent --platform=managed --region=${REGION} --update-env-vars="PUBLIC_URL=${PLANNER_AGENT_URL}" --project=${PROJECT_ID}
        echo "Planner Agent URL: ${PLANNER_AGENT_URL}"
        curl ${PLANNER_AGENT_URL}/agent-card | jq # Verify Agent Card
        ```

**2. Platform Interaction Agent (A2A Enabled - `agents/platform_mcp_client/`)**
    *   `platform_agent.py` updated with `PlatformAgent(AgentWithTaskManager)`.
    *   `a2a_server.py` defines the Platform Interaction agent's Agent Card.
    *   **Build & Deploy (Needs `MCP_SERVER_URL`):**
        ```bash
        # From ~/instavibe-bootstrap/agents directory
        # Ensure environment variables are set, activate Python env
        export AGENT_NAME="platform_mcp_client"
        export IMAGE_NAME="platform-mcp-client-agent"
        export SERVICE_NAME_PLATFORM="platform-mcp-client"
        export IMAGE_PATH="${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:latest"
        export MCP_SERVER_URL=$(gcloud run services describe mcp-tool-server --platform=managed --region=${REGION} --format='value(status.url)')/sse
        # The PUBLIC_URL for the Agent Card will be determined after deployment.

        gcloud builds submit . \
          --project=${PROJECT_ID} \
          --region=${REGION} \
          --config=cloudbuild.yaml \
          --substitutions=_AGENT_NAME=${AGENT_NAME},_IMAGE_PATH=${IMAGE_PATH}

        gcloud run deploy ${SERVICE_NAME_PLATFORM} \
          --image=${IMAGE_PATH} \
          --platform=managed \
          --region=${REGION} \
          --allow-unauthenticated \
          --project=${PROJECT_ID} \
          --set-env-vars="MCP_SERVER_URL=${MCP_SERVER_URL},A2A_HOST=0.0.0.0,A2A_PORT=8080,GOOGLE_GENAI_USE_VERTEXAI=TRUE,GOOGLE_CLOUD_LOCATION=${REGION},GOOGLE_CLOUD_PROJECT=${PROJECT_ID}" \
          --min-instances=1

        export PLATFORM_MPC_CLIENT_URL=$(gcloud run services describe ${SERVICE_NAME_PLATFORM} --platform=managed --region=${REGION} --format='value(status.url)')
        gcloud run services update ${SERVICE_NAME_PLATFORM} --platform=managed --region=${REGION} --update-env-vars="PUBLIC_URL=${PLATFORM_MPC_CLIENT_URL}" --project=${PROJECT_ID}
        echo "Platform MCP Client URL: ${PLATFORM_MPC_CLIENT_URL}"
        curl ${PLATFORM_MPC_CLIENT_URL}/agent-card | jq
        ```

**3. Social Agent (A2A Enabled - `agents/social/`)**
    *   `social_agent.py` updated with `SocialAgent(AgentWithTaskManager)`.
    *   `a2a_server.py` defines the Social agent's Agent Card.
    *   **Build & Deploy (Needs Spanner details):**
        ```bash
        # From ~/instavibe-bootstrap/agents directory
        # Ensure environment variables are set, activate Python env
        export AGENT_NAME="social"
        export IMAGE_NAME="social-agent"
        export SERVICE_NAME_SOCIAL="social-agent"
        export IMAGE_PATH="${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:latest"
        # The PUBLIC_URL for the Agent Card will be determined after deployment.

        gcloud builds submit . \
          --config=cloudbuild.yaml \
          --project=${PROJECT_ID} \
          --region=${REGION} \
          --substitutions=_AGENT_NAME=${AGENT_NAME},_IMAGE_PATH=${IMAGE_PATH}

        gcloud run deploy ${SERVICE_NAME_SOCIAL} \
          --image=${IMAGE_PATH} \
          --platform=managed \
          --region=${REGION} \
          --set-env-vars="SPANNER_INSTANCE_ID=${SPANNER_INSTANCE_ID},SPANNER_DATABASE_ID=${SPANNER_DATABASE_ID},GOOGLE_CLOUD_PROJECT=${PROJECT_ID},A2A_HOST=0.0.0.0,A2A_PORT=8080,GOOGLE_GENAI_USE_VERTEXAI=TRUE,GOOGLE_CLOUD_LOCATION=${REGION}" \
          --allow-unauthenticated \
          --project=${PROJECT_ID} \
          --min-instances=1

        export SOCIAL_AGENT_URL=$(gcloud run services describe ${SERVICE_NAME_SOCIAL} --platform=managed --region=${REGION} --format='value(status.url)')
        gcloud run services update ${SERVICE_NAME_SOCIAL} --platform=managed --region=${REGION} --update-env-vars="PUBLIC_URL=${SOCIAL_AGENT_URL}" --project=${PROJECT_ID}
        echo "Social Agent URL: ${SOCIAL_AGENT_URL}"
        curl ${SOCIAL_AGENT_URL}/agent-card | jq
        ```
**Note on `PUBLIC_URL`:** The `PUBLIC_URL` environment variable, set in the Agent Card and during Cloud Run deployment, is crucial. It advertises how the agent can be reached. The commands above now include a step to fetch the deployed URL and update the service with this value for `PUBLIC_URL`, ensuring the Agent Card is accurate.

### Orchestrator Agent (A2A Client)

The Orchestrator Agent acts as the central coordinator in the multi-agent system. It receives user requests, determines the appropriate specialized remote A2A-enabled agent(s) to handle the request, and delegates tasks accordingly. For the codelab, the Orchestrator is run locally using the ADK Dev UI, while the specialized agents run remotely on Cloud Run.

**Key Concepts & Functionality (`agents/orchestrate/`):**

*   **A2A Client:** The Orchestrator acts as an A2A client, sending tasks to the A2A server interfaces of the Planner, Platform, and Social agents.
*   **Agent Definition (`agents/orchestrate/host_agent.py`):**
    *   The `HostAgent` class manages connections to remote agents.
    *   `register_agent_card(card: AgentCard)`: Stores connection details from the fetched Agent Cards of remote agents.
    *   `create_agent()`: Defines the Orchestrator ADK agent (`orchestrate_agent`).
        *   **Model:** `gemini-2.0-flash`.
        *   **Tools:**
            *   `list_remote_agents`: To get information about the connected Planner, Platform, and Social agents.
            *   `send_task`: The A2A function to delegate work to a remote agent.
        *   **Instructions (`root_instruction`):** This is a critical part. The instructions guide the Orchestrator on:
            *   Understanding user intent and request complexity (single vs. multi-step).
            *   Using `list_remote_agents` for agent discovery and capability assessment.
            *   Selecting the appropriate agent(s) for the task(s).
            *   Planning and sequencing tasks if multiple agents are involved, considering dependencies.
            *   Delegating tasks using `send_task`, providing the `remote_agent_name` and the `user_request` or necessary parameters.
            *   Managing ongoing interactions and monitoring task states.
            *   Communicating the plan and progress to the user.
            *   The instruction template dynamically includes the list of available remote agents (`self.agents`).
*   **Testing the Full A2A System with Orchestrator:**
    1.  Ensure the Planner, Platform Interaction, and Social agents are deployed and running on Cloud Run, and their URLs are accessible.
    2.  Set environment variables for the Orchestrator:
        ```bash
        # From ~/instavibe-bootstrap directory
        . ./set_env.sh
        source ./env/bin/activate # Activate Python virtual environment

        # Retrieve URLs of deployed A2A agents
        export PLANNER_AGENT_URL=$(gcloud run services describe planner-agent --platform=managed --region=${REGION} --format='value(status.url)')
        export PLATFORM_MPC_CLIENT_URL=$(gcloud run services describe platform-mcp-client --platform=managed --region=${REGION} --format='value(status.url)')
        export SOCIAL_AGENT_URL=$(gcloud run services describe social-agent --platform=managed --region=${REGION} --format='value(status.url)')

        export REMOTE_AGENT_ADDRESSES="${PLANNER_AGENT_URL},${PLATFORM_MPC_CLIENT_URL},${SOCIAL_AGENT_URL}"
        echo "Remote Agent Addresses: ${REMOTE_AGENT_ADDRESSES}"

        cd ./agents
        # Ensure .env file in agents/orchestrate/ has GOOGLE_CLOUD_PROJECT and REMOTE_AGENT_ADDRESSES
        sed -i "s|^\(O\?GOOGLE_CLOUD_PROJECT\)=.*|GOOGLE_CLOUD_PROJECT=${PROJECT_ID}|" ~/instavibe-bootstrap/agents/orchestrate/.env
        # Note the use of double quotes for REMOTE_AGENT_ADDRESSES to handle the comma-separated list correctly in sed
        sed -i "s|^\(O\?REMOTE_AGENT_ADDRESSES\)=.*|REMOTE_AGENT_ADDRESSES=\"${REMOTE_AGENT_ADDRESSES}\"|" ~/instavibe-bootstrap/agents/orchestrate/.env
        ```
    3.  Launch the ADK Dev UI for the Orchestrator:
        ```bash
        adk web
        ```
    4.  Access the UI (Cloud Shell Web Preview, port 8000) and select the `orchestrate` agent.
    5.  **Test Scenarios:**
        *   **Complex multi-agent task (Social then Planner):**
            ```
            You are an expert event planner for a user named Diana.
            Your task is to design a fun and personalized night out.

            Here are the details for the plan:
            - Friends to invite: Ian, Nora
            - Desired date: "2025-6-15"
            - Location idea or general preference: "Chicago"
            ... (full prompt from codelab Section 11)
            ```
            *   Observe: The Orchestrator should delegate to the Social Agent first, then use its output to task the Planner Agent. Check the ADK Dev UI Events tab for `send_task` calls to remote agent URLs.
        *   **Direct task for Platform Agent:**
            ```
            Hey, can you set up an event for Laura and Charlie? Let's call it 'Vienna Concert & Castles Day'.
            here are more info
            {
              "event_name": "Vienna Concert & Castles Day",
              "description": "A refined and unforgettable day...",
              "event_date": "2025-06-14T10:00:00+02:00",
              "locations": [
                { "name": "Schönbrunn Palace", "description": "Palace tour and gardens", "latitude": 48.1849, "longitude": 16.3122, "address": "Schönbrunner Schloßstraße 47, 1130 Wien, Austria" },
                { "name": "Musikverein Vienna", "description": "Classical concert", "latitude": 48.2007, "longitude": 16.3721, "address": "Musikvereinsplatz 1, 1010 Wien, Austria" }
              ],
              "attendee_names": ["Laura", "Charlie", "Oscar"]
            }
            And I am Oscar
            ```
            *   Observe: The Orchestrator should delegate this to the "InstaVibe Posting Agent" (Platform Interaction Agent). Verify the event appears in the InstaVibe web application.

## Deployment to Agent Engine

For a production scenario, Google Vertex AI Agent Engine provides a managed, scalable environment for hosting agents. This section covers deploying the Orchestrator agent to Agent Engine and enabling the InstaVibe web application to call it.

**Key Concepts & Functionality:**

*   **Vertex AI Agent Engine:** A fully managed service on Vertex AI for deploying, scaling, and managing AI agents, abstracting infrastructure complexities.
*   **Orchestrator on Agent Engine:** The Orchestrator agent (previously tested locally) is deployed to Agent Engine. It continues to communicate with the specialized agents (Planner, Platform, Social) running on Cloud Run via A2A.
*   **InstaVibe Web App Integration:** The InstaVibe web application (running on Cloud Run) is updated to make remote calls to the Orchestrator agent hosted on Agent Engine.

**Deployment Steps:**

1.  **Deploy Orchestrator to Agent Engine:**
    *   The codelab uses a custom script (`agents/app/agent_engine_app.py`) to package and deploy the Orchestrator agent. This script needs the `REMOTE_AGENT_ADDRESSES` environment variable (comma-separated URLs of the Planner, Platform, and Social agents on Cloud Run) to be correctly set.
    ```bash
    # Ensure you are in the ~/instavibe-bootstrap/agents/ directory
    # Ensure environment variables are set: . ../set_env.sh
    # Ensure Python env is active: source ../env/bin/activate
    # Ensure REMOTE_AGENT_ADDRESSES is set correctly from the previous A2A setup step

    python -m app.agent_engine_app \
      --set-env-vars "AGENT_BASE_URL=${REMOTE_AGENT_ADDRESSES}"
    ```
    *   This deployment process might take a few minutes.

2.  **Update InstaVibe Web Application for Agent Engine Integration:**
    *   **Initialize Agent Engine Client (`instavibe/introvertally.py`):**
        The InstaVibe app code is modified to use the `vertex_ai_hubble_agent_engine` client to interact with the deployed Orchestrator.
        The `ORCHESTRATE_AGENT_ID` (the unique ID of the agent deployed on Agent Engine) is retrieved from an environment variable.
        ```python
        # Example snippet from introvertally.py
        # ORCHESTRATE_AGENT_ID = os.environ.get('ORCHESTRATE_AGENT_ID')
        # agent_engine = agent_engines.get(ORCHESTRATE_AGENT_ID)
        ```
    *   **Make Remote Calls (`instavibe/introvertally.py`):**
        *   The functions that handle the "Introvert Ally" feature are updated to use `agent_engine.stream_query()` to send requests to the Orchestrator agent on Agent Engine. This is used for both generating the initial plan and for user confirmation to post the event.
    *   **Update Flask Routes (`instavibe/ally_routes.py`):**
        Ensure necessary functions from `introvertally.py` (like `call_agent_for_plan`, `post_plan_event`) are imported and used by the Flask routes.
    *   **Add UI Link (`instavibe/templates/base.html`):**
        A navigation link to the "Introvert Ally" feature is added to the base template.

3.  **Retrieve Orchestrator Agent Engine ID:**
    *   The codelab uses a helper script (`instavibe/temp-endpoint.py`) to fetch the deployed Orchestrator's Agent Engine resource ID, as direct `gcloud` commands might be limited for this specific ID.
    ```bash
    # Ensure you are in the ~/instavibe-bootstrap/instavibe/ directory
    # Ensure Python env is active: source ../env/bin/activate
    python temp-endpoint.py
    export ORCHESTRATE_AGENT_ID=$(cat temp_endpoint.txt)
    echo "ORCHESTRATE_AGENT_ID set to: ${ORCHESTRATE_AGENT_ID}"
    ```

4.  **Redeploy InstaVibe Web Application:**
    *   Rebuild the InstaVibe application Docker image and redeploy it to Cloud Run with the new `ORCHESTRATE_AGENT_ID` environment variable.
    ```bash
    # Ensure you are in the ~/instavibe-bootstrap/instavibe/ directory
    # Ensure environment variables are set: . ../set_env.sh
    # Ensure GOOGLE_MAPS_API_KEY is available (e.g., export GOOGLE_MAPS_API_KEY=$(cat ~/mapkey.txt))
    # Ensure ORCHESTRATE_AGENT_ID is exported from the previous step

    export IMAGE_TAG="latest"
    # export APP_FOLDER_NAME="instavibe" # Not strictly needed for this script block
    export IMAGE_NAME="instavibe-webapp"
    export IMAGE_PATH="${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${IMAGE_TAG}"
    export SERVICE_NAME="instavibe"

    gcloud builds submit . \
      --tag=${IMAGE_PATH} \
      --project=${PROJECT_ID}

    gcloud run deploy ${SERVICE_NAME} \
      --image=${IMAGE_PATH} \
      --platform=managed \
      --region=${REGION} \
      --allow-unauthenticated \
      --set-env-vars="SPANNER_INSTANCE_ID=${SPANNER_INSTANCE_ID}" \
      --set-env-vars="SPANNER_DATABASE_ID=${SPANNER_DATABASE_ID}" \
      --set-env-vars="APP_HOST=0.0.0.0" \
      --set-env-vars="APP_PORT=8080" \
      --set-env-vars="GOOGLE_CLOUD_LOCATION=${REGION}" \
      --set-env-vars="GOOGLE_CLOUD_PROJECT=${PROJECT_ID}" \
      --set-env-vars="GOOGLE_MAPS_API_KEY=${GOOGLE_MAPS_API_KEY}" \
      --set-env-vars="ORCHESTRATE_AGENT_ID=${ORCHESTRATE_AGENT_ID}" \
      --project=${PROJECT_ID} \
      --min-instances=1 \
      --cpu=2 \
      --memory=2Gi # As specified in codelab for this deployment
    ```

**Testing the Full AI-Powered InstaVibe Experience:**

*   Navigate to your InstaVibe application URL.
*   Access the "Introvert Ally" feature.
*   Request an event plan. Observe the interaction as the web app calls Agent Engine, which orchestrates the A2A agents.
*   Confirm the plan to trigger posting to InstaVibe.
*   Verify the new post and event on the InstaVibe platform.

**Performance Analysis with Cloud Trace:**

*   Vertex AI Agent Engine integrates with Cloud Trace.
*   You can analyze traces in the Google Cloud Console (under "Trace") for `agent_run[orchestrate_agent]` to understand latencies in the multi-agent system and identify bottlenecks.

## Cleaning Up

To avoid ongoing charges to your Google Cloud account, it's important to delete the resources created during this project setup.

**Important:**
*   Ensure you are running these commands in the same Google Cloud project used for the workshop.
*   If your Cloud Shell session has ended, some environment variables (`$PROJECT_ID`, `$SPANNER_INSTANCE_ID`, `$REGION`, `$REPO_NAME`, etc.) might need to be re-exported or replaced with their actual values. Running `. ./set_env.sh` from the `instavibe-bootstrap` root can help reset many of these.
*   These commands will permanently delete your resources. Double-check if you have other important data in this project before proceeding.

1.  **Reset Environment Variables (Recommended):**
    ```bash
    cd ~/instavibe-bootstrap # Or your project's root directory
    . ./set_env.sh
    source ./env/bin/activate # If you need to run python scripts for cleanup
    ```

2.  **Delete Agent Engine Deployment:**
    The codelab uses a custom script (`runners/remote_delete.py`) to delete the Agent Engine deployment.
    ```bash
    # Ensure you are in ~/instavibe-bootstrap/runners directory
    # Ensure ORCHESTRATE_AGENT_ID is set, e.g., from ~/instavibe-bootstrap/instavibe/temp_endpoint.txt
    # export ORCHESTRATE_AGENT_ID=$(cat ~/instavibe-bootstrap/instavibe/temp_endpoint.txt)
    # cd ~/instavibe-bootstrap/runners
    # python remote_delete.py
    echo "Vertex AI Agent Engine deletion for ORCHESTRATE_AGENT_ID should be handled by \`python remote_delete.py\` as per codelab."
    echo "Ensure ORCHESTRATE_AGENT_ID is correctly set from instavibe/temp_endpoint.txt before running."
    echo "Example: export ORCHESTRATE_AGENT_ID=\$(cat ~/instavibe-bootstrap/instavibe/temp_endpoint.txt); cd ~/instavibe-bootstrap/runners; python remote_delete.py"

    ```
    *Note: The `remote_delete.py` script's existence and functionality are based on the codelab. If it's not present or if there's a more standard gcloud command available by the time you run this, adjust accordingly. For manual deletion, you might need to go to the Vertex AI > Agents section in the Google Cloud Console.*

3.  **Delete Cloud Run Services:**
    Delete the services deployed to Cloud Run one by one.
    ```bash
    # Ensure REGION and PROJECT_ID are set
    gcloud run services delete instavibe --platform=managed --region=${REGION} --project=${PROJECT_ID} --quiet
    gcloud run services delete mcp-tool-server --platform=managed --region=${REGION} --project=${PROJECT_ID} --quiet
    gcloud run services delete planner-agent --platform=managed --region=${REGION} --project=${PROJECT_ID} --quiet
    gcloud run services delete platform-mcp-client --platform=managed --region=${REGION} --project=${PROJECT_ID} --quiet
    gcloud run services delete social-agent --platform=managed --region=${REGION} --project=${PROJECT_ID} --quiet
    echo "Cloud Run services deletion initiated."
    ```

4.  **Delete Spanner Instance:**
    ```bash
    # Ensure SPANNER_INSTANCE_ID and PROJECT_ID are set
    echo "Deleting Spanner instance: ${SPANNER_INSTANCE_ID}..."
    gcloud spanner instances delete ${SPANNER_INSTANCE_ID} --project=${PROJECT_ID} --quiet
    echo "Spanner instance deletion initiated."
    ```
    *Note: Deleting the instance will also delete databases within it, like `graphdb`.*

5.  **Delete Artifact Registry Repository:**
    ```bash
    # Ensure REPO_NAME, REGION, and PROJECT_ID are set (e.g., REPO_NAME="introveally-repo")
    echo "Deleting Artifact Registry repository: ${REPO_NAME}..."
    gcloud artifacts repositories delete ${REPO_NAME} --location=${REGION} --project=${PROJECT_ID} --quiet
    echo "Artifact Registry repository deletion initiated."
    ```
    *Note: You might need to delete images within the repository first if the command fails.*

6.  **Delete API Key (Optional but Recommended):**
    *   Go to **APIs & Services > Credentials** in the Google Cloud Console.
    *   Select the API key named "Maps Platform API Key".
    *   Click **DELETE**.

7.  **Delete Local Workshop Files (Optional):**
    ```bash
    echo "Removing local workshop directory ~/instavibe-bootstrap..."
    # rm -rf ~/instavibe-bootstrap
    # rm -f ~/mapkey.txt
    # rm -f ~/project_id.txt
    echo "Local directory removal commented out for safety. Uncomment if you are sure."
    ```

8.  **Review IAM Permissions:**
    *   Review the IAM page in the Google Cloud Console and remove any project-level roles granted to the service account if they are no longer needed for other purposes. The roles granted in this codelab are quite common for a default service account if it's actively used for deployments.

Deactivate Python environment if you activated it:
```bash
# deactivate
```

## Contributing

Contributions to this project are welcome! If you have suggestions for improvements, bug fixes, or new features, please consider the following:

1.  **Fork the repository.**
2.  **Create a new branch** for your changes (`git checkout -b feature/your-feature-name`).
3.  **Make your changes** and commit them with clear messages.
4.  **Push your branch** to your fork (`git push origin feature/your-feature-name`).
5.  **Open a pull request** against the main repository, describing your changes.

Please ensure your code adheres to any existing styling and testing standards.

## License

The code samples and components within this repository are primarily licensed under the **Apache 2.0 License**.

Please refer to the `LICENSE` file in the original source repository if available, or the standard Apache 2.0 License terms:
[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless otherwise noted, all other content (such as documentation and descriptive text) is provided under the Creative Commons Attribution 4.0 License, and code samples are licensed under the Apache 2.0 License, as per the Google Developers Site Policies referenced in the codelab.
