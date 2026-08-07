# Agentic AI Engineer with LangChain and LangGraph

This repository contains notes, exercises, and projects for Udacity's
“Agentic AI Engineer with LangChain and LangGraph” Nanodegree. Course material
is organized in official course order:

1. [LangChain Agentic AI Fundamentals](01-langchain-agentic-ai-fundamentals/README.md)
2. [Building AI Agents with LangGraph](02-building-ai-agents-with-langgraph/README.md)
3. [Advanced Agentic AI Techniques](03-advanced-agentic-ai-techniques/README.md)

## How the repository is organized

Each course has a `README.md` index and three content areas:

- `notes/` contains numbered Markdown lesson notes.
- `exercises/` contains lesson exercises and files used by those exercises.
- `project/` contains the course's graded project and closely related assets.

Directories and files use lowercase kebab-case. Courses and ordered lessons
use two-digit prefixes such as `01-` and `02-`. Conventional names such as
`README.md`, and Python package/import names, are not renamed to kebab-case.

Follow each course index to see what has been added and what remains pending.

## Environment setup

This repository uses Python 3.12 or later and `uv`; the existing dependency
versions and resolved lock file are retained in `pyproject.toml` and `uv.lock`.
Install the locked environment with:

```shell
uv sync
```

Exercises that use external services load credentials from an ignored `.env`
file in the applicable exercise directory. Create and edit that file locally,
using the variable names required by the exercise, and never commit or share
its values. Do not run API-backed notebooks or applications unless you intend
to consume the corresponding service quota.
