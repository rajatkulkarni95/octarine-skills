# Octarine CLI Command Reference

Every command accepts these global flags:

| Flag | Purpose |
|------|---------|
| `--workspace <name\|uuid\|path>` | Target workspace (env `OCTARINE_WORKSPACE`) |
| `--json` / `--jsonl` / `--format table\|plain\|json\|jsonl` | Output format |
| `--fields a,b` | Project selected output fields (JSON/JSONL) |
| `--non-interactive` | Reject prompts |
| `--quiet`, `--verbose`, `--no-color`, `--no-progress` | Output control |
| `--request-id <id>` | Echoed back in `meta.requestId` |
| `--app-store <path>` | Override app store (env `OCTARINE_APP_STORE`), e.g. for a dev/staging store |
| `--database <path>` | Override shared SQLite DB (env `OCTARINE_DATABASE`) |

`<PATH>` arguments are workspace-relative note paths including `.md`. `--file -` reads JSON/YAML from stdin.

## workspace

| Command | Notes |
|---------|-------|
| `workspace list` | Workspaces registered in the Octarine app |
| `workspace show [ID]` | Workspace record |
| `workspace info [ID]` | Content statistics (notes, folders, …) |
| `workspace doctor [ID]` | Check metadata and structure |

## note

| Command | Options |
|---------|---------|
| `note create <PATH>` | `--content`, `--stdin`, `--file`, `--property k=v`…, `--tag t`…, `--if-missing`, `--unique` |
| `note read <PATH>` | `--lines start:end`, `--from-line`, `--to-line`, `--section <heading>`, `--max-bytes`, `--max-chars`, `--body-only`, `--frontmatter-only` |
| `note info <PATH>` | File information |
| `note list [FOLDER]` | Notes in a folder |
| `note random` | `--folder` |
| `note wordcount <PATH>` | Words and characters |
| `note outline <PATH>` | Heading outline |
| `note properties <PATH>` | Frontmatter properties |
| `note update <PATH>` | Replace content. `--content`, `--stdin`, `--revision` |
| `note append <PATH>` / `note prepend <PATH>` | `--content`, `--stdin`, `--revision` |
| `note move <SRC> <DEST>` | Move without rewriting links |
| `note rename <SRC> <DEST>` | Rename and update links. `--dry-run`, `--no-update-links` |
| `note duplicate <SRC> [DEST]` | `--unique` |
| `note task list <PATH>` | Tasks with `id` (line), `checked`, `text` |
| `note task set <PATH> <ID> <open\|done>` | `--revision` |
| `note trash <PATH>` | Move to system trash |

## folder

| Command | Options |
|---------|---------|
| `folder create <PATH>` | |
| `folder list [PATH]` | |
| `folder move <SRC> <DEST>` / `folder rename <SRC> <DEST>` | `--dry-run` |
| `folder duplicate <SRC> [DEST]` | `--unique` |
| `folder trash <PATH>` | |
| `folder config get <PATH>` / `folder config set <PATH> --file <FILE>` | Per-folder configuration (JSON/YAML) |

## search

`search <QUERY>` takes `--limit` (default 20), `--offset`, `--title`, `--case-sensitive`, `--whole-word`, `--regex`, `--tag <TAG>`, `--property <k=v>`, `--in-path <PREFIX>`, `--context-lines <N>` and `--max-output-bytes <N>`.

## link

| Command | Purpose |
|---------|---------|
| `link list [NOTE]` | Outgoing wikilinks |
| `link backlinks <TARGET>` | Notes linking to target |
| `link mentions <TARGET>` | Unlinked mentions of target |
| `link replace <OLD> <NEW>` | Rewrite links workspace-wide |
| `link unresolved` | Links with no target note |
| `link orphans` | Notes with no incoming links |
| `link deadends` | Notes with no outgoing links |

## property

| Command | Options |
|---------|---------|
| `property list <PATH>` | |
| `property get <PATH> <KEY>` | |
| `property set <PATH> <KEY> <VALUE>` | `--revision` |
| `property unset <PATH> <KEY>` | `--revision` |
| `property definitions list` / `show <NAME>` | Workspace property schema |
| `property definitions define <NAME> --type <TYPE>` | Types: `string`, `text`, `tag`, `number`, `integer`, `float`, `boolean`, `checkbox`, `list`, `array`, `tags`, `date`, `datetime`, `external_files`, `tasks` |
| `property definitions put --file <FILE>` | Replace all definitions (JSON array) |
| `property definitions delete <NAME>` / `validate` | |

## task / tag

| Command | Options |
|---------|---------|
| `task list` | `--folder`, `--todo` or `--done` |
| `tag list` | `--sort-count` |

## inbox

| Command | Notes |
|---------|-------|
| `inbox capture [TEXT]` | Text or stdin. `--priority` (default `none`), `--tag` |
| `inbox list` | |
| `inbox status <ITEM> <open\|in_progress\|done>` | |
| `inbox priority <ITEM> <PRIORITY>` | |
| `inbox snooze <ITEM> <UNTIL>` / `inbox unsnooze <ITEM>` | |
| `inbox tag <ITEM> add\|remove <TAG>` / `inbox tag <ITEM> list` | |
| `inbox append-to-note <ITEM> <NOTE>` | |
| `inbox append-to-daily <ITEM>` | `--date YYYY-MM-DD` (default today) |
| `inbox promote <ITEM> [DEST]` | Turn the item into a note |
| `inbox trash <ITEM>` | |
| `inbox doctor` | |

## view / note-type / theme

All three share the same subcommands:

| Command | Purpose |
|---------|---------|
| `list`, `get <ID>`, `schema` | Inspect |
| `create <NAME>`, `clone <ID> --name <NAME>` | Create |
| `update <ID> --file <FILE>`, `put --file <FILE>` | Replace from JSON/YAML |
| `set <ID> <VARIABLE> <VALUE>`, `unset <ID> <VARIABLE>` | Edit one field |
| `validate [ID]` | Validate one or all |
| `delete <ID>` | Remove |

Unknown JSON fields are preserved. Run `<resource> schema` before authoring a spec.

## metadata / schema

| Command | Purpose |
|---------|---------|
| `metadata list\|get <RESOURCE>` | Read a `.octarine/<RESOURCE>` JSON file |
| `metadata put <RESOURCE> --file <FILE>` | Write it |
| `schema list` / `schema show <RESOURCE>` | Persisted schema contracts: `note`, `property`, `view`, `type`, `theme`, `inbox` |

Prefer the typed commands (`view`, `note-type`, `theme`, `property definitions`) over raw `metadata put`.

## Other

| Command | Purpose |
|---------|---------|
| `doctor` | Workspace integrity check |
| `export --output <PATH>` | Export notes and metadata |
| `capabilities` | Feature list for this CLI version |
| `update --check` / `update --yes` | Signed self-update |
| `completion <bash\|zsh\|fish\|elvish\|powershell>` | Shell completions |
