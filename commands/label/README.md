# `gh label`

> **One-liner:** Create, list, edit, delete, and clone labels across GitHub repositories.

## When you reach for it

Labels are the backbone of triage (= the process of sorting and prioritizing) — you reach for `gh label` at three moments:

1. **Standing up a new repo** — `gh label clone` copies an entire label set from an existing repo so you don't retype 20 labels by hand.
2. **Running a bootstrap script** — `gh label create --force` upserts (= creates if absent, updates if present) labels idempotently (= safely re-runnable), so your setup script works on both a fresh repo and one that already has some labels.
3. **Ongoing taxonomy hygiene** — editing color/description, searching for a label, or deleting stale ones.

In the solo-builder loop this fits between repo creation and filing the first issues: labels must exist before you assign them.

## Subcommands

All five subcommands are shallow — none earns its own promoted node.

| Subcommand | Purpose | Node? |
|---|---|---|
| `gh label clone <source-repo>` | Copy every label from `<source-repo>` into the current (or `--repo`) repo | inline |
| `gh label create <name>` | Create a new label (or upsert with `--force`) | inline |
| `gh label delete <name>` | Delete a label from the repo | inline |
| `gh label edit <name>` | Change a label's color, description, or name | inline |
| `gh label list` | List all labels; supports search, sort, JSON output | inline |

## Key flags

### `gh label create`

- `-c`, `--color <hex>` — 6-character hex color (e.g. `E99695`). Omit to get a random color.
- `-d`, `--description <text>` — Short description shown in the UI.
- `-f`, `--force` — **Upsert**: if the label already exists, update its color and description instead of erroring. Essential for idempotent bootstrap scripts.

### `gh label clone`

- `-f`, `--force` — Overwrite labels that already exist in the destination repo. Without `--force`, existing labels are skipped.

### `gh label edit`

- `-c`, `--color <hex>` — Change the label's color.
- `-d`, `--description <text>` — Change the label's description.
- `-n`, `--name <new-name>` — Rename the label.

### `gh label delete`

- `--yes` — Skip the interactive confirmation prompt. Required for scripting.

### `gh label list`

- `-S`, `--search <string>` — Filter labels by name or description substring.
- `--sort <created|name>` — Sort order (default: `created`).
- `--order <asc|desc>` — Ascending or descending (default: `asc`).
- `-L`, `--limit <int>` — Max labels to return (default: 30).
- `--json <fields>` — JSON output; pipe to `--jq` for scripting.
- `-w`, `--web` — Open the labels page in your browser.

### All subcommands

- `-R`, `--repo <[HOST/]OWNER/REPO>` — Target a repo other than the one inferred from your current directory.

## Examples

```bash
# Create a label with a specific color and description
gh label create "priority:high" \
  --color D93F0B \
  --description "Needs immediate attention" \
  --repo borahanmirzaii/gh-mastery-sandbox

# Upsert: run safely even if the label already exists
gh label create "priority:high" \
  --color FF0000 \
  --description "Updated description" \
  --force \
  --repo borahanmirzaii/gh-mastery-sandbox

# Clone the entire label set from another repo into yours
gh label clone borahanmirzaii/gh-mastery \
  --repo borahanmirzaii/gh-mastery-sandbox

# Clone and overwrite any conflicting labels in the destination
gh label clone borahanmirzaii/gh-mastery \
  --force \
  --repo borahanmirzaii/gh-mastery-sandbox

# Rename a label and change its color
gh label edit "priority:high" \
  --name "priority:critical" \
  --color B60205 \
  --repo borahanmirzaii/gh-mastery-sandbox

# Search for labels matching a string, output as JSON
gh label list \
  --search "priority" \
  --json name,color,description \
  --jq '.[] | "\(.name) #\(.color)"' \
  --repo borahanmirzaii/gh-mastery-sandbox

# Delete a label non-interactively (useful in scripts)
gh label delete "priority:critical" \
  --yes \
  --repo borahanmirzaii/gh-mastery-sandbox
```

## Gotchas

- **`gh label clone <source-repo>` copies an entire label set.** It is the fastest way to bootstrap a new repo's taxonomy — one command pulls every label (name + color + description) from an existing repo. By default it skips labels that already exist in the destination; add `--force` to overwrite them. Example: `gh label clone borahanmirzaii/gh-mastery --repo borahanmirzaii/gh-mastery-sandbox`.

- **`gh label create --force` upserts.** Without `--force`, running `create` on a label that already exists errors out. With `--force` it silently updates the color and description instead. This makes `--force` the only safe flag for any label-bootstrap script you plan to re-run.

- **`gh label clone --force` is separate from `gh label create --force`.** The `clone` subcommand has its own `--force` flag, which means "overwrite labels in the destination." These are independent — you need to pass `--force` to `clone` explicitly if you want overwrite behavior there.

- **Color must be a 6-character hex string, no `#` prefix.** `--color E99695` works; `--color #E99695` does not.

- **`gh label list` defaults to 30 labels.** Repos with more than 30 labels require `-L` (e.g. `-L 100`) or you will silently miss some.

- **`gh label delete` prompts for confirmation interactively.** Scripts must pass `--yes` or they will hang waiting for input.

- **`gh label list` has an alias `gh label ls`.** Both work identically.

## Concepts

None. Labels are a flat GitHub primitive — no underlying concept doc needed.

## Sources

- Manual: https://cli.github.com/manual/gh_label
- Subcommands: https://cli.github.com/manual/gh_label_clone · https://cli.github.com/manual/gh_label_create · https://cli.github.com/manual/gh_label_edit · https://cli.github.com/manual/gh_label_delete · https://cli.github.com/manual/gh_label_list
- Local: `gh label --help` (gh 2.92.0)
