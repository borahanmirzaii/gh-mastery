# `gh project`

> **One-liner:** Manage GitHub Projects v2 — create, list, view, edit, and drive items and fields from the command line.

## When you reach for it

Whenever you need to interact with a Projects v2 board without opening a browser. In the solo-builder loop, this is where you move issues between columns ("Todo" → "In Progress"), query what's outstanding, add a freshly-created issue to the tracker board, or automate field updates as part of CI.

This repo uses it directly: **Project #14** (`gh-mastery`) tracks every command-group issue. The drills below target that real board for read-only queries and a disposable sandbox board for mutations.

## Subcommands

| Subcommand | Purpose | Node? |
|---|---|---|
| `gh project list` | List all projects for an owner | inline |
| `gh project create` | Create a new project | inline |
| `gh project view` | View a project in the terminal or browser | inline |
| `gh project edit` | Edit title, description, readme, or visibility | inline |
| `gh project close` | Close (or reopen) a project | inline |
| `gh project delete` | Permanently delete a project | inline |
| `gh project copy` | Copy a project, optionally including draft issues | inline |
| `gh project link` | Link a project to a repo or team | inline |
| `gh project unlink` | Unlink a project from a repo or team | inline |
| `gh project mark-template` | Mark (or unmark) a project as an org template | inline |
| `gh project item-list` | List items in a project with optional filter query | [→ `item-list/`](./item-list/) |
| `gh project item-add` | Add an existing issue or PR to a project | inline |
| `gh project item-create` | Create a draft issue item directly in a project | inline |
| `gh project item-edit` | Update a field value on a project item | inline |
| `gh project item-delete` | Delete a project item by its node ID | inline |
| `gh project item-archive` | Archive (or unarchive) a project item | inline |
| `gh project field-list` | List all fields defined on a project | [→ `field-list/`](./field-list/) |
| `gh project field-create` | Add a new field (text, number, date, single-select) | inline |
| `gh project field-delete` | Delete a field by its node ID | inline |

## Key flags

- `--owner <login>` — specifies who owns the project. Use `@me` for your own user account. **Required** by most subcommands; the only exception is when you're inside a repo whose default project is configured. For org projects, pass the org login.
- `--format json` — emit raw JSON instead of the default table. Pair with `--jq` or `--template` for scripting.
- `--jq <expr>` — filter or reshape the JSON output with a jq expression inline; no separate `jq` invocation needed.
- `-L / --limit <n>` — maximum items/fields/projects to fetch (default 30). Raise this when a board has more rows than the default page.
- `-w / --web` — open the result in your browser instead of printing it (on `list` and `view`).
- `--closed` — include closed projects in `list` output (omitted by default).
- `--query <string>` — on `item-list`, filter using the Projects filter syntax (e.g. `"assignee:@me is:issue -status:Done"`). Requires github.com or GHES 3.20+.
- `--title <string>` — name for create/edit operations.
- `--visibility PUBLIC|PRIVATE` — toggle project visibility on `edit`.
- `--undo` — reverses `close` (reopens) or `mark-template` (unmarks).
- `--id <node-id>` — used with item/field operations that target a specific object by its GraphQL node ID (not the human-readable number).
- `--single-select-option-id <id>` — on `item-edit`, set a single-select field to a specific option. Get option IDs via `field-list --format json`.
- `--data-type TEXT|SINGLE_SELECT|DATE|NUMBER` — required for `field-create`.

## Examples

```bash
# List all projects you own
gh project list --owner @me

# View project #14 (gh-mastery tracker) in the terminal
gh project view 14 --owner borahanmirzaii

# Open project #14 in the browser
gh project view 14 --owner borahanmirzaii --web

# Create a disposable sandbox project
gh project create --owner @me --title "zz-project-sandbox"

# List items in the gh-mastery board that are not yet Done
gh project item-list 14 --owner borahanmirzaii \
  --query "-status:Done" --format json --jq '.items[] | .title'

# Add issue #7 from gh-mastery to a project by URL
gh project item-add 1 --owner @me \
  --url https://github.com/borahanmirzaii/gh-mastery/issues/7

# Create a draft issue item directly on a board (no linked issue)
gh project item-create 1 --owner @me \
  --title "zz-project-draft-item" --body "Quick capture"

# Get all field IDs for project #14 as JSON
gh project field-list 14 --owner borahanmirzaii \
  --format json --jq '.fields[] | {name: .name, id: .id, type: .type}'

# Create a custom text field on your sandbox project
gh project field-create 1 --owner @me \
  --name "Sprint goal" --data-type TEXT

# Update the Status of an item (using node IDs from item-list + field-list)
gh project item-edit \
  --id <item-node-id> \
  --field-id <status-field-id> \
  --project-id <project-node-id> \
  --single-select-option-id <option-id>

# List projects including closed ones — useful when hunting an archived board
gh project list --owner @me --closed
```

## Gotchas

- **Projects v2 only.** `gh project` does not interact with "classic" projects (the old column-board style). Classic projects are deprecated; v2 is the current system. If you see a project on github.com that `gh project` can't find, it's likely classic.

- **`--owner` is almost always required.** Unlike `gh repo` (which infers from the current directory), `gh project` usually needs an explicit `--owner`. Without it most subcommands error out. Use `--owner @me` for personal projects, `--owner <org-name>` for org projects.

- **Org projects require the `project` OAuth scope plus `read:org`.** `gh auth status` shows your current scopes. If `field-list` or `item-list` returns an auth error on an org project, run `gh auth refresh -s project,read:org`.

- **Built-in Status field options (Todo / In Progress / Done) cannot be renamed via `gh project field-edit` — it covers only custom fields.** To rename or add Status options you must use GraphQL:

  ```bash
  # Step 1 — get the field ID
  FIELD_ID=$(gh project field-list <num> --owner @me --format json \
    --jq '.fields[] | select(.name=="Status") | .id')

  # Step 2 — get the project node ID
  PROJECT_ID=$(gh project view <num> --owner @me --format json --jq '.id')

  # Step 3 — mutation (rename "Todo" to "Backlog", supply the option's existing ID)
  gh api graphql -f query='
    mutation($pid: ID!, $fid: ID!, $oid: String!, $name: String!) {
      updateProjectV2Field(input: {
        projectId: $pid,
        fieldId: $fid,
        singleSelectOptions: [{id: $oid, name: $name, color: GRAY, description: ""}]
      }) { projectV2Field { ... on ProjectV2SingleSelectField { name options { id name } } } }
    }' \
    -F pid="$PROJECT_ID" -F fid="$FIELD_ID" \
    -F oid="<existing-option-id>" -F name="Backlog"
  ```

  The option IDs are in the `field-list --format json` output under `.fields[].options[].id`. Note: single-select field node IDs use the `PVTSSF_` prefix (not `PVTF_`) — use the exact value from `field-list --format json`.

- **Item and field operations use GraphQL node IDs, not human-readable numbers.** `--id`, `--field-id`, `--project-id`, and `--single-select-option-id` all expect the opaque `PVT_…` / `PVTF_…` / `PVTI_…` node IDs you get from `--format json`. Save them into shell variables.

- **`gh project item-edit` changes exactly one field per invocation.** Updating multiple fields requires multiple calls.

- **`--query` on `item-list` is a server-side filter** (Projects filter syntax, not `jq`). It requires github.com or GHES 3.20+. On older GHES instances, the flag is silently ignored and all items are returned.

- **`mark-template` is org-only** — the `--owner` must be an org login, not `@me`. Running it on a personal project returns an error.

- **`gh project copy` copies field structure but not Status option colours** — verify the board visually after copying if colour coding matters.

## Concepts

- [Projects v2 data model](../../concepts/projects-v2-data-model.md) — how projects, items, fields, and views relate in the GraphQL schema (forthcoming in Milestone 3).

## Sources

- Manual: https://cli.github.com/manual/gh_project
- Local: `gh project --help` (gh 2.92.0)
