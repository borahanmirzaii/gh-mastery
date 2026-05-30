# Recall — `gh repo create`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** You have an existing local git repo with commits. What is the correct `gh repo create` invocation to publish it to GitHub and push all commits in one shot?

<details><summary>Answer</summary>

```bash
gh repo create my-project --public --source=. --push
```

`--source=.` points at the current directory; `--push` pushes the local commits up immediately. Do **not** add `--clone` — that is for the opposite direction (creating an empty remote and cloning it down).
</details>

---

**Q2.** What happens if you combine `--source=.` and `--clone` in the same `gh repo create` command?

<details><summary>Answer</summary>

The command fails with an error. The two flags are mutually exclusive:
- `--source` = you already have something locally and are pushing it up.
- `--clone` = you want to create an empty remote and pull it down.

They represent opposite directions and cannot be used together.
</details>

---

**Q3.** How do you create a repo and immediately have it checked out locally, starting from nothing (no local files yet)?

<details><summary>Answer</summary>

```bash
gh repo create my-project --public --clone
```

`--clone` creates the remote and then runs a `git clone`, leaving you with a local directory ready to work in.
</details>

---

**Q4.** At creation time, you want to seed a `.gitignore` for Python and an MIT license. What flags do you use?

<details><summary>Answer</summary>

```bash
gh repo create my-project --public --gitignore Python --license mit --add-readme
```

Use `gh repo gitignore list` to see available template names and `gh repo license list` for license keywords.
</details>

---

**Q5.** You created a repo with `--disable-issues`. Later you want to re-enable issues. What command do you run?

<details><summary>Answer</summary>

```bash
gh repo edit my-project --enable-issues
```

`--disable-issues` is a `gh repo create`-only flag. In `gh repo edit`, features are toggled via `--enable-<feature>` (to turn on) or `--enable-<feature>=false` (to turn off). There is no `--disable-issues` flag in `edit`.
</details>

---

**Q6.** What is the alias for `gh repo create`?

<details><summary>Answer</summary>

`gh repo new` — it is a built-in alias that behaves identically to `gh repo create`.
</details>

---

**Q7.** When you use `--push` with a bare git repository as `--source`, what gets pushed?

<details><summary>Answer</summary>

All refs are mirrored — every branch and tag in the bare repo is pushed. For a non-bare repo, only the current branch's commits are pushed.
</details>
