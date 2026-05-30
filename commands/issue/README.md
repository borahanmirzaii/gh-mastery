# `gh issue`

> **One-liner:** Create, browse, and manage GitHub Issues from the terminal — the entry point to every issue workflow.

## When you reach for it

Issues are the single source of truth for work in the solo-builder loop. You reach for `gh issue` at **Step 1 of the Loop** — drafting the brief as an issue body — and again whenever you need to inspect, triage, or close work. Because this very repo (`gh-mastery`) tracks each command group as an issue and uses `gh issue develop` to spin up the worker branch, the whole curriculum is a live worked example of `gh issue`.

Concrete moments:
- Start a feature: `gh issue create` → captures the brief in the repo.
- Hand off to a worker: `gh issue develop` → creates the server-side linked branch the worker checks out.
- Triage a bug report: `gh issue edit` to relabel, reassign, or milestone.
- Close the loop: `gh issue close` (or auto-close via `Closes #N` in a PR body).

## Subcommands

| Subcommand | Purpose | Node? |
|---|---|---|
| `gh issue create` | Open a new issue (title, body, labels, assignees, milestone, project) | [→ `create/`](./create/) |
| `gh issue develop` | Create a server-side branch linked to an issue; optionally checkout locally | [→ `develop/`](./develop/) |
| `gh issue list` | List open (or filtered) issues in a repo | inline |
| `gh issue status` | Show issues assigned to you, mentioning you, or open by you | inline |
| `gh issue view` | Display an issue's title, body, and metadata | inline |
| `gh issue edit` | Modify title, body, labels, assignees, milestone, or project | inline |
| `gh issue close` | Close an issue with an optional reason and closing comment | inline |
| `gh issue reopen` | Reopen a previously closed issue | inline |
| `gh issue comment` | Add a comment; supports `--edit-last`, `--delete-last`, `--body-file -` | inline |
| `gh issue delete` | Permanently delete an issue (requires `--yes` to skip prompt) | inline |
| `gh issue lock` | Lock the conversation (off_topic, resolved, spam, too_heated) | inline |
| `gh issue unlock` | Unlock a locked conversation | inline |
| `gh issue pin` | Pin the issue to the top of the issue list | inline |
| `gh issue unpin` | Remove a pinned issue | inline |
| `gh issue transfer` | Move the issue to another repository | inline |

## Key flags

These are the flags you'll actually reach for day-to-day across the subcommands:

**On `gh issue create`:**
- `-t, --title` — Set the issue title non-interactively; pair with `--body` to skip all prompts.
- `-b, --body` — Supply the body inline. Useful in scripts; for long bodies prefer `--body-file`.
- `-F, --body-file file` — Read body from a file. Pass `-` to read from **stdin** (heredoc or pipe).
- `-l, --label` — Attach a label by name; repeat the flag for multiple labels.
- `-a, --assignee` — Assign by login; `@me` self-assigns without hard-coding your username.
- `-m, --milestone` — Attach by milestone name (not number).
- `-p, --project` — Add to a Project v2 by title (requires `project` scope — see Gotchas).
- `-T, --template` — Pre-fill body from an issue template by name.
- `-w, --web` — Open the browser creation form instead of the CLI flow.

**On `gh issue list`:**
- `-s, --state` — `open` (default), `closed`, or `all`.
- `-l, --label` — Filter by label; repeat for AND logic.
- `-a, --assignee` — Filter by assignee login (`@me` works here too).
- `-S, --search` — Full GitHub issue search query syntax (`is:open label:bug author:@me`).
- `--json fields` + `-q, --jq` — Machine-readable output; pipe into `jq` or scripts.

**On `gh issue close`:**
- `-r, --reason` — `completed`, `not planned`, or `duplicate`.
- `--duplicate-of` — Mark as duplicate of another issue by number or URL.
- `-c, --comment` — Leave a closing comment in one shot.

**On `gh issue develop`:**
- `-b, --base` — Branch the new branch from a specific remote branch (default: repo default branch).
- `-c, --checkout` — Check out the newly created branch locally immediately.
- `-n, --name` — Override the auto-generated branch name.
- `-l, --list` — List all branches already linked to this issue.
- `--branch-repo` — Create the branch in a different repo from the issue (fork workflows).

## Examples

```bash
# 1. Create an issue non-interactively with a label and self-assignment
gh issue create \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --title "zz-issue-demo: example issue" \
  --body "Demonstrates non-interactive creation." \
  --label bug \
  --assignee @me

# 2. Create an issue whose body comes from stdin (heredoc)
gh issue create \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --title "zz-issue-stdin: body from stdin" \
  --body-file - <<'EOF'
## Problem
Something is broken.

## Steps to reproduce
1. Do this.
2. See that.

**Spec:** https://github.com/borahanmirzaii/gh-mastery/blob/main/docs/superpowers/specs/2026-05-26-gh-mastery-design.md
EOF

# 3. Linked branch + local checkout in one command
gh issue develop 4 \
  --repo borahanmirzaii/gh-mastery \
  --base dev \
  --checkout

# 4. List open issues assigned to me as JSON, pull just the numbers
gh issue list \
  --repo borahanmirzaii/gh-mastery \
  --assignee @me \
  --json number,title \
  --jq '.[] | "#\(.number) \(.title)"'

# 5. Close an issue as duplicate with a comment
gh issue close 99 \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --reason duplicate \
  --duplicate-of 1 \
  --comment "Covered by #1."

# 6. Edit labels in bulk (useful after triage)
gh issue edit 42 \
  --repo borahanmirzaii/gh-mastery \
  --add-label "kind:command" \
  --remove-label "meta"
```

## Gotchas

- **Issue body links must be absolute URLs.** A relative path like `[spec](./docs/spec.md)` in an issue body resolves against `https://github.com/OWNER/REPO/issues/N`, not the repo root — the link 404s. Always use the full GitHub URL: `https://github.com/OWNER/REPO/blob/BRANCH/path/to/file.md`.

- **`gh issue develop` creates a server-side linked branch** — you can see it in the issue's "Development" panel on github.com. The branch exists on the remote immediately; `--checkout` fetches and switches to it locally. This is the mechanism this whole project's worker fan-out depends on: the Lead runs `gh issue develop <N> --base dev --checkout` to spin up an isolated worktree per command group.

- **`--body-file -` reads body from stdin.** Combine with a heredoc to write rich multi-section bodies in a script without a temp file. Critically, you can also pipe from a generator: `gh issue view 1 --json body --jq .body | gh issue create --body-file - ...`.

- **`--project` requires the `project` scope.** If the flag silently does nothing or errors, run `gh auth refresh -s project` first. The regular read/write issue scopes do not cover Projects v2.

- **`gh issue develop` auto-names the branch** using the pattern `<number>-<slugified-title>` (e.g., `4-master-gh-issue`). Override with `--name` if you need a different naming convention or if the title is very long.

- **`gh issue status` is repo-scoped** — run it inside a git repo (or pass `-R`) or it errors. It shows three buckets: issues created by you, assigned to you, and mentioning you.

- **`gh issue edit` accepts multiple issue numbers** — `gh issue edit 23 34 --add-label "help wanted"` updates them both in one call.

- **`gh issue delete --yes` is permanent** — there is no trash/undo. In the sandbox drills we always clean up this way; in production repos, prefer `close` instead.

- **`gh issue close --reason duplicate` vs `--duplicate-of`** — the `--reason duplicate` flag sets the close reason (label equivalent); `--duplicate-of <N>` additionally cross-links the two issues. You can use both together.

## Concepts

- None beyond standard GitHub Issues semantics. For Projects v2 (which `create` and `edit` touch via `--project`), see [`../../concepts/projects-v2-data-model.md`](../../concepts/projects-v2-data-model.md) once that node is authored.

## Sources

- Manual: https://cli.github.com/manual/gh_issue
- Local: `gh issue --help` (gh 2.92.0)
