# Drills — `gh pr merge`

> **Sandbox:** run everything against `borahanmirzaii/gh-mastery-sandbox`.
> **Namespacing:** every branch and PR title must be prefixed `zz-pr-*`. Clean up at the end.

---

## Drill 1 — Squash-merge and delete branch

**Goal:** Open `zz-pr-merge-1`, then squash-merge it with branch deletion — the project convention one-liner.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
cd /tmp/gh-mastery-sandbox
git checkout main && git pull

# Set up a branch with two commits (to see the squash effect)
git checkout -b zz-pr-merge-1
echo "commit A" >> drill-notes.txt && git add . && git commit -m "chore: merge drill commit A"
echo "commit B" >> drill-notes.txt && git add . && git commit -m "chore: merge drill commit B"
git push -u origin zz-pr-merge-1

gh pr create \
  --base main \
  --title "zz-pr-merge-1: squash demo" \
  --body "Two commits squashed into one." \
  --repo borahanmirzaii/gh-mastery-sandbox

PR_NUM=$(gh pr list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --head zz-pr-merge-1 \
  --json number --jq '.[0].number')

# Squash-merge + delete branch
gh pr merge "$PR_NUM" \
  --squash \
  --delete-branch \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh pr view "$PR_NUM" --repo borahanmirzaii/gh-mastery-sandbox --json state --jq .state` → `"MERGED"`. Branch `zz-pr-merge-1` no longer exists on the remote.

---

## Drill 2 — Enable auto-merge

**Goal:** Open `zz-pr-merge-2`, enable auto-merge with `--auto`, then disable it with `--disable-auto`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
cd /tmp/gh-mastery-sandbox
git checkout main && git pull

git checkout -b zz-pr-merge-2
echo "auto-merge drill" >> drill-notes.txt
git add . && git commit -m "chore: auto-merge drill"
git push -u origin zz-pr-merge-2

gh pr create \
  --base main \
  --title "zz-pr-merge-2: auto-merge demo" \
  --body "Testing auto-merge flag." \
  --repo borahanmirzaii/gh-mastery-sandbox

PR_NUM=$(gh pr list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --head zz-pr-merge-2 \
  --json number --jq '.[0].number')

# Enable auto-merge (sandbox has no required checks, so this may merge immediately)
gh pr merge "$PR_NUM" \
  --squash \
  --auto \
  --repo borahanmirzaii/gh-mastery-sandbox 2>&1 || echo "(auto-merge may not be supported on sandbox; that is expected)"

# If still open, disable auto-merge then close manually
gh pr merge "$PR_NUM" --disable-auto \
  --repo borahanmirzaii/gh-mastery-sandbox 2>&1 || true
```
</details>

**Verify:** `gh pr view "$PR_NUM" --repo borahanmirzaii/gh-mastery-sandbox --json autoMergeRequest --jq .` shows `null` after `--disable-auto`.

**Cleanup:**
```bash
gh pr close --delete-branch --repo borahanmirzaii/gh-mastery-sandbox zz-pr-merge-2 2>/dev/null || true
```

---

## Drill 3 — Merge by PR number from any directory

**Goal:** Without being on the PR branch, merge `zz-pr-merge-3` by its PR number using `-R` to target the sandbox.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
cd /tmp/gh-mastery-sandbox
git checkout main && git pull

git checkout -b zz-pr-merge-3
echo "remote merge drill" >> drill-notes.txt
git add . && git commit -m "chore: remote merge drill"
git push -u origin zz-pr-merge-3

PR_NUM=$(gh pr create \
  --base main \
  --title "zz-pr-merge-3: merge by number" \
  --body "Merged by PR number from main branch." \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --json number --jq .number)

# Switch back to main — we are NOT on the PR branch
git checkout main

# Merge by number
gh pr merge "$PR_NUM" \
  --squash \
  --delete-branch \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh pr view "$PR_NUM" --repo borahanmirzaii/gh-mastery-sandbox --json state --jq .state` → `"MERGED"`.

---

## Boss drill — Full PR lifecycle: create → review → merge

**Goal:** Chain `gh pr create`, `gh pr review --approve`, and `gh pr merge --squash --delete-branch` into a complete PR lifecycle script.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
cd /tmp/gh-mastery-sandbox
git checkout main && git pull

# 1. Branch and commit
git checkout -b zz-pr-merge-boss
echo "boss merge drill" >> drill-notes.txt
git add . && git commit -m "feat(boss): complete PR lifecycle drill"
git push -u origin zz-pr-merge-boss

# 2. Create PR (not draft this time — ready for review)
PR_URL=$(gh pr create \
  --base main \
  --fill \
  --title "zz-pr-merge-boss: full lifecycle" \
  --repo borahanmirzaii/gh-mastery-sandbox)

PR_NUM=$(echo "$PR_URL" | grep -o '[0-9]*$')
echo "PR #$PR_NUM created: $PR_URL"

# 3. Approve the PR (reviewing your own PR requires self-review permissions on the repo)
gh pr review "$PR_NUM" \
  --approve \
  --body "Self-approval for drill purposes." \
  --repo borahanmirzaii/gh-mastery-sandbox

# 4. Squash-merge + delete branch
gh pr merge "$PR_NUM" \
  --squash \
  --delete-branch \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh pr view "$PR_NUM" --repo borahanmirzaii/gh-mastery-sandbox --json state,mergedAt --jq .` → `state: "MERGED"` with a non-null `mergedAt` timestamp.

**Cleanup:** The `--delete-branch` flag handles branch cleanup. Confirm no `zz-pr-*` branches linger:
```bash
gh api repos/borahanmirzaii/gh-mastery-sandbox/branches \
  --jq '[.[].name | select(startswith("zz-pr-"))] | length'
```
Expected: `0`.
