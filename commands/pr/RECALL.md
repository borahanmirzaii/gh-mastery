# Recall — `gh pr`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** What does `gh pr create --fill` use to populate the PR title and body — and what's the catch if your commits are low-quality?

<details><summary>Answer</summary>

`--fill` pulls the title from the **last commit's subject line** and the body from the accumulated commit message bodies. If your commit messages are sparse ("fix", "wip", "update"), the PR will be equally sparse. Write meaningful commit messages, or override with `--title` and `--body` (explicit flags take precedence over `--fill`).
</details>

---

**Q2.** How does `Closes #N` in a PR body work, and what are the accepted keywords?

<details><summary>Answer</summary>

GitHub's closing-keywords feature auto-closes the referenced issue **when the PR is merged** (not when it's opened). Accepted keywords: `Closes`, `Fixes`, `Resolves` (and their variants: `close`, `fix`, `resolve`, case-insensitive). `gh issue develop <N>` inserts `Closes #N` in the PR body automatically when you branch from an issue.
</details>

---

**Q3.** What is the project convention for merging a PR, and why?

<details><summary>Answer</summary>

`gh pr merge --squash --delete-branch`. `--squash` keeps the history linear (one commit per feature), and `--delete-branch` removes both the local and remote branch in a single step. Without these flags you accumulate merge commits and stale branches.
</details>

---

**Q4.** What's the difference between `--fill`, `--fill-first`, and `--fill-verbose` on `gh pr create`?

<details><summary>Answer</summary>

- `--fill` — title from the last commit subject; body from all commit message bodies combined.
- `--fill-first` — title and body from the **first** commit only (useful when you want the branch's opening commit to define the PR).
- `--fill-verbose` — like `--fill` but includes the full message body of each commit, not just the subjects.

If you also pass `--title` or `--body`, those explicit values **override** the autofilled content.
</details>

---

**Q5.** How do you convert a ready PR back to draft, and what caveat applies?

<details><summary>Answer</summary>

`gh pr ready --undo` converts a ready PR back to draft. The caveat: this feature requires a GitHub plan that supports draft PRs (GitHub Free for public repos, and GitHub Pro/Team/Enterprise for private repos). On unsupported plans the command errors.
</details>

---

**Q6.** You're in the root of a different repo and want to list open PRs in `borahanmirzaii/gh-mastery-sandbox` without `cd`-ing. What flag do you use?

<details><summary>Answer</summary>

`-R, --repo`: `gh pr list -R borahanmirzaii/gh-mastery-sandbox`. This flag is inherited by every `gh pr` subcommand.
</details>

---

**Q7.** `gh pr checks --watch` exits with what code when at least one check fails? How would you use this in a CI gate script?

<details><summary>Answer</summary>

It exits with **code 1** when checks fail, and code **8** when checks are still pending (if `--watch` isn't used). In a script: `gh pr checks --watch --fail-fast && echo "all checks passed" || echo "checks failed"`. The `--fail-fast` flag exits watch mode the moment any single check fails rather than waiting for all to finish.
</details>

---

**Q8.** What scope does `gh auth refresh` need before you can add a PR to a GitHub Project, and what command adds that scope?

<details><summary>Answer</summary>

The `project` scope. Run `gh auth refresh -s project`. Without it, `gh pr create --project "My Board"` (or `gh pr edit --add-project`) will fail with an authorization error.
</details>
