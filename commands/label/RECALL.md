# Recall — `gh label`

Spaced-repetition (= study technique: review at increasing intervals) self-test.
Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** You want to copy every label from `acme/platform` into your new repo `acme/new-service`. What single command does this?

<details><summary>Answer</summary>

```bash
gh label clone acme/platform --repo acme/new-service
```

`gh label clone <source-repo>` copies the entire label set — names, colors, and descriptions — in one shot. It skips labels that already exist in the destination unless you add `--force`.
</details>

---

**Q2.** You have a label-bootstrap script that calls `gh label create` for each label. The first time you run it everything works, but the second time it errors on every label. What flag fixes this, and what does it actually do?

<details><summary>Answer</summary>

Add `--force` (short: `-f`) to every `gh label create` call.

`--force` turns `create` into an **upsert**: if the label is absent it creates it; if it already exists it updates the color and description instead of erroring. This makes the script safely re-runnable.
</details>

---

**Q3.** What is the difference between `--force` on `gh label clone` and `--force` on `gh label create`?

<details><summary>Answer</summary>

They are independent flags that happen to share the same name:

- **`gh label clone --force`** — overwrite labels in the *destination* repo that conflict with the source.
- **`gh label create --force`** — upsert the single label being created: update color/description if it already exists.

Passing `--force` to one does not affect the other.
</details>

---

**Q4.** You run `gh label create "urgent" --color #FF0000 --repo owner/repo` and get an error about the color. What is wrong and how do you fix it?

<details><summary>Answer</summary>

The `#` prefix is not accepted. Color must be a plain 6-character hex string with **no** `#`:

```bash
gh label create "urgent" --color FF0000 --repo owner/repo
```
</details>

---

**Q5.** You have a repo with 45 labels and run `gh label list --repo owner/repo`. How many labels do you see, and how do you see all of them?

<details><summary>Answer</summary>

You see only **30** — the default limit. To retrieve all 45:

```bash
gh label list --limit 100 --repo owner/repo
```

Use `-L` / `--limit` with a value larger than the total count. There is no `--paginate` for `label list`; just set a high enough limit.
</details>

---

**Q6.** You want to delete a label in a script without any interactive prompt. What flag do you need?

<details><summary>Answer</summary>

```bash
gh label delete "stale" --yes --repo owner/repo
```

Without `--yes`, the CLI prompts "Are you sure?" and hangs in a non-interactive context.
</details>

---

**Q7.** You want to rename the label `wontfix` to `declined` and update its description, all in one command. How?

<details><summary>Answer</summary>

```bash
gh label edit "wontfix" \
  --name "declined" \
  --description "Deliberately not going to be fixed" \
  --repo owner/repo
```

`gh label edit` accepts `--name`, `--color`, and `--description` in any combination — you can change one, two, or all three at once.
</details>

---

**Q8.** What does `gh label clone` do when a label from the source already exists in the destination (without `--force`)?

<details><summary>Answer</summary>

It **skips** the conflicting label — the existing destination label is left unchanged. No error is raised; the command just moves on to the next label. Add `--force` if you want the source's color and description to overwrite the destination's.
</details>

---

**Q9.** What is the alias for `gh label list`?

<details><summary>Answer</summary>

`gh label ls` — both commands are identical.
</details>
