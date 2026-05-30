# Drills — `gh issue`

> **Sandbox:** run everything against `borahanmirzaii/gh-mastery-sandbox`
> (add `--repo borahanmirzaii/gh-mastery-sandbox` to every command) unless a drill says otherwise.
> **Namespacing:** create only objects prefixed `zz-issue-*` and delete them at the end, so parallel drills never collide.

---

## Drill 1 — Create an issue non-interactively

**Goal:** Create an issue with a title, body, and label entirely from flags — no prompts, no browser.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh issue create \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --title "zz-issue-drill1: non-interactive creation" \
  --body "This issue was created entirely from CLI flags." \
  --label "bug"
```

Capture the issue number from the output URL (e.g., `#7`).
</details>

**Verify:** `gh issue list --repo borahanmirzaii/gh-mastery-sandbox --search "zz-issue-drill1" --json number,title --jq '.[].title'` → prints `zz-issue-drill1: non-interactive creation`.

---

## Drill 2 — Body from stdin (heredoc)

**Goal:** Create an issue whose body is supplied via `--body-file -` and a heredoc. This is the pattern for scripted issue creation with multi-section bodies.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh issue create \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --title "zz-issue-drill2: stdin body" \
  --body-file - <<'EOF'
## Problem

Something needs fixing.

## Acceptance criteria

- [ ] It works.

**Ref:** https://github.com/borahanmirzaii/gh-mastery/blob/main/docs/superpowers/plans/2026-05-26-gh-mastery.md
EOF
```

Note the absolute URL in the body — a relative `./docs/...` path would 404 on github.com.
</details>

**Verify:** `gh issue list --repo borahanmirzaii/gh-mastery-sandbox --search "zz-issue-drill2" --json body --jq '.[].body'` → body contains "## Problem".

---

## Drill 3 — List, filter, and output as JSON

**Goal:** List open issues whose title contains `zz-issue` and output them as `#N title` pairs using `--json` + `--jq`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh issue list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --search "zz-issue in:title" \
  --state open \
  --json number,title \
  --jq '.[] | "#\(.number) \(.title)"'
```
</details>

**Verify:** Output shows at least Drill 1 and Drill 2 issues with their `#N` numbers.

---

## Drill 4 — Edit an issue (relabel + reassign)

**Goal:** Take the issue from Drill 1, add the label `documentation`, and self-assign it — all without opening the browser.

**Try it yourself first** (you'll need the issue number from Drill 1), then reveal:

<details><summary>Answer</summary>

```bash
# Replace <N> with the actual issue number from Drill 1
ISSUE_NUM=$(gh issue list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --search "zz-issue-drill1 in:title" \
  --json number --jq '.[0].number')

gh issue edit "$ISSUE_NUM" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --add-label "documentation" \
  --add-assignee @me
```
</details>

**Verify:** `gh issue view "$ISSUE_NUM" --repo borahanmirzaii/gh-mastery-sandbox --json labels,assignees --jq '{labels: [.labels[].name], assignees: [.assignees[].login]}'` → labels include `documentation`; assignees include your login.

---

## Drill 5 — Close with reason

**Goal:** Close the Drill 2 issue as "not planned" with a closing comment in a single command.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
ISSUE_NUM=$(gh issue list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --search "zz-issue-drill2 in:title" \
  --json number --jq '.[0].number')

gh issue close "$ISSUE_NUM" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --reason "not planned" \
  --comment "Closing as not planned for this drill."
```
</details>

**Verify:** `gh issue view "$ISSUE_NUM" --repo borahanmirzaii/gh-mastery-sandbox --json state,stateReason --jq '"state: \(.state) | reason: \(.stateReason)"'` → `state: CLOSED | reason: NOT_PLANNED`.

---

## Boss drill — Full solo-builder mini-workflow

**Goal:** Chain create → comment → develop (linked branch) → close → cleanup — the exact sequence this project's worker loop uses for every command group.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Step 1: Create the tracking issue
ISSUE_URL=$(gh issue create \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --title "zz-issue-boss: implement widget feature" \
  --body "$(cat <<'BODY'
## Brief

Build the widget feature.

## Acceptance criteria

- [ ] Widget renders.
- [ ] Widget has tests.

**Spec:** https://github.com/borahanmirzaii/gh-mastery-sandbox/blob/main/README.md
BODY
)" \
  --label "bug" \
  --assignee @me)

echo "Created: $ISSUE_URL"
ISSUE_NUM=$(echo "$ISSUE_URL" | grep -o '[0-9]*$')

# Step 2: Add a progress comment
gh issue comment "$ISSUE_NUM" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --body "Starting work. Branch will be created next."

# Step 3: Create the linked branch (server-side) — visible in the issue's Development panel
gh issue develop "$ISSUE_NUM" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --name "zz-issue-boss-branch"

# Step 4: Verify the branch is linked
gh issue develop "$ISSUE_NUM" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --list

# Step 5: Close the issue as completed
gh issue close "$ISSUE_NUM" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --reason completed \
  --comment "Work done. Branch zz-issue-boss-branch created and linked."

# Step 6: Cleanup — delete the issue and the linked branch
gh issue delete "$ISSUE_NUM" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --yes

gh api -X DELETE "repos/borahanmirzaii/gh-mastery-sandbox/git/refs/heads/zz-issue-boss-branch" 2>/dev/null || true
```
</details>

**Verify end state:**
```bash
# No zz-issue-boss issues should remain
gh issue list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --search "zz-issue-boss in:title" \
  --state all \
  --json number --jq 'length'
# Expected: 0
```

**Cleanup for all drills** — delete any remaining `zz-issue-*` issues:
```bash
gh issue list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --search "zz-issue in:title" \
  --state all \
  --json number \
  --jq '.[].number' | \
xargs -I{} gh issue delete {} \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --yes 2>/dev/null || true
```
