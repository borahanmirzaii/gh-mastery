# `gh pr create`

> **One-liner:** Open a new pull request on GitHub from the current (or a specified) branch.

## When you reach for it

This is **step 3 of the solo-builder loop**: after `gh issue develop <N>` branches you off and you commit your changes, `gh pr create --fill --draft` publishes the PR in one command — no browser required. The `--draft` flag signals "not ready for review yet" so you can push progress without triggering reviewers. When the work is done, `gh pr ready` flips it open.

Also reach for it when:
- You want to file a PR against a fork's branch (`--head owner:branch`).
- You need a dry run before actually creating (`--dry-run`).
- You prefer to write the title/body in your editor (`--editor`).

## Key flags

- `-f, --fill` — Autofill title and body from the branch's commit messages. Title comes from the last commit subject; body from the aggregated commit bodies. **Explicit `--title`/`--body` override `--fill` when both are supplied.**
- `--fill-first` — Like `--fill` but uses only the first commit's info instead of the last.
- `--fill-verbose` — Like `--fill` but includes the full message body (not just the subject) of every commit in the description.
- `-d, --draft` — Open the PR as a draft; reviewers are not notified.
- `-B, --base <branch>` — The branch you want to merge *into* (target branch). Defaults to the repo's default branch, or to the value of `git config branch.<current>.gh-merge-base` if set.
- `-H, --head <branch>` — The branch containing your commits. Defaults to the current branch. Supports `<user>:<branch>` syntax for cross-fork PRs.
- `-t, --title <string>` — PR title. Overrides `--fill`.
- `-b, --body <string>` — PR body. Overrides `--fill`.
- `-F, --body-file <file>` — Read body from a file; use `-` to read from stdin.
- `-T, --template <file>` — Pre-populate the body from a PR template file.
- `-r, --reviewer <handle>` — Request review; supports `@me`, user logins, and `org/team` syntax. Repeatable.
- `-a, --assignee <login>` — Assign to a user; `@me` self-assigns.
- `-l, --label <name>` — Add a label. Repeatable.
- `-p, --project <title>` — Add to a GitHub Project (requires `project` scope — see Gotchas).
- `-m, --milestone <name>` — Add to a milestone by name.
- `--no-maintainer-edit` — Prevent maintainers of the base repo from pushing to the PR branch. Default is to allow it.
- `-e, --editor` — Skip prompts; open your `$EDITOR` to write title + body. First line = title, rest = body.
- `-w, --web` — Open the PR creation form in the browser instead.
- `--dry-run` — Print what would be created (may still push the branch to the remote).
- `--recover <string>` — Resume a previously interrupted `gh pr create` session.

## Examples

```bash
# The canonical solo-builder one-liner: draft PR, autofilled from commits
gh pr create --fill --draft

# Supply title explicitly but autofill body from commits
gh pr create --fill --title "feat: add retry logic"

# Target a specific base branch (not the repo default)
gh pr create --fill --base dev

# Open a PR against a fork (cross-repository PR)
gh pr create --head myuser:feature-branch --base upstream-owner:main

# Read the body from a file, useful for long descriptions written offline
gh pr create --title "feat: big feature" --body-file ./pr-description.md

# Request a review from a team and self-assign
gh pr create --fill --reviewer myorg/platform --assignee @me

# Dry run: see what would be created without actually creating
gh pr create --fill --dry-run

# Write the title + body in your $EDITOR
gh pr create --editor
```

## Gotchas

- **`--fill` autofills from commits, but sparse commits produce sparse PRs.** The title comes from the last commit's subject line. A commit message like "wip" or "fix" produces a useless PR title. Write descriptive commits, or always pair `--fill` with an explicit `--title`.

- **`Closes #N` / `Fixes #N` / `Resolves #N` in the body auto-closes the linked issue on merge.** `gh issue develop <N> --checkout` inserts this closing keyword automatically. Don't delete it from the body — it's the glue between the issue and the PR. The issue only closes at merge time, not when the PR is opened.

- **`--title` and `--body` each independently override `--fill`.** You can autofill the body while supplying a custom title: `gh pr create --fill --title "feat: custom"`. This does *not* force the entire fill to be skipped — only the flag that's explicitly provided overrides its autofilled counterpart.

- **Adding to a GitHub Project requires the `project` scope.** Run `gh auth refresh -s project` once per account before using `--project`. Otherwise you'll get an authorization error even if your token has all other scopes.

- **`--head` does not support organization syntax for cross-fork PRs.** `user:branch` works; `org:branch` does not (see [cli/cli#10093](https://github.com/cli/cli/issues/10093)). Use a personal fork for cross-org PR drills.

- **`gh pr create` is also `gh pr new`** — a built-in alias.

- **If the branch isn't pushed yet, `gh pr create` prompts where to push it** — including an option to fork the base repo. Use `git push -u origin <branch>` first to avoid the interactive prompt in scripts.

## Concepts

- None.

## Sources

- Manual: https://cli.github.com/manual/gh_pr_create
- Local: `gh pr create --help` (gh 2.92.0)
