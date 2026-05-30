# Drills — `gh issue develop`

> **Sandbox:** run most drills against `borahanmirzaii/gh-mastery-sandbox` (for the issue) and create branches there too.
> **Namespacing:** issues prefixed `zz-issue-*`; branches prefixed `zz-issue-develop-*`. Delete both at cleanup.

---

## Drill 1 — Create a linked branch and inspect the link

**Goal:** Create a sandbox issue, then use `gh issue develop` to create a server-side linked branch. Verify the link with `--list`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Step 1: Create the issue to develop against
ISSUE_URL=$(gh issue create \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --title "zz-issue-develop-d1: linked branch drill" \
  --body "Drill for gh issue develop.")
ISSUE_NUM=$(echo "$ISSUE_URL" | grep -o '[0-9]*$')
echo "Issue: $ISSUE_NUM"

# Step 2: Create the linked branch (no checkout)
gh issue develop "$ISSUE_NUM" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --name "zz-issue-develop-d1-branch"

# Step 3: Verify the link
gh issue develop "$ISSUE_NUM" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --list
```
</details>

**Verify:** `--list` output shows `zz-issue-develop-d1-branch` linked to the issue.

---

## Drill 2 — Custom base branch

**Goal:** Create a sandbox issue and a linked branch based on a specific branch (not the default). Confirm that `--base` is respected.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Assumes the sandbox has a 'main' branch (it does by default)
ISSUE_URL=$(gh issue create \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --title "zz-issue-develop-d2: custom base branch" \
  --body "Testing --base flag.")
ISSUE_NUM=$(echo "$ISSUE_URL" | grep -o '[0-9]*$')

gh issue develop "$ISSUE_NUM" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --base main \
  --name "zz-issue-develop-d2-branch"

gh issue develop "$ISSUE_NUM" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --list
```
</details>

**Verify:** `--list` shows `zz-issue-develop-d2-branch` linked to the issue.

---

## Drill 3 — Override auto-generated branch name

**Goal:** Create an issue whose auto-generated branch name would be long/messy, then override it with `--name` to something clean.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
ISSUE_URL=$(gh issue create \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --title "zz-issue-develop-d3: this title is very long and would produce an ugly branch name" \
  --body "Demonstrating --name override.")
ISSUE_NUM=$(echo "$ISSUE_URL" | grep -o '[0-9]*$')

# Without --name, the branch would be something like:
# <num>-zz-issue-develop-d3-this-title-is-very-long...
# Override to something clean:
gh issue develop "$ISSUE_NUM" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --name "zz-issue-develop-d3-clean"

gh issue develop "$ISSUE_NUM" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --list
```
</details>

**Verify:** `--list` shows `zz-issue-develop-d3-clean` — not the long auto-generated name.

---

## Boss drill — Full develop flow: issue → linked branch → verify → cleanup

**Goal:** Simulate the full solo-builder handoff pattern: create issue, develop linked branch, list it, then clean up branch and issue.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Step 1: File the issue
ISSUE_URL=$(gh issue create \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --title "zz-issue-develop-boss: feature implementation" \
  --body "$(cat <<'BODY'
## Brief

Implement the feature described in the spec.

**Spec:** https://github.com/borahanmirzaii/gh-mastery/blob/main/docs/superpowers/specs/2026-05-26-gh-mastery-design.md
BODY
)" \
  --assignee @me)
ISSUE_NUM=$(echo "$ISSUE_URL" | grep -o '[0-9]*$')
echo "Filed issue #$ISSUE_NUM"

# Step 2: Create the server-side linked branch
gh issue develop "$ISSUE_NUM" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --base main \
  --name "zz-issue-develop-boss-branch"

# Step 3: Confirm the link is visible
echo "=== Linked branches ==="
gh issue develop "$ISSUE_NUM" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --list

# Step 4: Cleanup — delete branch then issue
gh api -X DELETE \
  "repos/borahanmirzaii/gh-mastery-sandbox/git/refs/heads/zz-issue-develop-boss-branch" \
  2>/dev/null && echo "Branch deleted."

gh issue delete "$ISSUE_NUM" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --yes && echo "Issue deleted."
```
</details>

**Verify end state:**
```bash
gh issue list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --search "zz-issue-develop in:title" \
  --state all \
  --json number --jq 'length'
# Expected: 0
```

**Full cleanup — all drills:**
```bash
# Delete all zz-issue-* issues in the sandbox
gh issue list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --search "zz-issue in:title" \
  --state all \
  --json number \
  --jq '.[].number' \
| xargs -I{} gh issue delete {} \
    --repo borahanmirzaii/gh-mastery-sandbox \
    --yes 2>/dev/null || true

# Delete any leftover zz-issue-develop-* branches
for BRANCH in zz-issue-develop-d1-branch zz-issue-develop-d2-branch zz-issue-develop-d3-clean zz-issue-develop-boss-branch; do
  gh api -X DELETE \
    "repos/borahanmirzaii/gh-mastery-sandbox/git/refs/heads/$BRANCH" \
    2>/dev/null || true
done

echo "All cleaned up."
```
