# Drills — `gh pr create`

> **Sandbox:** run everything against `borahanmirzaii/gh-mastery-sandbox`.
> **Namespacing:** every branch and PR title must be prefixed `zz-pr-*`. Clean up at the end.

---

## Drill 1 — Create a draft PR with `--fill`

**Goal:** Push a commit to a `zz-pr-create-1` branch and open a draft PR whose title and body come entirely from the commit message — no manual input.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
cd /tmp/gh-mastery-sandbox   # your local clone of the sandbox
git checkout main && git pull

git checkout -b zz-pr-create-1
echo "# drill note" >> drill-notes.txt
git add drill-notes.txt
git commit -m "feat(drill): add drill note for create drill 1

This body text will become the PR description when --fill is used."

git push -u origin zz-pr-create-1

gh pr create \
  --base main \
  --fill \
  --draft \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh pr view --head zz-pr-create-1 --repo borahanmirzaii/gh-mastery-sandbox --json isDraft,title --jq .` → `isDraft: true`; title matches the commit subject.

**Cleanup:**
```bash
gh pr close --delete-branch \
  --repo borahanmirzaii/gh-mastery-sandbox \
  zz-pr-create-1
```

---

## Drill 2 — Override `--fill` with an explicit title

**Goal:** Create `zz-pr-create-2` with `--fill` for the body but a hand-crafted `--title`. Observe that the title is NOT taken from the commit.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
cd /tmp/gh-mastery-sandbox
git checkout main && git pull

git checkout -b zz-pr-create-2
echo "drill 2" >> drill-notes.txt
git add drill-notes.txt
git commit -m "chore: wip commit with bad message"
git push -u origin zz-pr-create-2

# --fill autofills body; --title overrides the fill for the title
gh pr create \
  --base main \
  --fill \
  --title "zz-pr-create-2: explicit title overrides fill" \
  --draft \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh pr view --head zz-pr-create-2 --repo borahanmirzaii/gh-mastery-sandbox --json title --jq .title` → the explicit title, not "chore: wip commit with bad message".

**Cleanup:**
```bash
gh pr close --delete-branch --repo borahanmirzaii/gh-mastery-sandbox zz-pr-create-2
```

---

## Drill 3 — Create a PR with a reviewer and label

**Goal:** Open `zz-pr-create-3` requesting a review from yourself (`@me`) and adding a label, all in one `gh pr create` command.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
cd /tmp/gh-mastery-sandbox
git checkout main && git pull

# Make sure a label exists (create it once if needed)
gh label create "drill" --color 0075ca --repo borahanmirzaii/gh-mastery-sandbox 2>/dev/null || true

git checkout -b zz-pr-create-3
echo "drill 3" >> drill-notes.txt
git add drill-notes.txt
git commit -m "feat(drill): reviewer + label test"
git push -u origin zz-pr-create-3

gh pr create \
  --base main \
  --fill \
  --title "zz-pr-create-3: reviewer + label" \
  --reviewer borahanmirzaii \
  --label "drill" \
  --draft \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh pr view --head zz-pr-create-3 --repo borahanmirzaii/gh-mastery-sandbox --json labels,reviewRequests --jq .` shows the label and reviewer.

**Cleanup:**
```bash
gh pr close --delete-branch --repo borahanmirzaii/gh-mastery-sandbox zz-pr-create-3
```

---

## Boss drill — Issue-linked PR with closing keyword

**Goal:** Create a sandbox issue, branch off it, open a PR whose body contains `Closes #<N>`, and verify the link appears in the PR.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
cd /tmp/gh-mastery-sandbox
git checkout main && git pull

# 1. Create a throwaway issue
ISSUE_NUM=$(gh issue create \
  --title "zz-pr-boss: test issue for closing keyword" \
  --body "This issue should auto-close when the linked PR merges." \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --json number --jq .number)

echo "Created issue #$ISSUE_NUM"

# 2. Branch
git checkout -b "zz-pr-boss-closing"
echo "boss create drill" >> drill-notes.txt
git add drill-notes.txt
git commit -m "fix(boss): close issue $ISSUE_NUM"
git push -u origin zz-pr-boss-closing

# 3. Open PR with explicit closing keyword in body
gh pr create \
  --base main \
  --title "zz-pr-boss: close issue via PR" \
  --body "This PR fixes the reported problem.

Closes #${ISSUE_NUM}" \
  --draft \
  --repo borahanmirzaii/gh-mastery-sandbox

# 4. Verify the closing reference appears
gh pr view --head zz-pr-boss-closing \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --json body,closingIssuesReferences --jq .
```
</details>

**Verify:** `closingIssuesReferences` in the JSON output lists issue `#$ISSUE_NUM`.

**Cleanup:**
```bash
# Close PR + delete branch
gh pr close --delete-branch \
  --repo borahanmirzaii/gh-mastery-sandbox \
  zz-pr-boss-closing

# Close the throwaway issue
gh issue close "$ISSUE_NUM" --repo borahanmirzaii/gh-mastery-sandbox
```
