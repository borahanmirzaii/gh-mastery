# Recall — `gh project field-list`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** What does the default table output of `gh project field-list` omit that makes it unsuitable for scripting?

<details><summary>Answer</summary>

The table view omits **option IDs** for single-select fields. If you need the option node IDs (required by `item-edit --single-select-option-id` and GraphQL mutations), you must use `--format json`. The table shows field names and types only.
</details>

---

**Q2.** You want to rename the "Todo" Status option to "Backlog". Can you do this with `gh project field-edit`?

<details><summary>Answer</summary>

No — `gh project field-edit` does not exist as a subcommand (as of gh 2.92.0), and even if it did, built-in Status options are read-only via the CLI. You must use the `updateProjectV2Field` GraphQL mutation via `gh api graphql`. You need three IDs: the **project node ID** (`PVT_…` from `gh project view --format json`), the **field node ID** (`PVTF_…` from `field-list --format json`), and the **option ID** (short string from `.options[].id` in `field-list` JSON).
</details>

---

**Q3.** When calling `updateProjectV2Field` to rename a single Status option, do you need to supply the other options in the mutation too?

<details><summary>Answer</summary>

Yes. You must include **all** existing options in the `singleSelectOptions` array. Options omitted from the array are **deleted**, along with any item values set to those options. Always read the full current options list first and pass all of them, modifying only the one you want to change.
</details>

---

**Q4.** How do you get the node ID of the `Status` field in project #14 in a single `gh` command?

<details><summary>Answer</summary>

```bash
gh project field-list 14 --owner borahanmirzaii \
  --format json \
  --jq '.fields[] | select(.name=="Status") | .id'
```
</details>

---

**Q5.** What do field node IDs look like vs single-select option IDs? Why does this matter?

<details><summary>Answer</summary>

- **Field node IDs** start with `PVTF_` for most fields, or `PVTSSF_` for single-select fields (e.g. `PVTSSF_lAHODj…`) — long opaque strings.
- **Option IDs** are short hex strings (e.g. `f75ad846`).

They are passed to different parameters in mutations: `fieldId` gets the `PVTF_…` node ID, while `singleSelectOptions[].id` and `item-edit --single-select-option-id` get the short option IDs. Mixing them up causes the mutation to fail.
</details>

---

**Q6.** Write the `field-list` command that shows only the single-select fields in project #14, including their options.

<details><summary>Answer</summary>

```bash
gh project field-list 14 --owner borahanmirzaii \
  --format json \
  --jq '.fields[] | select(.type=="SINGLE_SELECT") | {name,id,options}'
```
</details>
