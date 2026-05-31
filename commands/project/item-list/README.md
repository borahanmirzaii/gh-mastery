# `gh project item-list`

> **One-liner:** List (and optionally filter) the items on a GitHub Projects v2 board, with support for the native Projects filter syntax and JSON output for scripting.

## When you reach for it

Whenever you need to query a board programmatically — "show me everything assigned to me that isn't Done", "how many items are In Progress?", or "give me the node IDs of all open issues so I can pipe them into `item-edit`." In the solo-builder loop this is the *read side* of project management: you run it before editing statuses or archiving stale items. This repo's tracker board (Project #14) is a live, real-world target for read-only drills.

## Key flags

- `<number>` — the project number (positional argument, required). Find it in the URL or via `gh project list`.
- `--owner <login>` — who owns the project. Use `@me` for yourself, or an org login. **Required.**
- `--query <string>` — server-side filter using the [Projects filter syntax](https://docs.github.com/en/issues/planning-and-tracking-with-projects/customizing-views-in-your-project/filtering-projects). E.g. `"assignee:@me -status:Done"`. Only works on github.com and GHES 3.20+.
- `-L / --limit <n>` — max items to return (default 30). Raise this if the board has more items than 30.
- `--format json` — emit JSON instead of the default table. Nearly always paired with `--jq`.
- `--jq <expr>` — filter or transform the JSON output in-process. E.g. `'.items[] | .title'`.
- `--template <tmpl>` — format JSON as a Go template. Less common than `--jq` but available.

## Examples

```bash
# List all items in your project #1
gh project item-list 1 --owner @me

# List items in the gh-mastery board — real read-only target
gh project item-list 14 --owner borahanmirzaii

# Filter to items not yet Done
gh project item-list 14 --owner borahanmirzaii \
  --query "-status:Done"

# Filter to open issues assigned to yourself
gh project item-list 14 --owner borahanmirzaii \
  --query "assignee:@me is:issue is:open"

# Filter by label (useful for finding all kind:command items)
gh project item-list 14 --owner borahanmirzaii \
  --query "label:kind:command"

# Get raw JSON and extract just the titles
gh project item-list 14 --owner borahanmirzaii \
  --format json --jq '.items[].title'

# Get item node IDs (needed for item-edit / item-delete)
gh project item-list 1 --owner @me \
  --format json --jq '.items[] | {title: .title, id: .id}'

# Raise the limit when a board has more than 30 items
gh project item-list 14 --owner borahanmirzaii --limit 100

# Count items in a project
gh project item-list 14 --owner borahanmirzaii \
  --format json --jq '.items | length'
```

## Gotchas

- **`--query` is server-side and version-gated.** The Projects filter syntax (e.g. `assignee:@me -status:Done`) is processed on the server and requires github.com or GHES 3.20+. On older GHES, the flag is silently accepted but completely ignored — you get all items back. Always validate the filter is actually working by checking the count with and without it.

- **Default limit is 30.** Boards with more than 30 items silently truncate. Always pass `-L 100` (or higher) when counting items or fetching IDs for scripting. There is no pagination cursor in the CLI output — if you need more than your limit, raise it or use `gh api graphql` directly.

- **`--owner` is required.** There is no project-number-only shorthand; you must always specify who owns the board.

- **Item IDs are GraphQL node IDs, not row numbers.** The `id` field in JSON output (e.g. `PVTI_…`) is what `item-edit`, `item-delete`, and `item-archive` expect. The visible row number in the Projects UI is not exposed by this command.

- **Draft issues show up in item-list.** Items created with `item-create` (draft issues — not linked to a real issue) appear in `item-list` alongside linked issues and PRs. Their `content` field in JSON is `null`; their `title` is set directly.

- **`--query` label filter syntax uses `label:<name>`, not `--label`.** The filter is a string argument, not a separate flag. Labels with colons in the name (like `kind:command`) work fine: `--query "label:kind:command"`.

## Concepts

- [Projects v2 data model](../../../concepts/projects-v2-data-model.md) — how items, fields, and views relate in the GraphQL schema (forthcoming in Milestone 3).

## Sources

- Manual: https://cli.github.com/manual/gh_project_item-list
- Local: `gh project item-list --help` (gh 2.92.0)
