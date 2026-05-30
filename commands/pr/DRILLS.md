# Drills — `gh pr`

> **Sandbox:** run everything against `borahanmirzaii/gh-mastery-sandbox`.
> Add `-R borahanmirzaii/gh-mastery-sandbox` or clone the repo and `cd` into it.
> **Namespacing:** every object you create must be prefixed `zz-pr-*`. Delete everything at the end.

---

## Drill 1 — Create a draft PR from a branch

**Goal:** Push a small change to a `zz-pr-drill-1` branch in the sandbox, open it as a draft PR using `--fill`, then inspect it with `gh pr status` and `gh pr view`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Clone the sandbox (one-time setup if not already done)
git clone git@github.com:borahanmirzaii/gh-mastery-sandbox.git /tmp/gh-mastery-sandbox
cd /tmp/gh-mastery-sandbox

# Create and push the branch
git checkout -b zz-pr-drill-1
echo "drill 1 change" >> drill-notes.txt
git add drill-notes.txt
git commit -m "chore(drill): pr drill 1 placeholder"
git push -u origin zz-pr-drill-1

# Open a draft PR — title + body autofilled from the commit
gh pr create \
  --base main \
  --fill \
  --draft \
  --title "zz-pr-drill-1: draft PR demo" \
  -R borahanmirzaii/gh-mastery-sandbox

# Inspect
gh pr status -R borahanmirzaii/gh-mastery-sandbox
gh pr view --head zz-pr-drill-1 -R borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh pr list --repo borahanmirzaii/gh-mastery-sandbox --search "zz-pr-drill-1"` shows one open draft PR.

**Cleanup:**
```bash
gh pr close --delete-branch \
  --repo borahanmirzaii/gh-mastery-sandbox \
  "$(gh pr list --repo borahanmirzaii/gh-mastery-sandbox --search 'zz-pr-drill-1' --json number --jq '.[0].number')"
```

---

## Drill 2 — Flip a draft PR to ready, then check CI

**Goal:** Create a second draft PR (`zz-pr-drill-2`), flip it to ready with `gh pr ready`, then watch for checks with `gh pr checks`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
cd /tmp/gh-mastery-sandbox
git checkout main && git pull

git checkout -b zz-pr-drill-2
echo "drill 2 change" >> drill-notes.txt
git add drill-notes.txt
git commit -m "chore(drill): pr drill 2 placeholder"
git push -u origin zz-pr-drill-2

gh pr create \
  --base main \
  --fill \
  --draft \
  --title "zz-pr-drill-2: ready-flip demo" \
  -R borahanmirzaii/gh-mastery-sandbox

# Flip out of draft
gh pr ready --repo borahanmirzaii/gh-mastery-sandbox zz-pr-drill-2

# View checks (sandbox has no CI so this will show an empty list — that's fine)
gh pr checks --repo borahanmirzaii/gh-mastery-sandbox zz-pr-drill-2
```
</details>

**Verify:** `gh pr view --repo borahanmirzaii/gh-mastery-sandbox zz-pr-drill-2 --json isDraft --jq .isDraft` → `false`.

**Cleanup:**
```bash
gh pr close --delete-branch --repo borahanmirzaii/gh-mastery-sandbox zz-pr-drill-2
```

---

## Drill 3 — Comment on a PR, then view comments

**Goal:** Create `zz-pr-drill-3`, post a comment on it with `gh pr comment`, then verify it appears in `gh pr view --comments`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
cd /tmp/gh-mastery-sandbox
git checkout main && git pull

git checkout -b zz-pr-drill-3
echo "drill 3 change" >> drill-notes.txt
git add drill-notes.txt
git commit -m "chore(drill): pr drill 3 placeholder"
git push -u origin zz-pr-drill-3

gh pr create \
  --base main \
  --fill \
  --title "zz-pr-drill-3: comment demo" \
  -R borahanmirzaii/gh-mastery-sandbox

# Leave a comment
gh pr comment zz-pr-drill-3 \
  --body "This is a drill comment — testing gh pr comment." \
  --repo borahanmirzaii/gh-mastery-sandbox

# View with comments
gh pr view zz-pr-drill-3 --comments --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh pr view --repo borahanmirzaii/gh-mastery-sandbox zz-pr-drill-3 --comments` shows your comment text.

**Cleanup:**
```bash
gh pr close --delete-branch --repo borahanmirzaii/gh-mastery-sandbox zz-pr-drill-3
```

---

## Drill 4 — Edit a PR's metadata without the web UI

**Goal:** Open `zz-pr-drill-4` and use `gh pr edit` to add a label, change the title, and add yourself as an assignee — all in one command.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
cd /tmp/gh-mastery-sandbox
git checkout main && git pull

git checkout -b zz-pr-drill-4
echo "drill 4 change" >> drill-notes.txt
git add drill-notes.txt
git commit -m "chore(drill): pr drill 4 placeholder"
git push -u origin zz-pr-drill-4

gh pr create \
  --base main \
  --fill \
  --title "zz-pr-drill-4: edit demo" \
  -R borahanmirzaii/gh-mastery-sandbox

# Edit: new title + assign yourself
gh pr edit zz-pr-drill-4 \
  --title "zz-pr-drill-4: edited title" \
  --add-assignee "@me" \
  --repo borahanmirzaii/gh-mastery-sandbox

# Confirm
gh pr view zz-pr-drill-4 --json title,assignees --jq '{title,assignees}' \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** The JSON output shows the updated title and your login under `assignees`.

**Cleanup:**
```bash
gh pr close --delete-branch --repo borahanmirzaii/gh-mastery-sandbox zz-pr-drill-4
```

---

## Boss drill — Full solo-builder PR loop

**Goal:** Chain the complete PR lifecycle: create branch → open draft PR → flip ready → leave a review comment → squash-merge with branch deletion.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
cd /tmp/gh-mastery-sandbox
git checkout main && git pull

# 1. Branch
git checkout -b zz-pr-boss
echo "boss drill change" >> drill-notes.txt
git add drill-notes.txt
git commit -m "feat(boss): boss drill change for gh-mastery"
git push -u origin zz-pr-boss

# 2. Open draft PR (--fill autofills title from commit message)
gh pr create \
  --base main \
  --fill \
  --draft \
  -R borahanmirzaii/gh-mastery-sandbox

# 3. Verify it's a draft
gh pr view --head zz-pr-boss \
  --json isDraft,number \
  --repo borahanmirzaii/gh-mastery-sandbox

# 4. Flip to ready
gh pr ready --repo borahanmirzaii/gh-mastery-sandbox zz-pr-boss

# 5. Add a review comment (approve)
PR_NUM=$(gh pr view --head zz-pr-boss \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --json number --jq .number)

gh pr review "$PR_NUM" \
  --approve \
  --body "Looks good — boss drill complete." \
  --repo borahanmirzaii/gh-mastery-sandbox

# 6. Squash-merge and delete branch
gh pr merge "$PR_NUM" \
  --squash \
  --delete-branch \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh pr view "$PR_NUM" --repo borahanmirzaii/gh-mastery-sandbox --json state --jq .state` → `"MERGED"`.

**Cleanup:** The `--delete-branch` flag in step 6 deletes both remote and local branches. Confirm no `zz-pr-*` branches remain:
```bash
gh api repos/borahanmirzaii/gh-mastery-sandbox/branches \
  --jq '[.[].name | select(startswith("zz-pr-"))] | length'
```
Expected: `0`.
