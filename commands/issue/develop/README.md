# `gh issue develop`

> **One-liner:** Create a branch that is server-side linked to a GitHub Issue — making the branch visible in the issue's "Development" panel — and optionally check it out locally in one step.

## When you reach for it

`gh issue develop` is the handoff (= the transition point, hand-off) between planning and execution in the solo-builder loop. After you file an issue (`gh issue create`), you run `gh issue develop <N> --base dev --checkout` to:

1. Create the branch on GitHub with a formal link to the issue.
2. Check it out locally so you can start committing immediately.

This whole project depends on this exact behavior: the Lead runs `gh issue develop` for each command-group issue, which creates the linked branch that every worker agent then targets. The branch shows up in the issue's "Development" panel on github.com — giving visibility into which branches are in flight against which issues, and ensuring that a PR opened from that branch auto-suggests `Closes #N`.

When **not** to use it: if you just want a local branch without the GitHub-side link, use `git checkout -b` directly.

## Key flags

- `-b, --base string` — Base the new branch on this remote branch instead of the repo's default branch. Always pass `--base dev` in this project (never base on `main`).
- `-c, --checkout` — Fetch and check out the new branch locally immediately after creating it server-side.
- `-n, --name string` — Override the auto-generated branch name. Without this, the name is `<number>-<slugified-title>` (e.g., `4-master-gh-issue`).
- `-l, --list` — List all branches already linked to this issue (read-only — does not create anything).
- `--branch-repo string` — Create the branch in a different repository from the one that owns the issue (fork workflows or monorepo setups).
- `-R, --repo` — Specify which repo owns the issue (not necessarily where the branch will live).

## Examples

```bash
# 1. Create a linked branch from issue #4 based on dev, and check it out
gh issue develop 4 \
  --repo borahanmirzaii/gh-mastery \
  --base dev \
  --checkout

# 2. Create the branch but do NOT check out yet
gh issue develop 4 \
  --repo borahanmirzaii/gh-mastery \
  --base dev
# Later: git fetch origin && git checkout 4-master-gh-issue

# 3. Override the auto-generated branch name
gh issue develop 4 \
  --repo borahanmirzaii/gh-mastery \
  --base dev \
  --name "feature/issue-command-group" \
  --checkout

# 4. List branches already linked to issue #4
gh issue develop 4 \
  --repo borahanmirzaii/gh-mastery \
  --list

# 5. Create a branch in a fork repo for an issue in the upstream repo
gh issue develop 123 \
  --repo cli/cli \
  --branch-repo monalisa/cli \
  --checkout
```

## Gotchas

- **Creates a server-side linked branch** — not just a local branch. The branch is visible in the issue's "Development" panel on github.com immediately, and the link persists even if you delete the local checkout. This is distinct from `git checkout -b`, which only creates a local branch with no GitHub-level link.

- **Auto-generated branch names follow `<number>-<slugified-title>`** — for issue #4 titled "Master `gh issue`", the branch is `4-master-gh-issue`. Titles with special characters or very long titles get aggressively slugified. Use `--name` to override if you need a specific convention.

- **`--base` configures the PR base too** — from the help text: "The new branch will be configured as the base branch for pull requests created using `gh pr create`." This means `gh pr create` in that branch will pre-fill `--base dev` (or whatever you passed). Very convenient in this project; potentially surprising if you forget to pass `--base` and get a PR targeting `main`.

- **Without `--checkout`, no local changes happen** — the branch is created on the remote, but you still need `git fetch && git checkout` to work on it locally.

- **`--list` is read-only** — it only shows linked branches; it cannot unlink them. Deleting a linked branch removes the link automatically.

- **`--branch-repo` vs `--repo`** — `--repo` targets the repo that *owns the issue*; `--branch-repo` targets the repo where the *branch will be created*. For fork workflows they are different repos.

- **The PR body gets `Closes #N` automatically when the branch came from `gh issue develop`** — GitHub infers the relationship from the Development panel link, and `gh pr create --fill` picks up the commit messages including the closure keyword.

## Concepts

- None beyond standard GitHub branch-issue linking. The server-side link is a GitHub feature (not a `gh` invention) — it's what appears in the "Development" section of an issue's sidebar.

## Sources

- Manual: https://cli.github.com/manual/gh_issue_develop
- Local: `gh issue develop --help` (gh 2.92.0)
