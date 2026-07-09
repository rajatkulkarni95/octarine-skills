---
name: octarine-notes
description: "Create, edit, and organize notes in an Octarine workspace by writing Markdown files directly to disk. Use when the user asks an agent to save notes, daily notes, meeting notes, plans, research, attachments, tags, wiki links, templates, or any other content into Octarine."
---

# Octarine Notes

Octarine is a local-first note-taking app. Notes are plain Markdown files in a user-selected workspace, and Octarine watches the filesystem for changes. To save content into Octarine, write files into the workspace; do not use an API.

## Workspace Discovery

Before creating or editing notes, identify the Octarine workspace.

1. Check whether the user has already provided a workspace path in the current conversation, agent memory, `AGENTS.md`, `CLAUDE.md`, or another global config.
2. Walk up from the current working directory. If a parent contains `.octarine/`, treat that parent as the workspace root.
3. If no workspace is known, ask the user for the Octarine workspace path.
4. Validate the workspace before writing. The directory should exist and contain `.octarine/`.
5. After the user confirms a valid workspace path, persist it in global agent memory or global config when available so future sessions can reuse it.

Do not guess a default workspace path. Common locations such as `~/Documents/Octarine/` are only suggestions to help the user find the folder.

## Workspace Layout

Typical workspace structure:

```text
<workspace>/
├── *.md
├── <folder>/*.md
├── Daily/
│   └── Weekly/
├── .templates/
├── .attachments/
├── .files/
└── .octarine/
```

Rules:

- Never modify `.octarine/`.
- Avoid custom dot-prefixed folders unless the user has enabled dot-folders in Octarine.
- Write notes as `.md` files.
- Put images and media in `.attachments/`.
- Put imported non-Markdown files in `.files/`.
- Avoid `node_modules` and `.git` folder names; Octarine ignores them.

## Writing Notes

Write Markdown files directly into the workspace.

```markdown
# My Note Title

Content goes here.
```

Save as:

```text
<workspace>/My Note Title.md
```

For notes in folders, create directories as needed:

```text
<workspace>/Projects/Alpha/design-doc.md
<workspace>/Areas/Health/workout-log.md
```

Use safe filenames:

- Use `.md` extension.
- Avoid `< > : " / \ | ? *` inside a filename segment.
- Spaces are supported.
- If a same-name file exists and the user did not ask to overwrite it, append a number such as `Note 1.md`.

## Frontmatter

Octarine supports YAML frontmatter.

```markdown
---
title: Meeting Notes
status: draft
date: 2026-07-09
pinned: true
tags:
  - meetings
  - engineering
---

# Meeting Notes
```

Supported value types include strings, numbers, booleans, dates, and YAML lists. Useful special properties:

- `tags`: YAML list of strings.
- `pinned`: boolean.
- `locked`: boolean.

Avoid custom properties beginning with `oct.` because they are reserved.

## Daily And Weekly Notes

Daily notes must live in `Daily/` and use ISO dates:

```text
<workspace>/Daily/YYYY-MM-DD.md
```

Weekly notes must live in `Daily/Weekly/` and use ISO week format:

```text
<workspace>/Daily/Weekly/YYYY-[W]WW.md
```

Examples:

```text
Daily/2026-07-09.md
Daily/Weekly/2026-W28.md
```

Do not add a date heading unless the user asks for one; Octarine displays the date in the UI.

## Octarine Markdown

Octarine supports standard Markdown plus these conventions:

- Tags: write `#tag`, `#nested/tag`, or `#work/engineering`.
- Wiki links: use `[[Note Name]]` for root notes and `[[Folder/Note Name]]` for notes in folders. Use the full path relative to the workspace root, without `.md`.
- Tasks: use `- [ ]` and `- [x]`.
- Callouts: use blockquotes such as `> [!INFO]`, `> [!TIP]`, `> [!WARNING]`, `> [!ERROR]`, or `> [!SUCCESS]`.
- Math: use `$inline$` or block math with `$$` delimiters on separate lines.
- Diagrams: use fenced `mermaid` code blocks.
- Underline: use single tildes, for example `~underlined~`.
- Highlight: use `==highlighted==`; color-prefixed highlights such as `==🟡yellow highlight==` are supported.
- Colored text: use color-prefixed single tildes such as `~🔵blue text~`.

For tables, standard Markdown tables are supported. To set column widths, place an Octarine column-width comment immediately before the table:

```markdown
<!--octarine-table-cols:150,250,200-->
| Feature | Syntax |
|---------|--------|
| Bold | `**text**` |
```

## Attachments

Save local images and videos into `.attachments/`, then reference them with wikilinks.

```markdown
[[screenshot.png]]
[[screenshot.png|500]]
[[demo.mp4]]
[[demo.mp4|600]]
```

Use standard Markdown for external images:

```markdown
![Alt text](https://example.com/image.png)
![Alt text|300](https://example.com/image.png)
```

Recognized media formats include `.png`, `.jpg`, `.jpeg`, `.gif`, `.svg`, `.webp`, `.mp4`, `.webm`, and `.mov`.

## Templates

Templates live in `.templates/` and may use these variables:

- `{{title}}`
- `{{date}}`
- `{{date:FORMAT}}`
- `{{time}}`
- `{{time:FORMAT}}`

Date and time formats use date-fns syntax, for example `{{date:MMMM d, yyyy}}` or `{{time:h:mm a}}`.

## Common Operations

| Operation | Action |
|-----------|--------|
| Create a note | Write a `.md` file in the workspace |
| Update a note | Edit or overwrite the existing `.md` file |
| Add a daily note | Write `Daily/YYYY-MM-DD.md` |
| Add a weekly note | Write `Daily/Weekly/YYYY-[W]WW.md` |
| Add an image | Save to `.attachments/` and reference `[[image.png\|500]]` |
| Add a file attachment | Save to `.files/` |
| Add metadata | Use YAML frontmatter |
| Add links | Use workspace-relative `[[wiki links]]` |
| Add tasks | Use Markdown task checkboxes |

## Completion Checklist

After writing files:

1. Confirm the target path is inside the workspace.
2. Confirm no `.octarine/` file was modified.
3. Confirm Markdown syntax is valid enough for the intended content.
4. Tell the user the exact note path created or edited.
