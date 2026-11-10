# Web Memory Agent

A fast chat agent with exactly **one external tool: web search**. It can search the web when freshness or evidence matters, cite the sources it used, and remember what you asked and when across sessions.

## Features

- **One tool only:** DuckDuckGo web search, exposed to the model as a single `web_search` function.
- **Timestamped memory:** every user and assistant message is stored in SQLite with an ISO timestamp.
- **Relevant recall:** recent conversation turns and keyword-relevant older turns are added to the model context.
- **Grounded answers:** search results are passed as untrusted evidence; the agent is instructed to cite sources and distinguish facts from uncertainty.
- **Fast by design:** small default model, bounded search results, bounded history, and no multi-agent framework.
- **Private by default:** chat history stays in the local SQLite database; it is not sent to the search provider.

## Quick start

Requires Python 3.10+ and an OpenAI-compatible API key.

```bash
git clone https://github.com/aggharsh2005/aggharsh2005.git
cd aggharsh2005/web-memory-agent
python -m venv .venv
# Windows: .venv\\Scripts\\activate
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
```

Copy `.env.example` to `.env` and set `OPENAI_API_KEY`. The default endpoint is OpenAI; you can use any compatible provider by setting `OPENAI_BASE_URL` and `MODEL`.

```bash
streamlit run app.py
```

## Configuration

| Variable | Default | Purpose |
|---|---|---|
| `OPENAI_API_KEY` | — | Required model API key |
| `OPENAI_BASE_URL` | OpenAI default | Optional compatible API endpoint |
| `MODEL` | `gpt-4o-mini` | Fast chat model |
| `DB_PATH` | `agent_memory.db` | Local SQLite history |
| `MAX_SEARCH_RESULTS` | `5` | Maximum results per search |

## How memory works

The app writes each turn to SQLite with a UTC timestamp. For each response it supplies the latest turns plus a small keyword-ranked selection of older turns. The UI shows the exact time each message was recorded. This is conversational memory, not a claim that the model remembers anything outside this app/database.

Use **Clear memory** in the sidebar to permanently clear the local conversation history.

## Tool boundary

The model can call only `web_search`. There are no browser automation, shell, filesystem, email, calendar, or arbitrary API tools. SQLite is an internal persistence layer, not an agent tool.

## Reliability notes

- Search is optional: the agent answers directly when web lookup is unnecessary.
- Search results are untrusted input and cannot override system instructions.
- The model is told to cite URLs for claims derived from search.
- If search fails, the agent explains that and continues without pretending it searched.
- Never put secrets in source control. Keep `.env` local.

See [design.md](design.md) for architecture, UX, safety, and trade-offs.
