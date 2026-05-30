# Recall — `gh issue create`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** What flag reads the issue body from stdin, and what value do you pass to it?

<details><summary>Answer</summary>

`--body-file -` — the dash (`-`) is the value, instructing `gh` to read from standard input. Example:

```bash
gh issue create --title "My issue" --body-file - <<'EOF'
Body text here.
EOF
```
</details>

---

**Q2.** You include `[Plan](./docs/plans/2026-05-26-gh-mastery.md)` in an issue body. The link 404s on github.com. What went wrong, and what's the fix?

<details><summary>Answer</summary>

Issue body links resolve against `https://github.com/OWNER/REPO/issues/N`, not the repo root. A relative path (`./docs/...`) therefore resolves to a non-existent path under `/issues/`.

Fix: use the absolute URL:
```
https://github.com/OWNER/REPO/blob/BRANCH/docs/plans/2026-05-26-gh-mastery.md
```
</details>

---

**Q3.** You run `gh issue create --project "My Board"` and the issue is created but never appears in the project. What's the most likely cause and fix?

<details><summary>Answer</summary>

The token lacks the `project` scope. Projects v2 operations require explicit authorization:

```bash
gh auth refresh -s project
```

After refreshing, retry the create or use `gh issue edit --add-project "My Board"`.
</details>

---

**Q4.** `gh issue create --milestone` takes a name or a number?

<details><summary>Answer</summary>

It takes the **name** (a string), not the milestone number. Example:

```bash
gh issue create --milestone "M1 — Core (daily drivers)" ...
```

Passing `--milestone 1` will error or not find the milestone.
</details>

---

**Q5.** What is the alias for `gh issue create`?

<details><summary>Answer</summary>

`gh issue new` — it's a built-in alias that behaves identically.
</details>

---

**Q6.** You want to self-assign an issue without hardcoding your GitHub username in a script. What value do you pass to `--assignee`?

<details><summary>Answer</summary>

`@me` — GitHub CLI resolves this to the currently authenticated user:

```bash
gh issue create --assignee @me --title "..."
```
</details>
