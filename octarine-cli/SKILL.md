---
name: octarine-cli
description: "Read, search, create, edit, and organize notes in an Octarine workspace with the `octarine` command-line tool. Use when the `octarine` binary is available and the user asks an agent to find or read notes, search the workspace, update notes or properties, manage tasks, tags, links, folders, or the Inbox, or inspect workspace health in Octarine."
---

# Octarine CLI

`octarine` is a standalone binary that operates directly on an Octarine workspace's Markdown files. It does not need the desktop app running and makes no network calls (except `octarine update`). Prefer it over hand-editing files: writes are atomic, revision-checked, locked per workspace, and link-aware.

## Preflight

1. Run `octarine --version`. If the command is missing, tell the user the Octarine CLI is not installed. If the `octarine-notes` skill is available, fall back to writing Markdown files directly.
2. Run `octarine workspace list` to see the workspaces registered in the Octarine app.
3. Pick the workspace and pass it on every call with `--workspace <name|uuid|path>`. Alternatively, export `OCTARINE_WORKSPACE` once.

Workspace resolution order is `--workspace`, then `OCTARINE_WORKSPACE`, then the nearest ancestor directory containing `.octarine/`, then the only workspace in the app store. If several exist and none is selected, the CLI exits with `WORKSPACE_REQUIRED`. Ask the user which one to use; do not guess.

The CLI cannot register or remove workspaces. Those operations belong to the desktop app.

## Agent Conventions

- Always use `--json` (one response) or `--jsonl` (streamed rows) when parsing output. The default table output is for humans.
- Use `--fields a,b` to keep responses small, e.g. `--fields path,revision`.
- Pass `--non-interactive` so the CLI never waits on a prompt.
- Bound large reads and searches: `search --limit N --max-output-bytes N`, `note read --max-chars N` or `--lines 1:80`.
- Paths are workspace-relative and include `.md`, e.g. `Projects/Alpha.md`. Paths that escape the workspace are rejected.
- Pipe long content through stdin with `--stdin` (notes) or `--file -` (JSON/YAML resources) instead of shell-quoting it.

### Response envelope

```json
{"schemaVersion":1,"ok":true,"data":{...},"meta":{"truncated":false}}
{"schemaVersion":1,"ok":false,"error":{"code":"NOT_FOUND","message":"...","retryable":false},"meta":{}}
```

Check `ok` and `meta.truncated`. If output was truncated, narrow the query or page with `--offset`.

| Exit | Error code | Meaning |
|------|------------|---------|
| 0 | — | Success |
| 1 | `RUNTIME_ERROR` | Other failure |
| 2 | `INVALID_PROPERTY` | Bad property value |
| 3 | `NOT_FOUND` | Note, folder, or resource missing |
| 4 | `ALREADY_EXISTS`, `REVISION_CONFLICT` | Destination exists, or the note changed since you read it |
| 5 | `WORKSPACE_REQUIRED`, `PATH_ESCAPE` | No workspace selected, or path outside workspace/protected |
| 7 | — | Schema or metadata problem |

## Safe Editing With Revisions

Every note read or write returns `revision` (`sha256:...`). Pass it back with `--revision` on `note update/append/prepend`, `note task set`, and `property set/unset`. If someone edited the note in the meantime (the user in the app, another agent), the write fails with `REVISION_CONFLICT` instead of clobbering their change.

```bash
rev=$(octarine --workspace Notes note read Projects/Alpha.md --json --fields revision | jq -r .data.revision)
printf '%s\n' "## Update" "Shipped v2." | octarine --workspace Notes note append Projects/Alpha.md --stdin --revision "$rev" --json
```

On `REVISION_CONFLICT`, re-read the note, reapply the change to the fresh content, and retry. Never drop `--revision` just to force the write.

## Core Workflows

### Find and read

```bash
octarine --workspace Notes search "launch plan" --limit 10 --context-lines 1 --json
octarine --workspace Notes search launch --title --json             # match titles only
octarine --workspace Notes search 'v\d+\.\d+' --regex --in-path Projects --json
octarine --workspace Notes search q3 --tag work --property status=active --json
octarine --workspace Notes note list Projects --json
octarine --workspace Notes note read Projects/Alpha.md --section "Decisions" --json
octarine --workspace Notes note read Projects/Alpha.md --frontmatter-only --json
octarine --workspace Notes note outline Projects/Alpha.md --json
```

### Create and edit

```bash
octarine --workspace Notes note create "Meetings/2026-10-11 Sync.md" \
  --stdin --tag meetings --property status=draft --json < body.md
octarine --workspace Notes note create Inbox/Idea.md --content "..." --unique --json   # add a suffix if the name is taken
octarine --workspace Notes note create Daily/2026-10-11.md --if-missing --json        # no-op if it already exists
octarine --workspace Notes note update Projects/Alpha.md --stdin --revision "$rev" --json   # replace the whole body
octarine --workspace Notes note prepend Projects/Alpha.md --content "> [!INFO] Archived" --revision "$rev" --json
```

`--property key=value` and `--tag name` can be repeated. Note content follows Octarine Markdown: `[[Folder/Note]]` wikilinks, `#tags`, `- [ ]` tasks, `> [!INFO]` callouts. Daily notes live at `Daily/YYYY-MM-DD.md`.

### Organize

```bash
octarine --workspace Notes note rename Old.md Projects/New.md --dry-run --json   # preview link rewrites
octarine --workspace Notes note rename Old.md Projects/New.md --json             # renames and updates [[links]]
octarine --workspace Notes note move Draft.md Archive/Draft.md --json            # move without rewriting links
octarine --workspace Notes note duplicate Templates/Weekly.md --unique --json
octarine --workspace Notes folder create Projects/Beta --json
octarine --workspace Notes folder rename Projects/Beta Projects/Gamma --dry-run --json
```

Use `note rename` (not `move`) when a note's links should follow it.

### Properties

```bash
octarine --workspace Notes property list Projects/Alpha.md --json
octarine --workspace Notes property set Projects/Alpha.md status active --revision "$rev" --json
octarine --workspace Notes property unset Projects/Alpha.md due --revision "$rev" --json
octarine --workspace Notes property definitions list --json
octarine --workspace Notes property definitions define Priority --type number --json
```

Never use property names starting with `oct.`; they are reserved.

### Tasks, tags, links

```bash
octarine --workspace Notes task list --todo --folder Projects --json
octarine --workspace Notes note task list Projects/Alpha.md --json       # returns task ids
octarine --workspace Notes note task set Projects/Alpha.md 8 done --revision "$rev" --json   # state: open|done
octarine --workspace Notes tag list --sort-count --json
octarine --workspace Notes link backlinks "Projects/Alpha" --json
octarine --workspace Notes link unresolved --json        # [[links]] with no target note
octarine --workspace Notes link orphans --json           # notes nothing links to
octarine --workspace Notes link replace "Old Name" "Projects/New Name" --json
```

A task's id comes from `note task list`. It is a line number, so re-list the tasks after any edit that shifts lines.

### Inbox

```bash
echo "Call Sam about invoices" | octarine --workspace Notes inbox capture --priority high --tag finance --json
octarine --workspace Notes inbox list --json
octarine --workspace Notes inbox status INB-3 in_progress --json   # open | in_progress | done
octarine --workspace Notes inbox snooze INB-3 2026-10-14 --json
octarine --workspace Notes inbox tag INB-3 add urgent --json
octarine --workspace Notes inbox append-to-daily INB-3 --json      # today's Daily note; --date YYYY-MM-DD to override
octarine --workspace Notes inbox append-to-note INB-3 Projects/Alpha.md --json
octarine --workspace Notes inbox promote INB-3 Projects/Invoices.md --json
```

Inbox items are referenced by id (`INB-n`).

## Destructive Operations

`note trash`, `folder trash`, `inbox trash`, `view|note-type|theme delete`, `property definitions delete`, `link replace`, and `note update` (a full overwrite) change or remove user content. Confirm with the user before running them unless they explicitly asked for that exact action. Trash commands move content to the system trash, where it can be recovered.

Prefer `--dry-run` where available (`note rename`, `folder move`, `folder rename`) and show the user the result before applying.

## Reference

For the full command list (views, note types, themes, metadata, schemas, export, doctor, updates, completions), read [references/commands.md](references/commands.md). `octarine <command> --help` is authoritative for the installed version, and `octarine capabilities --json` lists the features that version supports.

## Completion Checklist

1. Every write used `--json` and returned `"ok": true`.
2. Edits to existing notes passed `--revision` from a fresh read.
3. No destructive command ran without user confirmation.
4. Tell the user the exact note paths created or changed.
