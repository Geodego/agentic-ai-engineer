---
name: format-course-note
description: Format pasted plain-text course lessons as repository Markdown note files. Use when asked to add, convert, or reformat a lesson or chapter from user-provided course content. Preserve the supplied content and do not research, summarize, or invent lesson material.
---

# Format a Course Note

## Gather repository context

1. Read the root `AGENTS.md` completely.
2. Read the `README.md` in the user-supplied course directory completely.
3. Treat the user-supplied course directory, lesson number, lesson title, source text, and target filename, when supplied, as authoritative.
4. Do not consult the Udacity syllabus or the web to infer, verify, or change user-supplied lesson information.

Never infer or change an explicitly supplied lesson number. Format lesson numbers with two digits, such as `01` or `12`.

## Resolve the target safely

Place the note in the course's `notes/` directory. When the user supplies a target filename, use that filename as authoritative and do not silently rename it. Otherwise, create a lowercase kebab-case slug from the supplied lesson title and use:

```text
<course-directory>/notes/<two-digit-lesson-number>-<lesson-title-slug>.md
```

Check whether the resolved target exists before writing. If it exists, stop and request explicit permission before overwriting or restructuring it. Read the existing note first and preserve all personal annotations when applying an authorized update.

When updating an existing note, preserve each existing heading's relative position, scope, and associated content unless the user explicitly requests a structural change. Do not merge an existing section with newly supplied content merely because their headings match.

## Preserve the lesson content

- Preserve every item of source content and its meaning.
- Do not summarize, condense, omit, explain, expand, or invent material.
- Do not perform web research.
- Correct spelling or technical claims only when the user explicitly requests corrections.
- Apply minor punctuation normalization only when Markdown formatting requires it.
- Do not alter code unless the user explicitly requests a correction.
- Do not add tables, blockquotes, callouts, summaries, metadata, or frontmatter unless they appear in the source or the user requests them.

## Structure the Markdown

- Use exactly one `#` heading containing the supplied lesson title.
- Convert major sections to sequentially numbered `##` headings: `## 1. ...`, `## 2. ...`, and so on.
- Convert subsections to headings numbered under their parent: `### 1.1 ...`, `### 1.2 ...`, and so on.
- Use `####` only for components genuinely nested beneath a `###` subsection.
- Never skip heading levels.
- Leave exactly one blank line after every heading before subsequent content.
- Determine the level of sections such as “Final Thoughts,” “Summary,” and “Conclusion” from their position and scope. Use `##` only when the source presents the section as standalone major content; retain it as a subsection when it concludes a parent topic.
- Preserve distinct same-named sections when they belong to different scopes or when one is existing content and the other is newly supplied content.
- Convert suitable plain-text sequences into Markdown ordered or unordered lists without changing their wording or order.

## Format technical elements

- Use inline code for package names, classes, functions, methods, commands, parameters, and short code expressions.
- Do not format ordinary terminology as inline code.
- Put multiline code examples in fenced code blocks with the appropriate language identifier.
- Preserve code exactly unless the user explicitly requests corrections.
- Ensure every code fence is closed.

## Use the repository example

Use `01-langchain-agentic-ai-fundamentals/notes/01-creating-a-simple-langchain-application.md` as a read-only formatting reference. Do not modify it while formatting another note. Improve on its formatting pattern by:

- leaving a blank line after every heading;
- formatting identifiers such as `langchain-openai`, `HumanMessage`, and `invoke()` as inline code; and
- preserving the original “Final Thoughts” section as a `###` subsection after “Messages,” before later major sections.

Treat this file only as a structural example. Never copy its lesson content into another note.

## Update the course index

Update the relevant course `README.md` with one relative Markdown link to the note. Keep note links in numeric lesson order, do not create a duplicate entry, and preserve all existing progress text. Do not otherwise rewrite the course index.

## Validate and hand off

Before finishing:

1. Verify the heading hierarchy and sequential numbering.
2. Verify that every heading is followed by a blank line.
3. Verify that all code fences are closed and use appropriate language identifiers.
4. Verify that the target path and filename match the authoritative inputs and naming rules.
5. Verify that the course `README.md` contains one correct relative link in numeric lesson order and retains its progress text.
6. Run `git diff --check`.
7. Inspect `git status --short` and confirm that only intended files changed.

Do not stage or commit the result.
