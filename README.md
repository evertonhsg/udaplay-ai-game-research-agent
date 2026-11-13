# UdaPlay – AI Game Research Agent

UdaPlay answers natural-language questions about video games (titles, release dates, platforms, genres, publishers).
It uses a two-tier retrieval strategy:

1. **RAG over a local dataset** of 15 games stored in a persistent ChromaDB vector database.
2. **LLM-as-judge evaluation** of the retrieved documents (useful? + confidence score).
3. **Web search fallback** with the Tavily API when the local knowledge is not good enough.
4. A **structured, cited answer** (internal game names and/or web URLs).

```
user question ──► retrieve_game ──► evaluate_retrieval ──┬─ useful & confident ──► final answer (internal source)
                    (ChromaDB)        (LLM judge)        └─ not useful ──► game_web_search ──► final answer (URLs)
                                                                             (Tavily)
```

## Contents

| Path | What it is |
|---|---|
| `starter/Udaplay_01_starter_project.ipynb` | Part 1: loads the game JSON files, embeds them into ChromaDB (`chromadb/`, collection `udaplay`), demonstrates semantic search |
| `starter/Udaplay_02_starter_project.ipynb` | Part 2: the three tools, the stateful `Agent` (state machine from `lib/`), example queries with full traces, plus the advanced agent |
| `starter/lib/` | Course library: LLM wrapper, messages, tools, state machine, agent, memory |
| `starter/games/` | The 15 game documents |

### Part 2 highlights
- **Tools:** `retrieve_game` (vector DB), `evaluate_retrieval` (LLM judge returning `EvaluationReport{useful, description, confidence}`), `game_web_search` (Tavily).
- **Stateful agent:** conversation state is kept per `session_id`, so follow-ups like *"who published it?"* work.
- **Visible process:** each query prints the tool calls, their results, the workflow path, and the final cited answer.
- **Advanced (Stand Out):**
  - an explicit state machine where every tool is a node (`rewrite_query → retrieve_game → evaluate_retrieval → [game_web_search] → generate_answer → [memorize]`)
  - **persistent long-term memory**: what the agent learns from web searches is reused in later sessions without searching again
  - **structured JSON output** (`AgentReport`) alongside the natural-language answer

## Running it

```bash
cd starter
python3.11 -m venv .venv && source .venv/bin/activate
pip install "chromadb>=1.0.4" "openai>=1.73.0" "pydantic>=2.11.3" "python-dotenv>=1.1.0" "tavily-python>=0.5.4" "pdfplumber>=0.11.6" jupyter
cp .env.example .env   # then fill in OPENAI_API_KEY, OPENAI_BASE_URL, TAVILY_API_KEY
```

Run `Udaplay_01_starter_project.ipynb` first (it builds `chromadb/`), then `Udaplay_02_starter_project.ipynb`, both from the `starter/` folder.
