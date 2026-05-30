# Recall — `gh pr review`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** What are the three mutually exclusive review types for `gh pr review`, and what does each one do on GitHub?

<details><summary>Answer</summary>

- `--approve` — Submits an "APPROVED" review. Counts toward required-approvals branch protection thresholds.
- `--request-changes` — Submits a "CHANGES_REQUESTED" review. Blocks merge (under normal branch protection) until dismissed or overridden by a subsequent approval.
- `--comment` — Submits a neutral "COMMENT" review. No blocking effect; appears in the Reviews section (not just the comments thread). Distinct from `gh pr comment`.

Exactly one of the three must be supplied per invocation.
</details>

---

**Q2.** Which flag is mandatory when using `--request-changes`, and why?

<details><summary>Answer</summary>

`--body` (or `--body-file`). You must provide an explanation of what needs to change — the CLI (and GitHub) require a body for change-request reviews. Without it the command errors. Always pair it: `gh pr review 123 --request-changes --body "What to fix..."`.
</details>

---

**Q3.** What is the difference between `gh pr review --comment` and `gh pr comment`?

<details><summary>Answer</summary>

- `gh pr review --comment` — Posts a **review** of type COMMENT. It appears in the "Reviews" section of the PR timeline on GitHub and is tracked as a formal review event.
- `gh pr comment` — Posts a **conversation comment** in the PR's general comment thread. It's not a review; it doesn't appear under "Reviews".

Both are visible on the PR page, but they are tracked differently by GitHub's branch-protection review logic.
</details>

---

**Q4.** You are checked out on branch `feature/retry-logic` and a PR exists for it. How do you approve it without specifying its number?

<details><summary>Answer</summary>

`gh pr review --approve` — without any argument. `gh pr review` infers the PR from the current branch. Add `-b` for a message: `gh pr review --approve -b "LGTM"`.
</details>

---

**Q5.** Why can't you typically approve your own PR in a production repo, and how does this affect drill behavior?

<details><summary>Answer</summary>

Branch protection rules commonly require reviews from **other contributors** (not the PR author). GitHub itself enforces this at the API level when "Require approvals" is combined with "Dismiss stale pull request approvals." In the sandbox (with no branch protection configured), self-review works — which is why the drills show self-approval. In production, you'd need a teammate to run `gh pr review <N> --approve`.
</details>

---

**Q6.** After an author pushes fixes in response to a "request changes" review, how do you re-request a review from the original reviewer using `gh`?

<details><summary>Answer</summary>

`gh pr edit <N> --add-reviewer <login>` — this re-requests the review from the specified reviewer. Note that `gh pr review` is for *submitting* your review, not for *requesting* one from someone else. Re-requesting is done via `gh pr edit --add-reviewer`.
</details>
