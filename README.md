# ✈️ TravelBuddy — AI-Powered Multi-Agent Travel Planner

TravelBuddy is an AI-powered travel planning application that transforms natural-language travel requests into personalized, practical, and budget-aware travel itineraries.

The application uses a **multi-agent architecture built with LangGraph and LangChain**, where specialized agents independently handle flights, hotels, weather, budget analysis, and itinerary generation. A supervisor agent dynamically determines which specialist agents are required for each request.

The system also incorporates **AI guardrails, Model Context Protocol (MCP), Human-in-the-Loop (HITL) approval, persistent state management, fallback handling, and external travel-data integrations** to create a more reliable and controlled AI workflow.

---

## 🌟 Key Features

* ✈️ **AI-powered travel planning**
* 🤖 **Multi-agent architecture using LangGraph**
* 🧭 **Supervisor agent for intelligent agent routing**
* 🛡️ **Input guardrails for request validation**
* 👤 **Human-in-the-Loop (HITL) itinerary approval**
* ✈️ **Flight information and recommendations**
* 🏨 **Hotel and accommodation research**
* 🌤️ **Real-time weather and forecast information**
* 💰 **Budget feasibility analysis**
* 🗓️ **Personalized day-by-day itinerary generation**
* 🔌 **Model Context Protocol (MCP) integrations**
* 💾 **PostgreSQL-based state persistence and checkpointing**
* 🔄 **Fallback handling for failed external services**
* 🔐 **Environment-variable based API key management**
* 🌐 **FastAPI backend with web interface**

---

## 🧠 AI Architecture

TravelBuddy follows a **supervisor-based multi-agent architecture**.

The overall workflow is:

```text
                         User Request
                              │
                              ▼
                    ┌──────────────────┐
                    │  Input Guardrail │
                    └────────┬─────────┘
                             │
                  Valid Travel Request?
                       /            \
                     No              Yes
                     │                │
                     ▼                ▼
                  Block          Supervisor
                                  Agent
                                    │
                 ┌──────────────────┼──────────────────┐
                 │                  │                  │
                 ▼                  ▼                  ▼
          Flight Agent        Hotel Agent       Weather Agent
                 │                  │                  │
                 ▼                  ▼                  ▼
               MCP                MCP                MCP
                 │                  │                  │
                 └──────────────────┼──────────────────┘
                                    │
                                    ▼
                              Budget Agent
                                    │
                                    ▼
                            Itinerary Agent
                                    │
                                    ▼
                         Draft Travel Itinerary
                                    │
                                    ▼
                         Human-in-the-Loop
                             Approval
                           /          \
                      Approved       Revision
                         │              │
                         └──────┬───────┘
                                ▼
                         Final Response
```

---

## 🤖 Multi-Agent System

TravelBuddy uses specialized agents instead of relying on a single LLM call.

### 1. Supervisor Agent

The Supervisor Agent analyzes the user's request and determines which specialist agents are required.

It extracts important travel constraints such as:

* Destination
* Origin
* Duration
* Budget
* Travel style
* Special preferences

The supervisor also ensures that the **Itinerary Agent** is included in the workflow.

---

### 2. Flight Agent ✈️

The Flight Agent handles flight-related information such as:

* Departure and arrival airports
* Airlines
* Routes
* Typical flight duration
* Estimated airfare
* Peak-season pricing considerations
* Booking advice

The agent uses MCP-based aviation tools to obtain airport and airline information.

---

### 3. Hotel Agent 🏨

The Hotel Agent researches accommodation options based on the user's destination and travel requirements.

It can provide:

* Hotel recommendations
* Accommodation guidance
* Neighborhood suggestions
* Places to stay

Hotel search is integrated through MCP-based search functionality.

---

### 4. Weather Agent 🌤️

The Weather Agent retrieves weather information and forecasts for the selected destination.

It provides:

* Current weather
* Forecast information
* Seasonal guidance
* Travel and packing considerations

If live weather information is unavailable, the system falls back to general seasonal guidance and recommends verifying the forecast before departure.

---

### 5. Budget Agent 💰

The Budget Agent evaluates whether the planned trip is realistic for the user's budget.

It analyzes:

* Estimated cost categories
* Budget risk areas
* Potential savings
* Overall trip feasibility

When exact live prices are unavailable, estimated costs are explicitly treated as approximate.

---

### 6. Itinerary Agent 🗓️

The Itinerary Agent combines information from the specialist agents and generates a complete travel plan.

It considers:

* User preferences
* Flight information
* Accommodation
* Weather
* Budget
* Trip duration

The result is a practical, budget-aware itinerary prepared for human review.

---

### 7. Final Response Agent

After the Human-in-the-Loop review, the Final Response Agent produces the final polished travel response.

If the itinerary is approved, it preserves the approved decisions.

If the user requests changes, the agent incorporates the provided feedback before producing the final response.

---

# 🛡️ AI Guardrails

TravelBuddy implements a **custom LLM-based input guardrail** before the main multi-agent workflow begins.

The guardrail determines whether a request:

* Belongs to travel planning or travel information
* Is clearly unrelated to travel
* Requests harmful or illegal instructions

Valid travel-related requests can include:

* Destinations
* Flights
* Hotels
* Weather
* Budgets
* Visas
* Transportation
* Sightseeing
* Food
* Packing
* Itineraries

Invalid or unsafe requests are blocked before they reach the specialist agents.

The guardrail returns a structured decision containing:

```json
{
  "allowed": true,
  "reason": ""
}
```

This provides an additional control layer between the user and the agentic workflow.

---

# 👤 Human-in-the-Loop (HITL)

TravelBuddy incorporates **Human-in-the-Loop approval using LangGraph's interrupt mechanism**.

Instead of immediately returning an AI-generated itinerary, the system first creates a draft and pauses the workflow for human review.

The reviewer can:

* ✅ Approve the itinerary
* ✏️ Provide revision feedback
* 🔄 Request changes before finalization

The workflow then resumes based on the human decision.

```text
AI-generated draft
       │
       ▼
Human review
       │
   ┌───┴────┐
   │        │
Approve   Revise
   │        │
   │        ▼
   │   Human feedback
   │        │
   └───┬────┘
       ▼
Final response
```

This approach helps keep a **human decision-maker in the loop instead of allowing the AI to autonomously finalize every recommendation**.

---

# 🔌 Model Context Protocol (MCP)

TravelBuddy uses **Model Context Protocol (MCP)** to connect the AI agents with external tools and data sources.

MCP is used for travel-related tool integrations including:

* ✈️ Aviation information
* 🏨 Hotel/web search
* 🌤️ Weather information
* 🌦️ Weather forecasts

For example, the Flight Agent communicates with an aviation MCP server to retrieve airport and airline information.

The MCP-based approach provides a modular separation between:

```text
AI Agent
   │
   ▼
MCP Client
   │
   ▼
MCP Server
   │
   ▼
External Tool / API
```

This makes external tools easier to integrate into the agentic workflow.

---

# 💾 Persistent State & Checkpointing

TravelBuddy uses **PostgreSQL with LangGraph's `PostgresSaver`** for workflow state persistence and checkpointing.

This allows the LangGraph workflow to maintain important state such as:

* User request
* Selected agents
* Trip constraints
* Agent results
* Generated itinerary
* Human approval
* Human feedback
* Final response

PostgreSQL checkpointing is particularly important for the Human-in-the-Loop workflow because the graph needs to pause and later resume execution.

---

# 🔄 Error Handling & Fallbacks

TravelBuddy includes fallback mechanisms for several potential failure scenarios.

### Guardrail fallback

If guardrail parsing or the LLM response fails, the system uses a fallback behavior rather than immediately breaking the application.

### Supervisor fallback

If supervisor parsing fails, the system falls back to the complete travel-agent workflow.

### External API/MCP fallback

If an external service becomes unavailable, the relevant agent provides a fallback response instead of terminating the entire workflow.

For example, when live weather information is unavailable, the system provides general seasonal guidance and advises the user to verify the forecast.

This makes the overall workflow more resilient to external-service failures.

---

# 🧰 Technology Stack

## AI / Agentic AI

* **LangGraph** — Agent orchestration and workflow management
* **LangChain** — LLM application framework
* **Groq** — LLM inference
* **Multi-Agent Architecture**
* **Human-in-the-Loop (HITL)**
* **AI Input Guardrails**
* **Model Context Protocol (MCP)**

## Backend

* **Python**
* **FastAPI**
* **PostgreSQL**
* **Psycopg**

## External Integrations

* Aviation information through MCP
* Hotel/web search through MCP
* Weather information through MCP
* Weather forecast through MCP

## Frontend

* HTML
* CSS
* JavaScript
* Jinja templates

---

# 📁 Project Structure

```text
TravelBuddy/
│
├── app.py
│   └── FastAPI application entry point
│
├── backend.py
│   └── LangGraph multi-agent workflow
│
├── mcp_client.py
│   └── MCP client and tool integrations
│
├── custom_weather_mcp_server.py
│   └── Custom weather MCP server
│
├── tools/
│   └── External tool integrations
│
├── templates/
│   └── Frontend HTML templates
│
├── static/
│   └── CSS, JavaScript and static assets
│
├── test.py
│   └── Application testing
│
├── requirements.txt
│   └── Python dependencies
│
├── Dockerfile
│   └── Container configuration
│
└── .gitignore
```

---

# ⚙️ Installation

## 1. Clone the Repository

```bash
git clone https://github.com/nuhaaquil/TravelBuddy.git
cd TravelBuddy
```

## 2. Create a Virtual Environment

### Windows

```bash
python -m venv .venv
.venv\Scripts\activate
```

### macOS / Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
```

## 3. Install Dependencies

```bash
pip install -r requirements.txt
```

---

# 🔐 Environment Variables

Create a `.env` file in the project root.

Add the required configuration values:

```env
GROQ_API_KEY=your_groq_api_key

DATABASE_URL=your_postgresql_database_url

AVIATIONSTACK_API_KEY=your_aviationstack_api_key

TAVILY_API_KEY=your_tavily_api_key
```

Do not commit API keys or other secrets to GitHub.

---

# ▶️ Running the Application

Start the FastAPI application:

```bash
python app.py
```

Once the application starts, open:

```text
http://127.0.0.1:8000/
```

---

# 🔄 End-to-End Workflow

A typical TravelBuddy request follows this sequence:

```text
1. User enters a travel request
             ↓
2. Input Guardrail validates the request
             ↓
3. Supervisor Agent analyzes the request
             ↓
4. Required specialist agents are selected
             ↓
5. Flight information is retrieved
             ↓
6. Hotel information is retrieved
             ↓
7. Weather information is retrieved
             ↓
8. Budget feasibility is analyzed
             ↓
9. Itinerary Agent creates a draft itinerary
             ↓
10. Human reviews the generated itinerary
             ↓
11. Human approves OR provides feedback
             ↓
12. Final Response Agent generates the final plan
```

---

# 🧠 Agentic AI Concepts Demonstrated

This project demonstrates several important concepts in modern Generative AI and Agentic AI:

* Multi-agent systems
* Agent orchestration
* Supervisor-based routing
* Stateful AI workflows
* LangGraph StateGraph
* Reducer-based state updates
* Tool calling
* Model Context Protocol (MCP)
* AI guardrails
* Human-in-the-Loop workflows
* Workflow interruption and resumption
* Persistent checkpoints
* External API integration
* Error handling and fallbacks
* LLM-based decision making
* Prompt engineering
* Context aggregation
* Specialized AI agents

---

# 🛡️ Reliability & Responsible AI

TravelBuddy is designed with multiple control mechanisms rather than relying solely on a single LLM response.

### Input Control

Requests are checked using an AI-powered travel-domain guardrail before entering the main workflow.

### Specialized Responsibilities

Different agents are responsible for different tasks, reducing the need for a single agent to handle the entire problem.

### Human Oversight

Generated itineraries go through a Human-in-the-Loop approval stage.

### Persistent Workflow State

PostgreSQL checkpointing allows workflow state to be preserved across interruptions.

### Graceful Degradation

External service failures are handled through fallback responses where possible.

### Approximate Data Handling

When exact live pricing is unavailable, the system distinguishes estimated information from live information.

---

# 🚀 Future Enhancements

Potential improvements include:

* Real-time flight booking integration
* Real-time hotel booking
* More travel APIs
* Destination recommendation agent
* Visa information agent
* Restaurant recommendation agent
* Travel safety agent
* Currency conversion agent
* Personalized user profiles
* Trip history
* Itinerary export to PDF
* Authentication and authorization
* Improved observability and tracing
* Automated evaluation of agent responses
* More advanced guardrail and validation layers

---

# 📌 Project Highlights

TravelBuddy demonstrates how multiple modern AI engineering techniques can be combined into a single practical application:

> **LLM + LangChain + LangGraph + Multi-Agent Architecture + MCP + Guardrails + Human-in-the-Loop + PostgreSQL Checkpointing**

The project focuses not only on generating AI responses, but also on **orchestrating agents, connecting external tools, controlling AI behavior, maintaining workflow state, handling failures, and keeping humans involved in important decisions.**

---

# 👩‍💻 Author

**Nuha Aquil**

GitHub:
https://github.com/nuhaaquil

