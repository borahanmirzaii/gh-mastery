# `gh pr`

> **One-liner:** Create, review, and manage GitHub pull requests from the command line.

## When you reach for it

`gh pr` is the heartbeat of the solo-builder loop. After you push a feature branch, `gh pr create` opens the PR in seconds. When collaborators push back, `gh pr review` lets you approve or request changes without leaving the terminal. When everything is green, `gh pr merge --squash --delete-branch` lands the change and tidies up — one command for what the web UI spreads across three screens.

The typical solo-builder sequence:

1. `gh issue develop <N> --base dev --checkout` — branch linked to the issue.
2. … write code, commit …
3. `gh pr create --fill --draft` — open a draft PR; body autofilled from commits.
4. `gh pr ready` — flip out of draft when the work is done.
5. `gh pr merge --squash --delete-branch` — squash-merge and delete the branch.

## Subcommands

| Subcommand | Purpose | Node? |
|---|---|---|
| `gh pr create` | Open a new pull request | [→ `create/`](./create/) |
| `gh pr merge` | Merge a pull request | [→ `merge/`](./merge/) |
| `gh pr review` | Approve, comment, or request changes | [→ `review/`](./review/) |
| `gh pr checkout` | Check out a PR's branch locally | inline |
| `gh pr checks` | Show CI check status for a PR | inline |
| `gh pr close` | Close a PR without merging | inline |
| `gh pr comment` | Add a comment to a PR | inline |
| `gh pr diff` | Show the diff of a PR | inline |
| `gh pr edit` | Edit title, body, labels, reviewers, etc. | inline |
| `gh pr list` | List open (or filtered) PRs | inline |
| `gh pr lock` | Lock the PR conversation | inline |
| `gh pr ready` | Mark a draft PR as ready for review | inline |
| `gh pr reopen` | Reopen a closed PR | inline |
| `gh pr revert` | Create a revert PR for a merged PR | inline |
| `gh pr status` | Show PRs relevant to you (created, review-requested, mentioned) | inline |
| `gh pr unlock` | Unlock the PR conversation | inline |
| `gh pr update-branch` | Merge latest base-branch changes into the PR branch | inline |
| `gh pr view` | Display PR title, body, and metadata | inline |

## Key flags

These are the flags you'll actually reach for; promoted subcommands have their own detailed pages.

**Shared across most subcommands:**

- `-R, --repo OWNER/REPO` — Target a repo other than the one inferred from the current directory. Handy when running drills against the sandbox without `cd`-ing.

**`gh pr checkout`:**

- `-b, --branch <name>` — Use a custom local branch name instead of the PR's head branch name.
- `-f, --force` — Reset an existing local branch to the PR's latest state.

**`gh pr checks`:**

- `--watch` — Poll until all checks finish (great for CI gating in scripts).
- `--fail-fast` — Exit watch mode the moment any check fails.
- `--required` — Show only checks that are required to pass for merge.

**`gh pr close`:**

- `-d, --delete-branch` — Delete local and remote branch on close (mirrors merge behavior).
- `-c, --comment` — Leave a closing comment in one shot.

**`gh pr comment`:**

- `-b, --body` — Comment text inline (skips the interactive prompt).
- `--edit-last` — Edit your most recent comment rather than posting a new one.

**`gh pr diff`:**

- `--name-only` — Print only the changed file names, not the full patch.
- `-e, --exclude` — Glob pattern to omit files (e.g., `--exclude '*.lock'`).

**`gh pr edit`:**

- `--add-label / --remove-label` — Add or drop labels without opening the web UI.
- `--add-reviewer / --remove-reviewer` — Manages review requests; `@copilot` is a valid value.
- `-B, --base` — Change the target branch of an already-open PR.

**`gh pr list`:**

- `-s, --state` — `open` (default) | `closed` | `merged` | `all`.
- `-S, --search` — Full GitHub search syntax (e.g., `--search "review:required status:success"`).
- `--json <fields>` + `--jq` — Machine-readable output, great for scripting.

**`gh pr ready`:**

- `--undo` — Converts a ready PR back to draft (plan availability depends on your GitHub plan).

**`gh pr status`:**

- `-c, --conflict-status` — Show whether each PR has merge conflicts.

**`gh pr update-branch`:**

- `--rebase` — Rebase instead of merge when pulling in base-branch changes.

**`gh pr view`:**

- `-c, --comments` — Include all comments in the output.
- `-w, --web` — Open the PR in the browser.

## Examples

```bash
# Open a draft PR, autofilling title + body from commits
gh pr create --fill --draft

# Mark the current branch's PR as ready for review
gh pr ready

# Check CI status and wait for all checks to finish
gh pr checks --watch

# Check out PR #42 locally (creates a local branch named after the PR head)
gh pr checkout 42

# List only PRs that are waiting for your review
gh pr list --search "review-requested:@me"

# Show a machine-readable summary of your relevant PRs
gh pr status --json number,title,state --jq '.currentUserMentioned[]'

# Edit a PR to change its base branch
gh pr edit 99 --base main

# Update the PR branch with the latest changes from its base branch
gh pr update-branch --rebase

# Revert a merged PR by creating a new revert PR
gh pr revert 42 --title "Revert: accidentally shipped debug code"
```

## Gotchas

- **`--fill` autofills from commits, but sparse commits produce sparse PRs.** `gh pr create --fill` sets the title from the last commit subject and the body from the commit message body. If your commits say "wip" or "fix", that's what the PR will say. Write good commit messages — or skip `--fill` and use `--title` + `--body` explicitly. The `--fill-first` variant uses only the very first commit; `--fill-verbose` includes the full commit message bodies of all commits.

- **`Closes #N` (or `Fixes #N` / `Resolves #N`) in the PR body auto-closes the linked issue on merge.** This is GitHub's [closing keywords](https://docs.github.com/en/issues/tracking-your-work-with-issues/linking-a-pull-request-to-an-issue) feature, triggered at merge time — not at PR open time. `gh issue develop` inserts `Closes #N` automatically when you create the branch from an issue; don't delete it.

- **`--squash --delete-branch` is the project convention for landing a PR.** Squash-merge keeps history linear (one commit per PR) and `--delete-branch` removes both the local and remote branch in one shot. Without `--squash`, you get a merge commit; without `--delete-branch`, you accumulate stale branches.

- **`gh pr ready --undo` converts a PR back to draft** — it's the inverse of `gh pr ready`, not a separate subcommand. Availability depends on your GitHub plan.

- **`gh pr checkout` is also `gh pr co`** — a built-in alias. The `co` alias that appears at the top level (`gh co`) also points here.

- **`--title` and `--body` override `--fill`.** If you pass both `--fill` and `--title`, the explicit `--title` wins. This lets you autofill the body while supplying a hand-crafted title: `gh pr create --fill --title "feat: custom title"`.

- **Adding a PR to a GitHub Project requires the `project` scope.** Run `gh auth refresh -s project` first; otherwise `gh pr create --project` (or `gh pr edit --add-project`) will fail with a scope error.

## Concepts

- None. (PR mechanics are self-contained in the `gh pr` pages; see the GitHub docs on [pull requests](https://docs.github.com/en/pull-requests) for deeper background.)

## Sources

- Manual: https://cli.github.com/manual/gh_pr
- Local: `gh pr --help` (gh 2.92.0)
- Subcommand manuals: https://cli.github.com/manual/gh_pr_create · https://cli.github.com/manual/gh_pr_merge · https://cli.github.com/manual/gh_pr_review
