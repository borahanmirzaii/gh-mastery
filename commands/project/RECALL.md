# Recall — `gh project`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** Does `gh project` work with GitHub "classic" projects (the old column-board style)?

<details><summary>Answer</summary>

No. `gh project` is **Projects v2 only**. Classic projects (deprecated by GitHub) are not accessible through these commands. If a board shows in the GitHub UI but not in `gh project list`, it is a classic project.
</details>

---

**Q2.** What flag do you almost always need to pass to `gh project` subcommands, and what do you use when the project belongs to you personally versus an organisation?

<details><summary>Answer</summary>

`--owner`. Use `--owner @me` for your own user-account projects, and `--owner <org-name>` for organisation-owned projects. Without it, most subcommands fail with an auth or "not found" error.
</details>

---

**Q3.** You want to rename the built-in Status option "Todo" to "Backlog". Can you do this with `gh project field-edit`? If not, what do you use instead?

<details><summary>Answer</summary>

No. `gh project field-edit` **only works on custom fields**. Built-in Status option names must be changed via GraphQL. The mutation is `updateProjectV2Field`, passing the `fieldId` (from `field-list --format json`) and the new `singleSelectOptions` array with the existing option ID and the new name. Example skeleton:

```bash
gh api graphql -f query='
  mutation($pid:ID!,$fid:ID!,$oid:String!,$name:String!) {
    updateProjectV2Field(input:{
      projectId:$pid, fieldId:$fid,
      singleSelectOptions:[{id:$oid,name:$name,color:GRAY,description:""}]
    }) { projectV2Field { ... on ProjectV2SingleSelectField { name options {id name} } } }
  }' \
  -F pid="$PROJECT_NODE_ID" -F fid="$FIELD_ID" \
  -F oid="<existing-option-id>" -F name="Backlog"
```
</details>

---

**Q4.** What extra OAuth scope is required to access or list **organisation** projects, and how do you add it?

<details><summary>Answer</summary>

Org projects require both the `project` scope and `read:org`. Add them with:

```bash
gh auth refresh -s project,read:org
```

Check current scopes with `gh auth status`.
</details>

---

**Q5.** `gh project item-edit` and `gh project item-delete` require `--id`. What kind of value goes there, and how do you get it?

<details><summary>Answer</summary>

An opaque GraphQL **node ID** (looks like `PVTI_…`), not the human-readable item number. Retrieve it with:

```bash
gh project item-list <num> --owner @me --format json \
  --jq '.items[] | {title: .title, id: .id}'
```

Similarly, `--field-id` and `--project-id` expect `PVTF_…` and `PVT_…` node IDs from `field-list` and `project view --format json` respectively.
</details>

---

**Q6.** How many fields can `gh project item-edit` update in a single invocation?

<details><summary>Answer</summary>

**Exactly one.** To update multiple fields on the same item, you must issue multiple `item-edit` calls, one per field.
</details>

---

**Q7.** You run `gh project item-list 14 --owner borahanmirzaii --query "status:Done"` on an on-prem GHES 3.18 instance and get back every item, not just the Done ones. Why?

<details><summary>Answer</summary>

The `--query` filter (Projects filter syntax) is a **server-side feature** that requires github.com or GHES 3.20+. On older GHES, the flag is silently ignored and all items are returned. Either upgrade the GHES instance or apply client-side filtering with `--format json --jq 'select(.status=="Done")'`.
</details>

---

**Q8.** What is the alias for `gh project list`?

<details><summary>Answer</summary>

`gh project ls` — it is listed under `ALIASES` in `gh project list --help`.
</details>
