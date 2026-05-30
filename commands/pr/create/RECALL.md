# Recall — `gh pr create`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** `gh pr create --fill` sets the PR title from which part of the git history?

<details><summary>Answer</summary>

The **last commit's subject line** (the first line of the most recent commit message). If you have multiple commits, the subjects of earlier commits do not appear in the title — they go into the body. Use `--fill-first` to pull from the first commit instead.
</details>

---

**Q2.** You run `gh pr create --fill --title "feat: my override"`. Which title appears on GitHub?

<details><summary>Answer</summary>

`"feat: my override"` — the explicitly supplied `--title` value. When `--title` (or `--body`) is provided alongside `--fill`, **the explicit flag always wins** for that field. The other field (`--body`) is still autofilled from commits.
</details>

---

**Q3.** What happens to the linked issue when a PR with `Closes #N` in the body is merged?

<details><summary>Answer</summary>

GitHub **automatically closes issue #N** at merge time. The closing keyword is evaluated on merge, not on PR creation. Accepted keywords: `Closes`, `Fixes`, `Resolves` (case-insensitive, with or without `#`). `gh issue develop <N>` inserts this keyword automatically when you create the branch from the issue.
</details>

---

**Q4.** What extra authorization does `gh pr create --project "My Board"` require, and how do you grant it?

<details><summary>Answer</summary>

The `project` OAuth scope. Grant it with: `gh auth refresh -s project`. Without this scope, the `--project` flag fails with an authorization error even if all other token scopes are valid.
</details>

---

**Q5.** What is the difference between `--fill` and `--fill-verbose`?

<details><summary>Answer</summary>

`--fill` uses commit **subject lines** (the first line of each commit message) to build the PR body. `--fill-verbose` includes the full commit message — **subject + body** — for every commit. Use `--fill-verbose` when your commit bodies contain context that belongs in the PR description.
</details>

---

**Q6.** How do you create a PR without being prompted interactively, using your `$EDITOR` to write the title and body?

<details><summary>Answer</summary>

`gh pr create --editor` — this skips all prompts and opens your `$EDITOR`. The first line of what you write becomes the PR title; everything after the blank line becomes the body.
</details>

---

**Q7.** The sandbox branch hasn't been pushed yet. What does `gh pr create` do when you run it?

<details><summary>Answer</summary>

It **prompts you** to choose a remote to push to (or offers to fork the base repo). To avoid this interactive prompt in scripts, always `git push -u origin <branch>` before running `gh pr create`. The `--head` flag can also be used to explicitly specify the head branch and skip the push prompt.
</details>
