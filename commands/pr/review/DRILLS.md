# Drills — `gh pr review`

> **Sandbox:** run everything against `borahanmirzaii/gh-mastery-sandbox`.
> **Namespacing:** every branch and PR title must be prefixed `zz-pr-*`. Clean up at the end.

---

## Drill 1 — Approve a PR with a body

**Goal:** Create `zz-pr-review-1`, submit an approval with a one-line body, and verify the review appears in the PR's review list.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
cd /tmp/gh-mastery-sandbox
git checkout main && git pull

git checkout -b zz-pr-review-1
echo "review drill 1" >> drill-notes.txt
git add . && git commit -m "chore(drill): review drill 1"
git push -u origin zz-pr-review-1

gh pr create \
  --base main \
  --title "zz-pr-review-1: approve demo" \
  --body "Drill for gh pr review --approve." \
  --repo borahanmirzaii/gh-mastery-sandbox

# Approve (note: self-review may require permissive sandbox settings)
gh pr review zz-pr-review-1 \
  --approve \
  --body "LGTM — drill complete." \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh pr view zz-pr-review-1 --repo borahanmirzaii/gh-mastery-sandbox --json reviews --jq '.[reviews[0]]'` — you should see your review with state `APPROVED`.

**Cleanup:**
```bash
gh pr close --delete-branch --repo borahanmirzaii/gh-mastery-sandbox zz-pr-review-1
```

---

## Drill 2 — Request changes on a PR

**Goal:** Create `zz-pr-review-2` and submit a "request changes" review with a body explaining what needs fixing.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
cd /tmp/gh-mastery-sandbox
git checkout main && git pull

git checkout -b zz-pr-review-2
echo "review drill 2" >> drill-notes.txt
git add . && git commit -m "chore(drill): review drill 2"
git push -u origin zz-pr-review-2

PR_NUM=$(gh pr create \
  --base main \
  --title "zz-pr-review-2: request changes demo" \
  --body "Drill for request-changes review." \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --json number --jq .number)

# Request changes with mandatory body
gh pr review "$PR_NUM" \
  --request-changes \
  --body "The drill-notes.txt file needs a more descriptive entry. Please update and re-push." \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh pr view "$PR_NUM" --repo borahanmirzaii/gh-mastery-sandbox --json reviews --jq '.[reviews[].state]'` shows `CHANGES_REQUESTED`.

**Cleanup:**
```bash
gh pr close --delete-branch --repo borahanmirzaii/gh-mastery-sandbox zz-pr-review-2
```

---

## Drill 3 — Comment review vs PR comment

**Goal:** Create `zz-pr-review-3`. Post one *review comment* (`gh pr review --comment`) and one *conversation comment* (`gh pr comment`). Then inspect the PR to see where each one appears.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
cd /tmp/gh-mastery-sandbox
git checkout main && git pull

git checkout -b zz-pr-review-3
echo "review drill 3" >> drill-notes.txt
git add . && git commit -m "chore(drill): review drill 3"
git push -u origin zz-pr-review-3

PR_NUM=$(gh pr create \
  --base main \
  --title "zz-pr-review-3: review vs comment demo" \
  --body "Comparing gh pr review --comment vs gh pr comment." \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --json number --jq .number)

# Review comment (appears in the Reviews section on GitHub)
gh pr review "$PR_NUM" \
  --comment \
  --body "This is a REVIEW comment — neutral, in the Reviews section." \
  --repo borahanmirzaii/gh-mastery-sandbox

# Conversation comment (appears in the Comments thread)
gh pr comment "$PR_NUM" \
  --body "This is a CONVERSATION comment — in the general thread." \
  --repo borahanmirzaii/gh-mastery-sandbox

# Inspect both
gh pr view "$PR_NUM" --comments --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh pr view` with `--comments` shows both. The review comment shows under "Reviews"; the conversation comment shows in the comments thread.

**Cleanup:**
```bash
gh pr close --delete-branch --repo borahanmirzaii/gh-mastery-sandbox zz-pr-review-3
```

---

## Boss drill — Full review cycle: create → request-changes → re-push → approve → merge

**Goal:** Simulate a two-round review: open a PR, request changes, push a fix commit, approve, then squash-merge.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
cd /tmp/gh-mastery-sandbox
git checkout main && git pull

# Round 1: open PR
git checkout -b zz-pr-review-boss
echo "initial version" >> drill-notes.txt
git add . && git commit -m "feat(boss): initial version"
git push -u origin zz-pr-review-boss

PR_NUM=$(gh pr create \
  --base main \
  --fill \
  --title "zz-pr-review-boss: two-round review" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --json number --jq .number)

echo "PR #$PR_NUM created"

# Round 1: request changes
gh pr review "$PR_NUM" \
  --request-changes \
  --body "Please add a second line to drill-notes.txt." \
  --repo borahanmirzaii/gh-mastery-sandbox

# Author fixes the issue (a second commit)
echo "fixed version" >> drill-notes.txt
git add . && git commit -m "fix(boss): address review feedback"
git push origin zz-pr-review-boss

# Round 2: approve
gh pr review "$PR_NUM" \
  --approve \
  --body "Looks good now — LGTM." \
  --repo borahanmirzaii/gh-mastery-sandbox

# Land it
gh pr merge "$PR_NUM" \
  --squash \
  --delete-branch \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh pr view "$PR_NUM" --repo borahanmirzaii/gh-mastery-sandbox --json state,reviews --jq '{state, reviews: [.reviews[].state]}'` → `state: "MERGED"`, reviews include `CHANGES_REQUESTED` and `APPROVED`.

**Cleanup:** `--delete-branch` in the merge command handles branch removal. Confirm:
```bash
gh api repos/borahanmirzaii/gh-mastery-sandbox/branches \
  --jq '[.[].name | select(startswith("zz-pr-"))] | length'
```
Expected: `0`.
