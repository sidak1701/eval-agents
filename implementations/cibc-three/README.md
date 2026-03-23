# CIBC Three Notebook

This folder contains a notebook-based prototype for an E2B-backed analytics agent that can:

- inspect the files available in an E2B template under `/data`
- query a SQLite database when a database file is present
- answer natural-language questions directly from raw template files such as CSV and JSON
- answer natural-language questions about the available data
- rank potentially suspicious transactions from the raw fraud-analysis files

The main notebook is [Code Interpreter.ipynb](/home/coder/eval-agents/implementations/cibc-three/Code%20Interpreter.ipynb).

## Overview

The notebook uses `aieng.agents.tools.CodeInterpreter` to run Python inside an E2B sandbox. On top of that sandbox, it defines two lightweight ADK agents:

- a SQL agent for read-only queries against `/data/data.db` when a SQLite database exists
- a template-aware agent that can inspect template files, answer questions directly from CSV and JSON inputs, and run a bounded fraud analysis over those raw files

The template-aware flow is designed for newer E2B templates that expose raw files such as:

- `transactions_data.csv`
- `cards_data.csv`
- `users_data.csv`
- `mcc_codes.json`
- `train_fraud_labels.json`

## What The Notebook Adds

The notebook defines helper tools and agent wrappers for:

- listing files available in the E2B template
- previewing CSV, JSON, JSONL, text, and SQLite files
- answering natural-language questions from file contents when the data lives in raw files instead of a database
- inspecting a SQLite schema
- running validated read-only SQL queries
- scoring potentially suspicious transactions using explicit heuristics

The fraud scoring tool currently considers signals such as:

- card marked as exposed or on the dark web
- unusually large transaction amount
- merchant state mismatch relative to the user address-derived state
- non-empty transaction error fields
- online transaction channel
- higher-risk merchant category keywords

To keep the notebook stable inside E2B, the fraud analysis scans the large transactions file in chunks and uses a bounded scan window instead of processing the entire dataset in one pass.

## Setup

From the repository root:

```bash
uv sync
```

Set the environment variables you need in `.env`:

```bash
GOOGLE_API_KEY="your-api-key"

# Optional, for tracing
LANGFUSE_PUBLIC_KEY="pk-lf-..."
LANGFUSE_SECRET_KEY="sk-lf-..."
```

If both `GOOGLE_API_KEY` and `GEMINI_API_KEY` are set, ADK will prefer `GOOGLE_API_KEY`.

## Notebook Workflow

1. Open [Code Interpreter.ipynb](/home/coder/eval-agents/implementations/cibc-three/Code%20Interpreter.ipynb).
2. Run the setup cells that initialize `CodeInterpreter` and tracing.
3. Run the SQL agent cell if you want database-backed natural-language querying.
4. Run the template-aware agent cell to inspect the current E2B template contents.
5. Use the example cells at the bottom of the notebook to:
   - summarize available files
   - answer questions directly from raw files when there is no database-backed source for the question
   - identify how to join card, user, and transaction data
   - ask for potentially fraudulent transactions
   - ask for the top suspicious transactions with risk factors

## Example Questions

You can ask the template-aware agent questions such as:

- `What files are available in /data?`
- `Using the available CSV and JSON files, summarize what data is available.`
- `Which files should be joined to connect transactions to cardholders?`
- `Find potentially fraudulent transactions using the available files.`
- `Show the top 20 most suspicious transactions and include the risk factors used.`

For database-backed questions, you can ask the SQL agent things such as:

- `What are the top 5 merchant states by transaction count?`
- `Which merchant categories have the highest transaction volume?`

## Limitations

- The fraud ranking is heuristic, not a trained fraud model.
- The current notebook uses a bounded scan for the very large transactions file, so results are based on a sampled window rather than the full dataset.
- Tracing is initialized in the notebook, but any deeper ADK-specific tracing behavior depends on your environment configuration.

## Related Files

- [Code Interpreter.ipynb](/home/coder/eval-agents/implementations/cibc-three/Code%20Interpreter.ipynb)
- [README.md](/home/coder/eval-agents/implementations/cibc-three/README.md)