# 🤖 Agentic AI — Data Agent

A multi-agent system built with **LangGraph** that routes natural language queries to specialized sub-agents for **SQL analysis** or **ETL workflows**.

---

## Overview

The **Data Agent** acts as an intelligent router: it interprets a user's request, classifies it as a SQL or ETL task, and delegates it to the right sub-agent.

```
                Data Agent (Router)
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
    SQL Analyst Agent          ETL Analyst Agent
    ─────────────────          ──────────────────
    Schema context              Extract → Load
    SQL generation               Transform → Load
    Safety validation           Sandboxed execution
    Query execution
```

**Flow:** user query → router classifies (SQL/ETL) → sub-agent processes → structured result returned.

---

## Features

- **Smart routing** — classifies queries as SQL or ETL automatically
- **SQL Agent** — NL→SQL generation, schema-aware context, safety validation (blocks INSERT/UPDATE/DELETE/DROP/ALTER), execution on PostgreSQL
- **ETL Agent** — API extraction, Pandas transformations, safe code execution, supports CSV/JSON/Parquet
- **Multi-LLM routing** — cheaper models for simple queries, Claude for complex ones
- **Validated I/O** — Pydantic schemas throughout

---

## Setup

```bash
cd Data_Agent
python -m venv .venv
source .venv/bin/activate   # Windows: .\.venv\Scripts\Activate.ps1

pip install -e .            # or: uv pip install -r requirements.txt
```

Create a `.env` file:

```env
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

host=localhost
port=5432
user=postgres
password=your_password
database=data_agent_db
```

---

## Project Structure

```
Data_Agent/
├── agents/          # data_agent.py (router), sql_analyst.py, etl_analyst.py
├── Models/          # schema.py — Pydantic state models
├── utils/           # database.py, etl_tools.py, llm_pick.py
├── data/            # extract/, transform/, sample CSVs
├── main.py
└── feed_db.py
```

---

## Usage

```python
from agents.data_agent import data_agent
from langchain_core.messages import HumanMessage

response = data_agent.invoke({
    "messages": [HumanMessage(content="Show me the top 5 users with the highest ratings")],
    "route_response": ""
})
```

More examples:
- *"Extract data from `https://pokeapi.co/api/v2/pokemon` and save as CSV"* → ETL agent extracts & loads
- *"Transform rides.csv to keep only rating > 4.5, save to data/transform"* → ETL agent transforms & loads
- *"Average rating per vehicle type"* → SQL agent generates and runs a safe `SELECT`

Run directly: `python main.py`

---

## Agents at a Glance

| Agent | File | Job |
|---|---|---|
| **Router** | `agents/data_agent.py` | Classifies intent, dispatches to the right sub-agent |
| **SQL Analyst** | `agents/sql_analyst.py` | Curates question → gathers schema → generates & validates SQL → executes → answers |
| **ETL Analyst** | `agents/etl_analyst.py` | Picks a tool (extract/transform) → generates Pandas code → executes safely → reports |

---

## Security

- SQL queries are validated before execution; destructive statements are blocked
- Generated Pandas code runs in a sandboxed environment
- Credentials live only in `.env`, never in code

---

## Troubleshooting

| Issue | Fix |
|---|---|
| Database connection failed | Check PostgreSQL is running and `.env` credentials |
| API key not found | Confirm keys are set in `.env` |
| SQL query unsafe | Rephrase as a read-only `SELECT` |
| Module not found | Activate the venv and reinstall dependencies |

---

## Learn More

[LangGraph Docs](https://langchain-ai.github.io/langgraph/) · [LangChain Docs](https://python.langchain.com/) · [Claude API](https://docs.anthropic.com/) · [PostgreSQL Docs](https://www.postgresql.org/docs/)

---
