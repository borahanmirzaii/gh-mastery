# `gh pr merge`

> **One-liner:** Merge (or squash-merge, or rebase-merge) a pull request and optionally delete its branch.

## When you reach for it

This is the **final step of the solo-builder loop**: after the PR passes review and CI, `gh pr merge --squash --delete-branch` lands the change in one shot and removes the branch. No browser tab, no manual "Delete branch" button click.

Also reach for it when:
- You want to enable auto-merge so the PR merges automatically when checks pass (`--auto`).
- You're in a repo that uses a merge queue and you need to bypass it (`--admin`).
- You need to gate a merge script on a specific commit SHA (`--match-head-commit`).

## Key flags

- `-s, --squash` — Squash all commits into a single commit before merging. **Project convention.** Keeps history linear: one commit per PR.
- `-m, --merge` — Merge the commits with a merge commit (the git default). Creates a merge commit in the base branch.
- `-r, --rebase` — Rebase commits on top of the base branch. No merge commit; rewrites commit SHAs.
- `-d, --delete-branch` — Delete both the local and remote branch after a successful merge. Combine with `--squash` for the project convention one-liner.
- `--auto` — Enable auto-merge: the PR merges automatically once all required checks and approvals are satisfied. Useful when CI takes a while.
- `--disable-auto` — Turn off auto-merge on a PR where it was previously enabled.
- `--admin` — Use administrator privileges to merge a PR that doesn't meet branch-protection requirements, or to bypass a merge queue.
- `--match-head-commit <SHA>` — Refuse to merge unless the PR's head commit matches this SHA. A safety net against race conditions in scripts.
- `-t, --subject <text>` — Custom subject line for the merge commit (used with `--merge` or `--squash`).
- `-b, --body <text>` — Custom body for the merge commit.
- `-F, --body-file <file>` — Read merge commit body from a file (use `-` for stdin).
- `-A, --author-email <email>` — Override the author email for the merge commit.

## Examples

```bash
# Project convention: squash-merge the current branch's PR and delete the branch
gh pr merge --squash --delete-branch

# Merge a specific PR by number
gh pr merge 42 --squash --delete-branch

# Enable auto-merge so the PR lands once checks pass (no waiting around)
gh pr merge 42 --squash --auto

# Rebase-merge (no merge commit, rewrites SHAs)
gh pr merge 42 --rebase --delete-branch

# Bypass branch protection as an admin
gh pr merge 42 --squash --admin

# Script-safe merge: only merge if head is exactly this commit
gh pr merge 42 --squash --delete-branch \
  --match-head-commit abc1234def5678

# Merge with a custom commit subject
gh pr merge 42 --squash --subject "feat: add retry logic (#42)"
```

## Gotchas

- **`--squash --delete-branch` is the project convention.** Squash-merge gives you one clean commit per PR (linear history); `--delete-branch` removes the stale branch from both remote and local. Without it, merged branches pile up.

- **The three strategies are mutually exclusive.** You can only pass one of `--squash`, `--merge`, or `--rebase` per invocation. If none is provided and the repo has branch-protection rules that require a specific strategy, `gh` may prompt you interactively.

- **Merge queues change the behavior.** When the base branch requires a merge queue, no strategy flag is needed: if checks haven't passed yet, `--auto` activates; if checks have passed, the PR is added to the queue. Use `--admin` to bypass the queue entirely (audited action).

- **`--auto` is not the same as "merge now."** It means "merge automatically once all conditions are met." If conditions are never met (e.g., a required review is never added), the PR never merges. Use `--disable-auto` to cancel it.

- **Without an argument, the PR for the current branch is targeted.** You don't need to supply a PR number when you're checked out on the PR's branch; `gh pr merge` infers the PR. Useful in scripts that run on the feature branch.

- **`--delete-branch` deletes the local branch too** — not just the remote. If you have uncommitted local changes on the branch, make sure they're pushed before merging.

## Concepts

- None.

## Sources

- Manual: https://cli.github.com/manual/gh_pr_merge
- Local: `gh pr merge --help` (gh 2.92.0)
