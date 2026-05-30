# Recall — `gh pr merge`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** What is the project convention for merging a PR, and what does each flag in the one-liner do?

<details><summary>Answer</summary>

`gh pr merge --squash --delete-branch`.

- `--squash` — Collapses all commits in the PR into a single commit before merging, keeping history linear (one commit per feature/fix).
- `--delete-branch` — Deletes both the remote and local branch immediately after the merge succeeds.

Together they land the change cleanly and leave no stale branches behind.
</details>

---

**Q2.** What is the difference between `--auto` and merging normally?

<details><summary>Answer</summary>

`--auto` enables **auto-merge**: GitHub will merge the PR automatically once all required checks and approvals are satisfied — you don't have to come back and run `gh pr merge` again. Without `--auto`, the merge happens immediately (or fails immediately if requirements aren't met). Use `--disable-auto` to cancel a pending auto-merge.
</details>

---

**Q3.** You're on `main`, not on the PR branch. How do you merge PR #77 in a different repo?

<details><summary>Answer</summary>

`gh pr merge 77 --squash --delete-branch -R owner/repo`

Supply the PR number as a positional argument and use `-R` (or `--repo`) to specify the repository. `gh pr merge` doesn't require you to be on the PR branch — the branch is inferred from the PR number.
</details>

---

**Q4.** What happens when `gh pr merge` targets a branch that requires a **merge queue**?

<details><summary>Answer</summary>

- If required checks haven't yet passed: **auto-merge activates** automatically.
- If required checks have passed: the PR is **added to the merge queue**.
- You don't need to specify a merge strategy flag (`--squash`/`--merge`/`--rebase`) in merge-queue mode.
- To bypass the queue entirely (audited), pass `--admin`.
</details>

---

**Q5.** What does `--match-head-commit <SHA>` protect against?

<details><summary>Answer</summary>

It protects against **race conditions** in scripts: the merge is refused if the PR's head commit doesn't match the supplied SHA. This ensures you're merging exactly the code you reviewed, not a commit that was pushed after your check.
</details>

---

**Q6.** The three merge strategies (`--squash`, `--merge`, `--rebase`) each produce a different git history. Summarize what each does to the base branch.

<details><summary>Answer</summary>

- `--squash` — All PR commits are flattened into **one new commit** on the base branch. Original commits are discarded from the base history. Best for linear history.
- `--merge` — A **merge commit** is added to the base branch, preserving all PR commits as a branch in the history graph. Traditional git merge.
- `--rebase` — PR commits are **replayed on top of the base branch** with new SHAs, no merge commit. Looks like linear history but preserves individual commits (unlike squash).
</details>
