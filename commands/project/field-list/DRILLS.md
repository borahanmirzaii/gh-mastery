# Drills — `gh project field-list`

> **Sandbox:** Read-only drills target the real **gh-mastery Project #14** (`--owner borahanmirzaii`).
> Mutation drills create disposable projects prefixed `zz-project-*` and delete them at the end.
> **Namespace:** only `zz-project-*` objects — never mutate a real board's fields.

---

## Drill 1 — Inspect fields on the real board

**Goal:** List all fields in the gh-mastery project #14 and understand what you see in table vs JSON output.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Table view
gh project field-list 14 --owner borahanmirzaii

# JSON — name, type, id for each field
gh project field-list 14 --owner borahanmirzaii \
  --format json --jq '.fields[] | {name,type,id}'

# Single-select fields only (to find those with options)
gh project field-list 14 --owner borahanmirzaii \
  --format json \
  --jq '.fields[] | select(.type=="SINGLE_SELECT") | {name,id,options}'
```
</details>

**Verify:** The table shows field names and types but no option IDs. The JSON output for `Status` includes `.options[]` with `id` and `name` for each option (Todo, In Progress, Done).

---

## Drill 2 — Extract the Status field's option IDs

**Goal:** Retrieve the node ID of the `Status` field and the IDs of each option. These are the prerequisite values for any `item-edit --single-select-option-id` call or a `updateProjectV2Field` GraphQL mutation.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Field ID for Status
FIELD_ID=$(gh project field-list 14 --owner borahanmirzaii \
  --format json --jq '.fields[] | select(.name=="Status") | .id')
echo "Status field ID: $FIELD_ID"

# All option names and IDs for Status
gh project field-list 14 --owner borahanmirzaii \
  --format json \
  --jq '.fields[] | select(.name=="Status") | .options[] | {name,id}'
```
</details>

**Verify:** `$FIELD_ID` starts with `PVTF_`. Each Status option (Todo, In Progress, Done) has a distinct short string ID.

---

## Drill 3 — Create a custom field and confirm it appears in field-list

**Goal:** Create a single-select field with three options on a sandbox project, then verify it appears with the correct options in `field-list --format json`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Create sandbox project
PROJ=$(gh project create --owner @me --title "zz-project-fieldlist-drill" \
  --format json --jq '.number')

# Create a single-select field
gh project field-create "$PROJ" --owner @me \
  --name "zz-project-priority" \
  --data-type SINGLE_SELECT \
  --single-select-options "High,Medium,Low"

# Verify it appears in field-list
gh project field-list "$PROJ" --owner @me \
  --format json \
  --jq '.fields[] | select(.name=="zz-project-priority") | {name,type,options}'
```
</details>

**Verify:** The JSON output shows `"type": "SINGLE_SELECT"` and three options: High, Medium, Low — each with its own `id`.

**Cleanup:**
```bash
FIELD_ID=$(gh project field-list "$PROJ" --owner @me \
  --format json \
  --jq '.fields[] | select(.name=="zz-project-priority") | .id')
gh project field-delete --id "$FIELD_ID"
gh project delete "$PROJ" --owner @me
```

---

## Drill 4 — Full prerequisite chain: field-list → item-edit

**Goal:** Use `field-list` to collect the IDs needed to set a field value on a project item with `item-edit`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# 1. Create sandbox project + add item
PROJ=$(gh project create --owner @me --title "zz-project-chain-drill" \
  --format json --jq '.number')

gh project item-create "$PROJ" --owner @me \
  --title "zz-project-chain-task" --body "chain drill task"

# 2. Create a single-select priority field
gh project field-create "$PROJ" --owner @me \
  --name "Priority" --data-type SINGLE_SELECT \
  --single-select-options "High,Medium,Low"

# 3. Collect IDs
PROJECT_NODE=$(gh project view "$PROJ" --owner @me --format json --jq '.id')
FIELD_ID=$(gh project field-list "$PROJ" --owner @me \
  --format json --jq '.fields[] | select(.name=="Priority") | .id')
OPTION_ID=$(gh project field-list "$PROJ" --owner @me \
  --format json \
  --jq '.fields[] | select(.name=="Priority") | .options[] | select(.name=="High") | .id')
ITEM_ID=$(gh project item-list "$PROJ" --owner @me \
  --format json --jq '.items[0].id')

echo "Project: $PROJECT_NODE | Field: $FIELD_ID | Option: $OPTION_ID | Item: $ITEM_ID"

# 4. Set Priority = High on the item
gh project item-edit \
  --id "$ITEM_ID" \
  --field-id "$FIELD_ID" \
  --project-id "$PROJECT_NODE" \
  --single-select-option-id "$OPTION_ID"

# 5. Confirm
gh project item-list "$PROJ" --owner @me --format json \
  --jq '.items[] | {title, fieldValues}'
```
</details>

**Verify:** The item's field values include the Priority field set to "High".

**Cleanup:**
```bash
gh project delete "$PROJ" --owner @me
```

---

## Boss drill — Field audit + Status option introspection

You want to document every field on the gh-mastery board and produce a clean Markdown table of the Status options with their IDs, so you have everything ready to write a `updateProjectV2Field` GraphQL mutation.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# 1. Full field audit — all fields, types, IDs
echo "## gh-mastery project #14 fields"
gh project field-list 14 --owner borahanmirzaii \
  --format json \
  --jq '["Field","Type","ID"], (.fields[] | [.name, .type, .id]) | @tsv'

# 2. Status options table
echo ""
echo "## Status options"
gh project field-list 14 --owner borahanmirzaii \
  --format json \
  --jq '.fields[] | select(.name=="Status") | .options[] | "| \(.name) | \(.id) |"'

# 3. Project node ID (needed for the mutation's $pid)
echo ""
echo "## Project node ID"
gh project view 14 --owner borahanmirzaii --format json --jq '.id'

# 4. Status field node ID (needed for the mutation's $fid)
echo ""
echo "## Status field node ID"
gh project field-list 14 --owner borahanmirzaii \
  --format json --jq '.fields[] | select(.name=="Status") | .id'
```
</details>

**Verify:** You have a TSV-formatted field audit and a Markdown-formatted Status options table, plus both node IDs ready for a GraphQL mutation.

**Cleanup:** Read-only drill — nothing to delete.
