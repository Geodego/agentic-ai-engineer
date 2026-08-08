---
name: format-course-note
description: Format pasted plain-text course lessons as repository Markdown note files with a maintained table of contents. Use when asked to add, convert, append, or reformat a lesson or chapter from user-provided course content. Preserve the supplied content and do not research, summarize, or invent lesson material.
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

## Append content to an existing note

Require a user-supplied section title for every block of lesson text to append to an existing note. Treat this section title separately from the note's existing `#` lesson title. If the user does not supply the section title, stop and ask for it; do not derive or invent it from the pasted content.

Place the entire new content block under one numbered `##` heading containing the supplied section title. Determine its number from the last existing numbered `##` heading in document order and add one. Use `## 1. <section-title>` when the note has no numbered `##` heading.

For example, if the target note ends with `## 2. Chat history and prompt templates`, append the next supplied content block under:

```markdown
## 3. <supplied-section-title>
```

Nest headings within the supplied content beneath this new `##` parent and number them consistently. Do not merge the new block into an existing section unless the user explicitly requests that placement.

## Preserve the lesson content

- Preserve every item of source content and its meaning.
- Do not summarize, condense, omit, explain, expand, or invent material.
- Do not perform web research.
- Correct spelling or technical claims only when the user explicitly requests corrections.
- Apply minor punctuation normalization only when Markdown formatting requires it.
- Do not alter code unless the user explicitly requests a correction.
- Do not add tables, blockquotes, callouts, summaries, metadata, or frontmatter unless they appear in the source or the user requests them.

## Structure the Markdown

- Use exactly one `#` heading containing the supplied lesson title. When appending content, retain the existing `#` heading and do not add another one.
- Place an unnumbered `## Table of contents` immediately after the `#` lesson title.
- Under the table-of-contents heading, add one relative Markdown link for every subsequent heading in document order. Mirror the document hierarchy with nested list indentation, use each heading's exact visible text as the link label, and omit the `#` lesson title and the table-of-contents heading itself.
- Generate each fragment from the target heading using GitHub-style Markdown anchors: lowercase the text, remove punctuation, replace spaces with hyphens, and preserve the heading's numeric prefix. For example, `### 2.1 Building Stateful Interactions with LLMs` links as `[2.1 Building Stateful Interactions with LLMs](#21-building-stateful-interactions-with-llms)`.
- Rebuild the complete table of contents whenever creating a note or adding, removing, renaming, renumbering, or moving headings in an existing note. Do not append only the new entries.
- Convert major sections to sequentially numbered `##` headings: `## 1. ...`, `## 2. ...`, and so on. For content appended to an existing note, use its required supplied section title and the next number as defined above.
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
- preserving the original “Final Thoughts” section as a `###` subsection after “Messages,” before later major sections; and
- placing any future appended content under `## 3. <supplied-section-title>` because `## 2. Chat history and prompt templates` is currently its last numbered `##` heading; and
- rebuilding its table of contents so every current heading remains linked in document order.

Treat this file only as a structural example. Never copy its lesson content into another note.

## Update the course index

Update the relevant course `README.md` with one relative Markdown link to the note. Keep note links in numeric lesson order, do not create a duplicate entry, and preserve all existing progress text. Do not otherwise rewrite the course index.

## Validate and hand off

Before finishing:

1. Verify the heading hierarchy and sequential numbering.
2. Verify that the table of contents immediately follows the lesson title and contains one working, correctly nested link for every subsequent heading in document order.
3. When content was appended, verify that the user supplied its `##` section title and that its number is one greater than the last pre-existing numbered `##` heading.
4. Verify that every heading is followed by a blank line.
5. Verify that all code fences are closed and use appropriate language identifiers.
6. Verify that the target path and filename match the authoritative inputs and naming rules.
7. Verify that the course `README.md` contains one correct relative link in numeric lesson order and retains its progress text.
8. Run `git diff --check`.
9. Inspect `git status --short` and confirm that only intended files changed.

Do not stage or commit the result.
