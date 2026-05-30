# Drills — `gh project`

> **Sandbox:** All mutation drills create user-owned projects, items, and fields prefixed `zz-project-*`.
> Read-only drills target the real **gh-mastery Project #14** (`--owner borahanmirzaii`) or the public `cli/cli` project.
> Clean up with `gh project delete` and `gh project item-delete` at the end of each drill.
> **Namespace:** `zz-project-*` only — never touch a real project board for mutations.

---

## Drill 1 — List your projects

**Goal:** See all your open GitHub Projects v2 boards, then include closed ones.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Open projects only
gh project list --owner @me

# Include closed projects
gh project list --owner @me --closed

# As JSON, showing just number + title
gh project list --owner @me --format json \
  --jq '.projects[] | "\(.number) \(.title)"'
```
</details>

**Verify:** At least one project is listed. If you have none, move to Drill 2 first and come back.

---

## Drill 2 — Create and view a sandbox project

**Goal:** Create a disposable project named `zz-project-drill`, view it in the terminal, then open it in the browser.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Create it (note the project number printed)
gh project create --owner @me --title "zz-project-drill" \
  --format json --jq '.number'

# Store the number (replace 99 with actual number)
PROJ=99

# View in terminal
gh project view "$PROJ" --owner @me

# Open in browser
gh project view "$PROJ" --owner @me --web
```
</details>

**Verify:** `gh project list --owner @me` includes `zz-project-drill`.

**Cleanup:** `gh project delete "$PROJ" --owner @me` — confirm deletion.

---

## Drill 3 — Add items to a project and query them

**Goal:** Using your `zz-project-drill` board (or a fresh one), add a real issue from `gh-mastery` as an item, then list items with a status filter.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
PROJ=$(gh project create --owner @me --title "zz-project-items-drill" \
  --format json --jq '.number')

# Add issue #1 from gh-mastery to the project
gh project item-add "$PROJ" --owner @me \
  --url https://github.com/borahanmirzaii/gh-mastery/issues/1

# Create a draft item directly on the board
gh project item-create "$PROJ" --owner @me \
  --title "zz-project-draft-task" --body "This is a throwaway draft item"

# List all items
gh project item-list "$PROJ" --owner @me

# List as JSON, extracting titles only
gh project item-list "$PROJ" --owner @me \
  --format json --jq '.items[].title'
```
</details>

**Verify:** Two items appear: one linked issue, one draft. The `--jq` output shows both titles.

**Cleanup:**
```bash
# Delete each item by its node ID (get IDs from item-list --format json)
ITEM_IDS=$(gh project item-list "$PROJ" --owner @me \
  --format json --jq '.items[].id')
for ID in $ITEM_IDS; do
  gh project item-delete "$PROJ" --owner @me --id "$ID"
done
gh project delete "$PROJ" --owner @me
```

---

## Drill 4 — List fields on a real project (read-only)

**Goal:** Inspect the fields on the gh-mastery project (#14) to understand the structure — this is the read-only version of what you'd do before writing a GraphQL Status mutation.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Human-readable table
gh project field-list 14 --owner borahanmirzaii

# JSON — name, type, id, and options (for single-select fields)
gh project field-list 14 --owner borahanmirzaii \
  --format json --jq '.fields[] | {name,type,id,options}'
```
</details>

**Verify:** You see at minimum the built-in fields: `Title`, `Assignees`, `Status`, `Labels`, `Linked Pull Requests`, `Reviewers`, `Repository`, `Milestone`. The `Status` field shows its option IDs under `.options`.

---

## Drill 5 — Create and delete a custom field

**Goal:** Add a custom `TEXT` field named `zz-project-sprint-goal` to a sandbox project, then delete it.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
PROJ=$(gh project create --owner @me --title "zz-project-field-drill" \
  --format json --jq '.number')

# Create the field
gh project field-create "$PROJ" --owner @me \
  --name "zz-project-sprint-goal" --data-type TEXT \
  --format json --jq '.id'

# Capture the field node ID for deletion
FIELD_ID=$(gh project field-list "$PROJ" --owner @me \
  --format json \
  --jq '.fields[] | select(.name=="zz-project-sprint-goal") | .id')

echo "Field ID: $FIELD_ID"

# Delete the field
gh project field-delete --id "$FIELD_ID"

# Confirm deletion
gh project field-list "$PROJ" --owner @me
```
</details>

**Verify:** After deletion, `field-list` no longer shows `zz-project-sprint-goal`.

**Cleanup:** `gh project delete "$PROJ" --owner @me`

---

## Boss drill — Triage workflow: create board, add issues, set a field, query

Chain `project create`, `item-add`, `field-create`, `item-edit`, `item-list --query` into a single triage workflow that mirrors real project management.

**Scenario:** You're opening a sprint board, pulling in two gh-mastery issues, tagging them with a custom priority field, then querying what's outstanding.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# 1. Create the sprint board
PROJ=$(gh project create --owner @me --title "zz-project-sprint-board" \
  --format json --jq '.number')
echo "Project: $PROJ"

# 2. Add two real issues from gh-mastery
gh project item-add "$PROJ" --owner @me \
  --url https://github.com/borahanmirzaii/gh-mastery/issues/1
gh project item-add "$PROJ" --owner @me \
  --url https://github.com/borahanmirzaii/gh-mastery/issues/2

# 3. Create a custom single-select Priority field
FIELD_ID=$(gh project field-create "$PROJ" --owner @me \
  --name "zz-project-priority" --data-type SINGLE_SELECT \
  --single-select-options "High,Medium,Low" \
  --format json --jq '.id')
echo "Field: $FIELD_ID"

# 4. Get option IDs for the field
gh project field-list "$PROJ" --owner @me \
  --format json \
  --jq '.fields[] | select(.name=="zz-project-priority") | .options'

# 5. Get the project's node ID (needed for item-edit)
PROJECT_NODE_ID=$(gh project view "$PROJ" --owner @me \
  --format json --jq '.id')

# 6. Get item node IDs
gh project item-list "$PROJ" --owner @me \
  --format json --jq '.items[] | {title: .title, id: .id}'

# 7. Set priority on first item (replace <item-id> and <option-id> with real values)
# gh project item-edit \
#   --id <item-node-id> \
#   --field-id "$FIELD_ID" \
#   --project-id "$PROJECT_NODE_ID" \
#   --single-select-option-id <high-option-id>

# 8. List all items (no status filter on a fresh board — just show everything)
gh project item-list "$PROJ" --owner @me --format json \
  --jq '.items[].title'
```
</details>

**Verify:** The project has 2 linked items and 1 custom field. `item-list` shows both issue titles.

**Cleanup:**
```bash
gh project delete "$PROJ" --owner @me
```
Confirm with `gh project list --owner @me --limit 100 | grep zz-project-` — no `zz-project-*` rows should remain.
