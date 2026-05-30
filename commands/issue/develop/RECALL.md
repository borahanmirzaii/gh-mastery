# Recall — `gh issue develop`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** What are the two distinct things that `gh issue develop 4 --base dev --checkout` does — one server-side, one local?

<details><summary>Answer</summary>

1. **Server-side:** Creates a branch on GitHub linked to issue #4. The link is visible in the issue's "Development" panel on github.com. This is a GitHub-level association stored server-side.
2. **Local:** Fetches that branch from the remote and checks it out in your working tree (equivalent to `git fetch origin` + `git checkout 4-master-gh-issue`).

Without `--checkout`, only step 1 happens.
</details>

---

**Q2.** What is the auto-generated branch name pattern when you run `gh issue develop 7` for an issue titled "Add widget feature"?

<details><summary>Answer</summary>

`7-add-widget-feature` — the pattern is `<issue-number>-<slugified-title>`. The title is lowercased, spaces become hyphens, and non-alphanumeric characters are stripped.

Override with `--name "your-preferred-name"` if you need something different.
</details>

---

**Q3.** You ran `gh issue develop 12 --base dev`. Now how do you start working on it locally?

<details><summary>Answer</summary>

```bash
git fetch origin
git checkout 12-your-issue-title
```

Or, run from the start with `--checkout` to do it in one command:

```bash
gh issue develop 12 --base dev --checkout
```
</details>

---

**Q4.** How do you see which branches are already linked to issue #42, without creating a new branch?

<details><summary>Answer</summary>

```bash
gh issue develop 42 --list
```

The `--list` flag is read-only — it shows linked branches without creating anything.
</details>

---

**Q5.** You want to create a branch for issue #10 in a fork repo (`alice/my-fork`) while the issue lives in the upstream repo (`upstream/project`). Which flag controls where the branch is created vs. which repo owns the issue?

<details><summary>Answer</summary>

- `--repo upstream/project` — specifies the repo that **owns the issue**.
- `--branch-repo alice/my-fork` — specifies the repo where the **branch will be created**.

```bash
gh issue develop 10 \
  --repo upstream/project \
  --branch-repo alice/my-fork \
  --checkout
```
</details>

---

**Q6.** How does `--base dev` affect `gh pr create` later?

<details><summary>Answer</summary>

When you create a branch with `gh issue develop --base dev`, GitHub CLI records `dev` as the intended merge base for that branch. When you later run `gh pr create` from that branch, it pre-fills `--base dev` automatically — you don't have to remember to pass it manually. This prevents accidentally opening a PR against `main` when the project convention is to PR into `dev`.
</details>
