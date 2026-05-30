# Drills — `gh project item-list`

> **Sandbox:** Read-only drills target the real **gh-mastery Project #14** (`--owner borahanmirzaii`).
> Mutation drills create disposable projects prefixed `zz-project-*` and delete them at the end.
> **Namespace:** only `zz-project-*` objects — never mutate a real board.

---

## Drill 1 — Basic list on the real board

**Goal:** List all items in the gh-mastery project #14, then get them as JSON and extract just the titles.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Human-readable table (default output)
gh project item-list 14 --owner borahanmirzaii

# JSON — extract titles only
gh project item-list 14 --owner borahanmirzaii \
  --format json --jq '.items[].title'

# Count items
gh project item-list 14 --owner borahanmirzaii \
  --format json --jq '.items | length'
```
</details>

**Verify:** The table shows items from the gh-mastery board. The `--jq` output shows one title per line. The count matches what the table shows.

---

## Drill 2 — Raise the limit and check truncation

**Goal:** Confirm you're not missing items by comparing the default limit against a higher one.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Default — may be truncated at 30
COUNT_DEFAULT=$(gh project item-list 14 --owner borahanmirzaii \
  --format json --jq '.items | length')

# Raised limit
COUNT_HIGH=$(gh project item-list 14 --owner borahanmirzaii \
  --limit 100 --format json --jq '.items | length')

echo "Default: $COUNT_DEFAULT  |  High limit: $COUNT_HIGH"
```
</details>

**Verify:** If `COUNT_HIGH` > `COUNT_DEFAULT`, you were being truncated. Always use `-L 100` (or higher) in scripts.

---

## Drill 3 — Filter with `--query`

**Goal:** Use the Projects filter syntax to list only items that are not yet Done, and then only items assigned to you.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Items that are NOT Done (negation with - prefix)
gh project item-list 14 --owner borahanmirzaii \
  --query "-status:Done"

# Items assigned to yourself (uses @me shorthand)
gh project item-list 14 --owner borahanmirzaii \
  --query "assignee:@me"

# Open issues assigned to yourself
gh project item-list 14 --owner borahanmirzaii \
  --query "assignee:@me is:issue is:open"
```
</details>

**Verify:** The filtered list is a subset of the unfiltered list. Test that the filter works by comparing counts:
```bash
gh project item-list 14 --owner borahanmirzaii --format json --jq '.items|length'
gh project item-list 14 --owner borahanmirzaii --query "-status:Done" --format json --jq '.items|length'
```

---

## Drill 4 — Extract node IDs for scripting

**Goal:** Get the GraphQL node ID for each item in a sandbox project (node IDs are what `item-edit` and `item-delete` require).

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Create a sandbox project and add items to it
PROJ=$(gh project create --owner @me --title "zz-project-itemlist-drill" \
  --format json --jq '.number')

gh project item-add "$PROJ" --owner @me \
  --url https://github.com/borahanmirzaii/gh-mastery/issues/1
gh project item-create "$PROJ" --owner @me \
  --title "zz-project-draft-recall" --body "draft item for node ID drill"

# Now extract title + node ID for each item
gh project item-list "$PROJ" --owner @me \
  --format json --jq '.items[] | {title: .title, id: .id}'
```
</details>

**Verify:** Each item has a distinct `id` starting with `PVTI_`.

**Cleanup:**
```bash
gh project delete "$PROJ" --owner @me
```

---

## Boss drill — Query → triage → archive

Chain `item-list` with `item-archive` to simulate a sprint cleanup: find items with a particular status and archive them.

**Scenario:** You have a sandbox project with a few items. At the end of a sprint, you want to archive everything that is "Done."

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# 1. Create board and add items
PROJ=$(gh project create --owner @me --title "zz-project-triage-drill" \
  --format json --jq '.number')

gh project item-add "$PROJ" --owner @me \
  --url https://github.com/borahanmirzaii/gh-mastery/issues/1
gh project item-create "$PROJ" --owner @me \
  --title "zz-project-done-item" --body "This one is done"

# 2. List all items with their IDs
gh project item-list "$PROJ" --owner @me \
  --format json --jq '.items[] | {title: .title, id: .id}'

# 3. Archive a specific item by node ID (replace <id> with real value)
# gh project item-archive "$PROJ" --owner @me --id <PVTI_...>

# 4. Verify — archived items no longer appear in the default list
gh project item-list "$PROJ" --owner @me

# 5. To unarchive: gh project item-archive "$PROJ" --owner @me --id <id> --undo
```
</details>

**Verify:** After archiving, the item no longer appears in `item-list` output. The project still exists.

**Cleanup:**
```bash
gh project delete "$PROJ" --owner @me
# Confirm nothing left:
gh project list --owner @me --limit 100 | grep zz-project-
```
