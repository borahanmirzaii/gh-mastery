# Recall — `gh project item-list`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** What is the default maximum number of items `gh project item-list` returns, and how do you get more?

<details><summary>Answer</summary>

The default `--limit` is **30**. Pass `-L <n>` (e.g. `-L 100`) to raise it. There is no automatic pagination in the CLI; if you need all items on a large board, set the limit high or query the GraphQL API directly.
</details>

---

**Q2.** You run `gh project item-list 14 --owner borahanmirzaii --query "status:Done"` and get back every item on the board, not just the Done ones. Name two possible reasons.

<details><summary>Answer</summary>

1. **You're on GHES older than 3.20** — the `--query` filter requires github.com or GHES 3.20+. On older versions the flag is silently ignored.
2. **No items actually have the `Done` status set** — the filter works but returns an empty match, while the unfiltered result returns everything. (A less likely third reason: the status option name is case-sensitive or spelled differently in that project.)
</details>

---

**Q3.** What format of ID does `gh project item-list` return in its JSON output, and what is it used for?

<details><summary>Answer</summary>

A GraphQL **node ID** (opaque string like `PVTI_…`). This is the value you pass to `--id` in `gh project item-edit`, `gh project item-delete`, and `gh project item-archive`. It is not the visible row number from the Projects UI.
</details>

---

**Q4.** Write the one-liner that counts items currently in the gh-mastery project #14.

<details><summary>Answer</summary>

```bash
gh project item-list 14 --owner borahanmirzaii --limit 100 \
  --format json --jq '.items | length'
```
</details>

---

**Q5.** How do you filter `item-list` output to show only items that have the label `kind:command`?

<details><summary>Answer</summary>

Use the `--query` flag with the `label:` qualifier:

```bash
gh project item-list 14 --owner borahanmirzaii \
  --query "label:kind:command"
```

The filter is a string argument, not a separate `--label` flag. Labels containing colons (like `kind:command`) work fine in the filter string.
</details>

---

**Q6.** Draft items created with `gh project item-create` — do they appear in `item-list` output?

<details><summary>Answer</summary>

Yes. Draft issues appear in `item-list` alongside linked issues and PRs. In the JSON output, their `.content` field is `null` (no linked issue), but their `.title` is set directly. They are indistinguishable in the table view; check the JSON to tell them apart.
</details>
