# Project Instructions

Repository for the Udacity “Agentic AI Engineer with LangChain and LangGraph”
Nanodegree.

## Repository layout

The canonical course directories are:

- `01-langchain-agentic-ai-fundamentals/`
- `02-building-ai-agents-with-langgraph/`
- `03-advanced-agentic-ai-techniques/`

Within each course, put lesson notes in `notes/`, lesson exercises and their
supporting files in `exercises/`, and the graded course project with its data,
fixtures, notebooks, and assets in `project/`. Each course-level `README.md` is
the index for that course.

Use lowercase kebab-case for new directories and filenames, except established
conventional names such as `README.md` and Python import/package names. Prefix
ordered course and lesson material with two digits (`01-`, `02-`, and so on),
and preserve the official Udacity lesson order when it is known.

Preserve Udacity starter-code behavior and course content unless the user
explicitly asks for implementation changes. Keep course- or project-specific
dependency files beside the material they support, and do not change package
managers or dependency versions without an explicit request.

## Secrets and external services

Treat every `.env` file and credential as secret. Never display, print, stage,
or commit `.env` contents, API keys, or other credentials. Nested `.env` files
must remain ignored. Do not construct `.env.example` files from real secret
files; examples may contain placeholders only.

Do not run exercises, notebooks, tests, or applications that can call OpenAI,
another paid API, or any token-consuming external service without explicit
permission.

## Safe validation

The repository currently supports these offline, non-token-consuming checks:

- `git diff --check`
- `uv lock --check --offline`
- `git check-ignore -v <path-to-env-file>` (pass paths only; never inspect the
  files)

Also inspect `git status --short` and check changed relative Markdown links
before handing work back. Do not execute notebooks as validation.

## Commits

When asked to create a commit or draft a commit message, use the
`conventional-commits` skill.

Never commit credentials, ignored files, or add attribution trailers.
Do not amend, rebase, reset, or force-push without an explicit request.
