---
name: conventional-commits
description: Create or draft Conventional Commit messages for this repository. Use when asked to commit changes, prepare a commit, split changes into commits, or write a commit message.
---

# Conventional commits

Follow this workflow when creating or drafting commits.

## Staged changes

Treat the staging area as the complete source of truth:

1. Inspect `git status --short` and `git diff --cached`.
2. If nothing is staged, say there are no staged changes and stop.
3. Consider only staged changes when choosing the message.
4. Never stage changes or run `git add -p`. If the staged diff is not atomic,
   explain why it should be split and ask the user to restage it.

Do not include unstaged or untracked files. Even for broad requests such as
"commit the remaining files" or "commit uncommitted files," commit only what
the user staged.

Never stage or commit `.env`, credentials, API keys, or files matching a
`.gitignore` rule. Never add attribution trailers such as `Co-Authored-By` or
"Generated with." Do not amend, rebase, reset, or force-push without an
explicit request.

### Create a commit

1. Inspect the staged changes.
2. Check that they form one logical change and do not contain prohibited
   material.
3. Write one message following the rules below.
4. Commit with that message.
5. Report what was committed.

### Draft a message only

Return only the commit message text, with no preamble or commentary.

## Format

Use Conventional Commits:

```text
<type>(<scope>): <imperative summary>

<optional body>

<optional footer>
```

Use lowercase after the colon and no trailing period on the subject.

## Atomicity

Create one commit for one logical change. If the subject needs the word "and,"
the staged diff likely contains two concerns and should be split. Difficulty
naming a commit usually indicates an incoherent diff rather than a missing
word.

Do not alter the staging area to resolve an atomicity problem. Ask the user to
restage the intended change.

## Types

Choose by dominant intent:

- `feat`: add an agent, graph, tool, chain, workflow, or capability
- `fix`: correct incorrect behavior
- `refactor`: restructure code with no behavior change
- `perf`: improve speed, token use, or cost with no behavior change
- `test`: staged changes only affect tests or test helpers
- `docs`: README, AGENTS, notebooks-as-documentation, guides, or comments only
- `style`: formatting only, no logic change
- `build`: dependencies, `pyproject.toml`, `uv.lock`, or packaging
- `ci`: CI configuration and workflows
- `chore`: repository maintenance, ignores, metadata, or housekeeping
- `revert`: revert a previous commit

`refactor` means no behavior change; use `feat` or `fix` if behavior changes.
For deletions, match the type to the intent: removing a feature is `feat`,
removing dead code is `refactor`, and removing stray files is `chore`. Do not
default to `chore`.

## Scopes

- Derive the scope from changed paths.
- Use the smallest stable functional area.
- Use lowercase with no spaces.
- Omit the scope if it is unclear or the change spans the whole repository.
- Ignore noise path parts such as `tests`, `test`, `__init__.py`, `conftest.py`,
  and test file stems.

Use these provisional scopes until the directory structure settles:

```text
agents, graphs, tools, memory, retrieval, prompts, evals,
notebooks, project_instructions, docs
```

Course projects use their own scope, for example `project1`.

## Subject

- Use imperative mood so the subject completes "If applied, this commit will
  ___."
- Target 50 characters and never exceed 72.
- Describe the main behavior, content, or maintenance change.
- Do not mention unstaged or ignored files.

```text
add retry logic to the tool node          <- correct
adds retry logic to the tool node         <- wrong mood
added retry logic to the tool node        <- wrong mood
retry logic                               <- not a sentence
```

Reject `wip`, `fix`, `update`, `stuff`, `.`, a bare ticket number, or any
subject that states something happened without saying what.

## Body

Add a body only when the subject needs context.

- Separate it from the subject with one blank line.
- Wrap at 72 characters and use at most three lines.
- Explain why, not what; the diff already shows what changed.
- Mention generated or output files only if intentionally staged.

```text
fix(graphs): prevent infinite loop in the research agent

The conditional edge returned to the planner whenever the tool call
produced empty results, so an unanswerable query cycled until the
recursion limit. Route to the summariser instead.
```

## Footer

Optionally use `Refs: #12` for issue references or a `BREAKING CHANGE:`
paragraph for incompatible changes.
