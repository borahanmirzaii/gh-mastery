# Recall — `gh repo`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** How do you transfer a GitHub repository to a different owner when there is no `gh repo transfer` command?

<details><summary>Answer</summary>

Use the REST API directly:
```bash
gh api -X POST repos/{owner}/{repo}/transfer -f new_owner=<target-user-or-org>
```
`gh repo transfer` does not exist — running it gives an "unknown command" error.
</details>

---

**Q2.** In `gh repo create`, what is the difference between `--source=. --push` and `--clone`? When would you use each?

<details><summary>Answer</summary>

- `--source=. --push` — you already have a **local** repo and want to publish it to a **new GitHub remote**. Local commits are pushed up. Use this to turn an existing project into a GitHub repo.
- `--clone` — creates an **empty** remote and immediately clones it **down** to your machine. Use this when starting a brand-new project from scratch.

The two flags are mutually exclusive; using both together will error.
</details>

---

**Q3.** You want to disable issues on a repo using `gh repo edit`. What is the correct flag syntax?

<details><summary>Answer</summary>

```bash
gh repo edit --enable-issues=false
```

`--disable-issues` does NOT exist in `gh repo edit` (it only exists in `gh repo create`). To toggle a feature off in `edit`, you must use the `--enable-<feature>=false` form.
</details>

---

**Q4.** What OAuth scope does `gh repo delete` require, and how do you add it if you get a 403?

<details><summary>Answer</summary>

`gh repo delete` requires the `delete_repo` scope. Add it with:
```bash
gh auth refresh -s delete_repo
```
This scope is not granted by default when you first authenticate.
</details>

---

**Q5.** After running `gh repo fork owner/repo`, what remote names does gh set up locally?

<details><summary>Answer</summary>

- Your fork becomes `origin`.
- The original (parent) repository is added as `upstream`.
- If an `origin` remote already existed in the clone, it is renamed to `upstream`.

Control the fork's remote name with `--remote-name`.
</details>

---

**Q6.** What does `gh repo sync --force` do, and when should you be cautious about using it?

<details><summary>Answer</summary>

`--force` performs a **hard reset** on the destination branch to match the source. Any divergent commits on the destination branch are permanently discarded. Use it only when you explicitly want to throw away local divergent history (e.g., a fork that has drifted and you want to start clean from upstream).
</details>

---

**Q7.** How do you list all public repos for user `borahanmirzaii` and output only their names, one per line?

<details><summary>Answer</summary>

```bash
gh repo list borahanmirzaii --visibility public --json name --jq '.[].name'
```
</details>

---

**Q8.** Name three useful feature-flag options you can toggle with `gh repo edit`.

<details><summary>Answer</summary>

Any three from:
- `--enable-issues` / `--enable-issues=false`
- `--enable-wiki` / `--enable-wiki=false`
- `--enable-projects` / `--enable-projects=false`
- `--enable-discussions` / `--enable-discussions=false`
- `--delete-branch-on-merge` — auto-delete head branch after merge
- `--enable-squash-merge`, `--enable-rebase-merge`, `--enable-merge-commit` — control allowed merge strategies
- `--enable-auto-merge`
- `--allow-forking` (org repos)
</details>
