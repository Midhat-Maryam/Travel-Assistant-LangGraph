#  Travel Assistant LangGraph

**Project:** AI-powered travel assistant built with LangChain, LangGraph, and Gradio

---

##  Project Overview

An agentic AI travel assistant that helps users plan trips, check real-time weather, convert currencies, search for travel information, and estimate trip budgets. The assistant uses a LangGraph-powered stateful workflow to reason over user inputs, select the right tools, and maintain multi-turn conversation memory.

---

##  Use Case

**Travel Planning Assistant** — users can ask questions like:
- "What's the weather in Tokyo right now?"
- "Convert 1000 USD to Japanese Yen"
- "What are the top attractions in Istanbul?"
- "Estimate my budget for 7 days in Bangkok"

---

##  Tools Integrated

| Tool | Type | Description |
|---|---|---|
| `get_weather` | External API | Fetches real-time weather for any city |
| `get_exchange_rate` | External API | Converts between currencies at live rates |
| `search_travel_info` | External API | Searches the web for travel tips, attractions, visa info |
| `calculate_travel_budget` | Custom Python | Estimates total trip cost with emergency fund |

---

##  APIs Used

| API | Purpose | Link |
|---|---|---|
| OpenWeatherMap | Real-time weather data | https://openweathermap.org/api |
| ExchangeRate-API | Live currency conversion | https://exchangerate-api.com |
| Tavily Search | Web search for travel info | https://tavily.com |

---

##  LangGraph Workflow

```
User Input
    ↓
Gradio Interface
    ↓
Agent Node (GPT-4o-mini + tools)
    ↓
should_use_tools?
  ├── YES → ToolNode → (tool executes) → back to Agent Node
  └── NO  → END → Final Response → Gradio
```

**Nodes:**
- `agent` — main reasoning node; invokes the LLM with the full message history and system prompt
- `tools` — executes whichever tool(s) the model called

**Edges:**
- `agent → tools` (conditional, when tool calls are present)
- `tools → agent` (always, to process tool results)
- `agent → END` (when no tool calls are needed)

---

##  Memory Implementation

Memory is handled via LangGraph's `MemorySaver` checkpointer:

```python
memory = MemorySaver()
app    = graph_builder.compile(checkpointer=memory)
```

Each conversation is identified by a `thread_id` (generated per Gradio session with `uuid.uuid4()`). Every invocation passes this ID via the config:

```python
config = {"configurable": {"thread_id": session_id}}
```

**Graph state vs memory:**
- **Graph state** (`TravelState`) holds the message list for the current invocation only — it is passed between nodes within a single run.
- **Memory (MemorySaver)** persists the full conversation history across multiple turns using the thread ID as a key. This allows the agent to remember context from earlier in the conversation.

---

##  Gradio Interface

The UI is built with `gr.Blocks` and includes:
- A chat window with conversation history
- Text input and Send button
- New Conversation button (resets session ID and history)
- Example prompt buttons for quick testing
- A sidebar explaining how the agent works

---

##  How to Run

### 1. Install dependencies

```bash
pip install langchain langchain-openai langgraph gradio tavily-python requests
```

### 2. Set API keys

```python
import os
os.environ["OPENAI_API_KEY"]       = "your-openai-key"
os.environ["TAVILY_API_KEY"]       = "your-tavily-key"
os.environ["OPENWEATHER_API_KEY"]  = "your-openweather-key"
os.environ["EXCHANGERATE_API_KEY"] = "your-exchangerate-key"
```

### 3. Run the notebook

Open the notebook in Google Colab or Jupyter and run all cells. The Gradio app will launch with a public share link.

---

##  Example Prompts

- `Hello! I'm planning a trip to Tokyo. What can you help me with?`
- `What's the weather like in Paris right now?`
- `Convert 500 USD to Euros`
- `What are the top attractions in Istanbul?`
- `Do I need a visa to visit Japan as a Pakistani citizen?`
- `Estimate my budget for 5 days in Bangkok: hotel $50/night, food $20/day, transport $10/day, activities $150 total`
- `What's the best time of year to visit Bali?`
- `Convert 10000 PKR to AED`

---

##  Challenges Faced

- **ExchangeRate API error handling** — the simplified tool crashed when the API returned a failure response; added a `result != "success"` guard to surface clean error messages instead of a `KeyError`.
- **LangSmith 403 warnings** — tracing failed due to an incorrect API key; disabled tracing with `LANGCHAIN_TRACING_V2=false` to keep output clean.
- **Graph visualization in Colab** — `draw_mermaid_png()` requires a headless browser which is not always available; fell back to saving the Mermaid source as a `.md` file for GitHub rendering.

---

##  Future Improvements

- Add streaming responses for a more interactive feel
- Integrate a flight search API (e.g. Amadeus or Skyscanner)
- Add hotel search via Booking.com or similar API
- Deploy to Hugging Face Spaces for permanent public access
- Add LangSmith tracing for production monitoring
- Support multi-language responses

---

## 
Workflow Diagram

<img width="662" height="662" alt="Workflow_diagram" src="https://github.com/user-attachments/assets/9db89179-4edf-4b03-944a-57f09f0a135e" />


---

## 📁 Project Structure

```
screenshots/ 
├── README.md                # This file
├── workflow_diagram.png     # Architecture diagram
├── travel_assistant.ipynb   # Main notebook   
```
