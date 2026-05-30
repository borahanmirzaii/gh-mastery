# Recall — `gh issue`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** You want to create an issue whose body spans multiple sections. Instead of writing the body inline with `--body`, what flag lets you pipe it from a heredoc — and what value do you pass to that flag?

<details><summary>Answer</summary>

Use `--body-file -` (a dash). The dash tells `gh` to read from stdin, so you can pipe or use a heredoc:

```bash
gh issue create --title "My issue" --body-file - <<'EOF'
## Problem
Details here.
EOF
```
</details>

---

**Q2.** You paste a link into an issue body as `[see spec](./docs/spec.md)`. A teammate clicks it and gets a 404. Why, and what should the link look like instead?

<details><summary>Answer</summary>

Issue body links resolve against `https://github.com/OWNER/REPO/issues/N` — not the repo root. A relative path like `./docs/spec.md` therefore points to a non-existent sub-path of the issues page.

Always use the full absolute URL:
```
https://github.com/OWNER/REPO/blob/BRANCH/docs/spec.md
```
</details>

---

**Q3.** What does `gh issue develop 4 --base dev --checkout` actually do? Name the two distinct things that happen — one server-side, one local.

<details><summary>Answer</summary>

1. **Server-side:** Creates a branch linked to issue #4 on GitHub. The branch appears in the issue's "Development" panel on github.com. This is a GitHub-level association, not just a local branch.
2. **Local:** Fetches that branch from the remote and checks it out in your working tree (`--checkout`).

Without `--checkout`, only the server-side branch is created — you'd need a separate `git fetch && git checkout` to work on it locally.
</details>

---

**Q4.** You run `gh issue create --project "gh-mastery"` and the issue is created but not added to the project. What's the most likely cause?

<details><summary>Answer</summary>

Your GitHub token is missing the `project` scope. Adding issues to Projects v2 requires explicit authorization:

```bash
gh auth refresh -s project
```

After refreshing, re-run the `issue create` or add it manually via `gh issue edit --add-project`.
</details>

---

**Q5.** How do you close an issue as a duplicate of issue #12, leave a comment, and set the reason — all in a single `gh issue close` command?

<details><summary>Answer</summary>

```bash
gh issue close 99 \
  --reason duplicate \
  --duplicate-of 12 \
  --comment "This is a duplicate of #12."
```

`--reason duplicate` sets the close reason; `--duplicate-of 12` creates the cross-link between the two issues. Both flags can be used together.
</details>

---

**Q6.** You need the branch linked to issue #7 to be named `feature/widget` instead of the default auto-generated name. What flag controls this in `gh issue develop`?

<details><summary>Answer</summary>

Use `--name`:

```bash
gh issue develop 7 --name "feature/widget" --checkout
```

Without `--name`, `gh` auto-generates a name from the issue number and title, e.g. `7-my-issue-title`.
</details>

---

**Q7.** `gh issue edit` can update multiple issues in one call. Write the command that adds the label `"help wanted"` to issues #23 and #34 simultaneously.

<details><summary>Answer</summary>

```bash
gh issue edit 23 34 --add-label "help wanted"
```

Pass multiple issue numbers as space-separated arguments; `gh issue edit` applies the same changes to all of them.
</details>

---

**Q8.** What is the difference between `gh issue close --reason duplicate` and `gh issue close --duplicate-of <N>`?

<details><summary>Answer</summary>

- `--reason duplicate` sets the close reason (the state reason label GitHub shows — "Closed as duplicate").
- `--duplicate-of <N>` creates a cross-reference link between the two issues, showing "Marked as duplicate of #N" in the timeline.

They are complementary. Use both together to get the full duplicate-close experience:

```bash
gh issue close 99 --reason duplicate --duplicate-of 12
```
</details>
