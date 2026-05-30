# `gh pr review`

> **One-liner:** Submit an approval, a "request changes" review, or a plain comment review on a pull request — without opening a browser.

## When you reach for it

When a teammate's PR lands in your review queue, `gh pr review --approve` clears it immediately if the diff is already in your editor. When you spot something wrong, `gh pr review --request-changes -b "explanation"` sends the feedback in one command. For a neutral note that doesn't block or approve, `gh pr review --comment` posts a comment-style review (distinct from a PR comment — it's visible in the "Reviews" section on GitHub).

## Key flags

- `-a, --approve` — Approve the PR. The review is submitted as "APPROVED" and (if branch protection requires N approvals) counts toward the threshold.
- `-r, --request-changes` — Submit a "CHANGES_REQUESTED" review. The PR cannot be merged (under normal branch protection) until the reviewer dismisses it or re-reviews as approved.
- `-c, --comment` — Submit a comment-style review ("COMMENT"). Does not approve or block; shows up in the Reviews section, not just the Comments thread.
- `-b, --body <string>` — The text of the review. Required when using `--request-changes` (you must explain what to fix). Optional for `--approve` and `--comment`.
- `-F, --body-file <file>` — Read the review body from a file; use `-` for stdin. Useful for long, templated review notes.

**Note:** `--approve`, `--request-changes`, and `--comment` are mutually exclusive — exactly one is required per invocation.

## Examples

```bash
# Approve the PR for the current branch
gh pr review --approve

# Approve with a brief note
gh pr review --approve --body "LGTM — nice cleanup."

# Request changes on a specific PR with an explanation
gh pr review 123 --request-changes \
  --body "The retry loop doesn't handle the 429 case — see line 42."

# Post a neutral comment review (visible in the Reviews section)
gh pr review 123 --comment --body "Interesting approach; let's discuss the perf implications."

# Review a PR by URL
gh pr review https://github.com/owner/repo/pull/99 --approve

# Read a detailed review body from a file
gh pr review 55 --request-changes --body-file ./review-notes.md

# Review a PR in a different repo without cd-ing
gh pr review 77 --approve --repo owner/other-repo

# Approve the PR for a specific branch name (not number)
gh pr review feature/retry-logic --approve
```

## Gotchas

- **You usually cannot approve your own PR.** GitHub's branch protection rules commonly require reviews from *other* contributors. In drills against the sandbox, self-review may work depending on the repo's settings — but in production repos it typically won't.

- **`--comment` is not the same as `gh pr comment`.** `gh pr review --comment` submits a formal *review* (appears under "Reviews" on GitHub, shows as "Commented" in the review timeline). `gh pr comment` adds a *conversation comment* to the PR thread. Both are visible but tracked differently by GitHub's review-required branch protection.

- **`--request-changes` requires a `--body`.** Submitting a changes-request review without an explanation is blocked by the CLI (and GitHub). Always pair it with `-b "what needs fixing"` or `--body-file`.

- **Without an argument, the PR for the current branch is targeted.** If you're checked out on `feature/my-thing` and a PR exists for that branch, `gh pr review --approve` reviews it automatically.

- **Re-requesting review after changes:** use `gh pr edit <N> --add-reviewer <login>` to re-request a review from someone who already reviewed. `gh pr review` submits your review, it doesn't manage *who* is requested.

## Concepts

- None.

## Sources

- Manual: https://cli.github.com/manual/gh_pr_review
- Local: `gh pr review --help` (gh 2.92.0)
