# `gh repo`

> **One-liner:** Create, clone, fork, view, and manage GitHub repositories from the command line.

## When you reach for it

`gh repo` is the entry point for everything repository-lifecycle related in the solo-builder loop: you start a project with `gh repo create`, hand it to collaborators with `gh repo fork`, inspect any repo with `gh repo view`, keep forks in sync with `gh repo sync`, and tidy up archived or renamed repos without ever touching the web UI. It is also the command you run first when onboarding to an existing codebase — `gh repo clone owner/repo` wires up the upstream remote automatically if the target is a fork.

## Subcommands

| Subcommand | Purpose | Node? |
|---|---|---|
| `gh repo create` | Create a new GitHub repository (interactive or non-interactive) | [→ `create/`](./create/) |
| `gh repo edit` | Edit repository settings (features, merge strategy, visibility, topics) | [→ `edit/`](./edit/) |
| `gh repo list` | List repositories owned by a user or org | inline |
| `gh repo view` | Display repo description and README; open in browser | inline |
| `gh repo clone` | Clone a repository locally, auto-wiring upstream for forks | inline |
| `gh repo fork` | Create a fork of a repository | inline |
| `gh repo sync` | Sync a fork or branch from its source (fast-forward or hard reset) | inline |
| `gh repo rename` | Rename a repository | inline |
| `gh repo delete` | Delete a repository (requires `delete_repo` scope) | inline |
| `gh repo archive` | Archive a repository (makes it read-only) | inline |
| `gh repo unarchive` | Un-archive a previously archived repository | inline |
| `gh repo set-default` | Configure the default repository for the current directory | inline |
| `gh repo deploy-key` | Manage deploy keys in a repository | inline |
| `gh repo autolink` | Manage autolink references | inline |
| `gh repo gitignore` | List and view available `.gitignore` templates | inline |
| `gh repo license` | Explore repository licenses | inline |

## Key flags

Flags are subcommand-specific (see the promoted nodes for `create` and `edit`). The flags below apply broadly across the group:

- `-R, --repo [HOST/]OWNER/REPO` — Target a specific repository instead of inferring from the current directory. Accepted by `view`, `edit`, `rename`, `delete`, `archive`, `unarchive`, `sync`, `set-default`.
- `--json <fields>` — Output structured JSON for piping to `jq`; available on `view`, `list`. Run with `--json ''` to see all available fields.
- `-q, --jq <expression>` — Filter JSON output inline without a separate `jq` invocation.
- `--web` / `-w` — Open the result in the browser instead of printing to the terminal. Available on `view`.
- `--limit / -L <int>` — Cap the number of results returned by `list` (default 30).

## Examples

```bash
# Inspect any public repo without cloning it
gh repo view cli/cli

# Open the current repo's GitHub page in the browser
gh repo view --web

# List your own public repos, sorted by name, outputting JSON for scripting
gh repo list borahanmirzaii --visibility public --json name,url --jq '.[] | .name'

# Clone a fork — gh auto-adds an upstream remote pointing at the parent
gh repo clone borahanmirzaii/my-fork

# Sync a remote fork's default branch from its parent (fast-forward update)
gh repo sync borahanmirzaii/my-fork

# Rename the current repo interactively (prompts for confirmation)
gh repo rename new-name

# Archive a repo you no longer actively develop
gh repo archive borahanmirzaii/old-project --yes

# Transfer ownership via the REST API (there is NO gh repo transfer command — see Gotchas)
gh api -X POST repos/borahanmirzaii/my-repo/transfer -f new_owner=target-org
```

## Gotchas

- **`gh repo transfer` does NOT exist.** The CLI has no `transfer` subcommand. To transfer a repository to another user or organisation, use the REST API directly:
  ```bash
  gh api -X POST repos/{owner}/{repo}/transfer \
    -f new_owner=<target-user-or-org>
  ```
  You need admin rights on the source repo and (for org targets) owner rights on the destination org.

- **`--source=. --push` vs `--clone` in `gh repo create`.** These two flags serve opposite directions:
  - `--source=. --push` — you already have a local repo and want to publish it to a new GitHub remote. The local directory is pushed up.
  - `--clone` — you create an empty remote and immediately clone it down to your local machine. Use this when starting fresh.
  Using `--source` and `--clone` together will error; they are mutually exclusive (see [→ `create/`](./create/) for the full breakdown).

- **`gh repo edit` feature flags require explicit toggle-off syntax.** To disable a feature you must write `--enable-issues=false`, not `--disable-issues`. The `--disable-*` flags only exist in `gh repo create`, not in `edit` (see [→ `edit/`](./edit/)).

- **`gh repo delete` requires the `delete_repo` OAuth scope**, which is not granted by default. If you get a 403, run `gh auth refresh -s delete_repo` first.

- **Inferring the repository.** Most subcommands work without `-R` when you're inside a local clone — `gh` reads the `origin` remote or the configured default (set via `gh repo set-default`). Outside a clone, or when you have multiple remotes, specify `-R OWNER/REPO` explicitly.

- **`gh repo fork` sets up remotes automatically.** After forking, your fork becomes `origin` and the original becomes `upstream`. If you already had an `origin`, the old remote is renamed to `upstream`. Control this with `--remote-name`.

- **`gh repo sync --force` is a hard reset.** It discards any divergent local commits on the target branch. Use only when you want to throw away divergent history on the destination.

## Concepts

- None. (Repository concepts are fundamental GitHub constructs; no dedicated `concepts/` doc is needed for this group.)

## Sources

- Manual: https://cli.github.com/manual/gh_repo
- Local: `gh repo --help` (gh 2.92.0)
