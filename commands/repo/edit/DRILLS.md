# Drills — `gh repo edit`

> **Sandbox:** mutating drills create a disposable repo named `zz-repo-edit-*` and delete it at the end.
> **Namespacing:** only use the `zz-repo-edit-*` prefix so parallel drills never collide.

## Setup (run once before the drills)

```bash
# Create the shared drill sandbox repo
gh repo create borahanmirzaii/zz-repo-edit-sandbox \
  --public \
  --description "Edit drill sandbox — safe to delete" \
  --add-readme
```

---

## Drill 1 — Toggle features on and off

**Goal:** Enable issues and disable the wiki on `zz-repo-edit-sandbox`, then verify the settings via `--json`. Then reverse both changes.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO="borahanmirzaii/zz-repo-edit-sandbox"

# Enable issues, disable wiki
gh repo edit "$REPO" --enable-issues --enable-wiki=false

# Verify
gh repo view "$REPO" --json hasIssuesEnabled,hasWikiEnabled \
  --jq '{"issues": .hasIssuesEnabled, "wiki": .hasWikiEnabled}'
# Expected: {"issues": true, "wiki": false}

# Reverse
gh repo edit "$REPO" --enable-issues=false --enable-wiki
```
</details>

**Verify:** JSON shows `issues: true, wiki: false` after the first edit. **Cleanup:** settings reversed at the end.

---

## Drill 2 — Enforce squash-merge-only policy

**Goal:** Configure `zz-repo-edit-sandbox` to allow only squash merges (disable merge commits and rebase merges). Verify via JSON.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO="borahanmirzaii/zz-repo-edit-sandbox"

gh repo edit "$REPO" \
  --enable-squash-merge \
  --enable-merge-commit=false \
  --enable-rebase-merge=false

# Verify
gh repo view "$REPO" \
  --json squashMergeAllowed,mergeCommitAllowed,rebaseMergeAllowed \
  --jq '{squash: .squashMergeAllowed, mergeCommit: .mergeCommitAllowed, rebase: .rebaseMergeAllowed}'
# Expected: {squash: true, mergeCommit: false, rebase: false}
```
</details>

**Verify:** Only `squash` is `true`. **Cleanup:** re-enable merge commit and rebase if desired (not required — repo is deleted in teardown).

---

## Drill 3 — Add and remove topics

**Goal:** Add two topics (`gh-mastery-drill` and `automation`) to `zz-repo-edit-sandbox`, verify them, then remove one.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO="borahanmirzaii/zz-repo-edit-sandbox"

# Add topics
gh repo edit "$REPO" --add-topic gh-mastery-drill --add-topic automation

# Verify
gh repo view "$REPO" --json repositoryTopics --jq '[.repositoryTopics[].name] | sort | join(", ")'
# Expected: automation, gh-mastery-drill

# Remove one
gh repo edit "$REPO" --remove-topic automation

# Verify again
gh repo view "$REPO" --json repositoryTopics --jq '[.repositoryTopics[].name]'
# Expected: ["gh-mastery-drill"]
```
</details>

**Verify:** Topics appear after add, and only `gh-mastery-drill` remains after remove. **Cleanup:** topics will be gone when the sandbox repo is deleted in teardown.

---

## Drill 4 — Observe the wrong flag syntax producing an error

**Goal:** Confirm that `--disable-issues` does NOT work in `gh repo edit` (unlike `gh repo create`), and then use the correct `=false` form.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO="borahanmirzaii/zz-repo-edit-sandbox"

# This should fail — --disable-issues doesn't exist in edit
gh repo edit "$REPO" --disable-issues 2>&1 | head -3
# Expected: "unknown flag: --disable-issues"

# The correct form
gh repo edit "$REPO" --enable-issues=false

# Verify it actually disabled
gh repo view "$REPO" --json hasIssuesEnabled --jq .hasIssuesEnabled
# Expected: false

# Restore
gh repo edit "$REPO" --enable-issues
```
</details>

**Verify:** First command errors with "unknown flag". Second command silently succeeds. `hasIssuesEnabled` becomes `false`. **Cleanup:** issues re-enabled.

---

## Teardown

```bash
gh repo delete borahanmirzaii/zz-repo-edit-sandbox --yes
```

---

## Boss drill — Automate repo settings for a new project

**Goal:** Create a fresh repo, then apply a full "solo-builder standard" settings profile in one `gh repo edit` call: squash-merge-only, delete-branch-on-merge, issues on, wiki off, and a project topic.

<details><summary>Answer</summary>

```bash
REPO="borahanmirzaii/zz-repo-edit-boss"

# 1. Create
gh repo create "$REPO" --public --add-readme --description "Boss drill — edit settings profile"

# 2. Apply full settings profile in one shot
gh repo edit "$REPO" \
  --enable-squash-merge \
  --enable-merge-commit=false \
  --enable-rebase-merge=false \
  --delete-branch-on-merge \
  --enable-issues \
  --enable-wiki=false \
  --enable-projects \
  --add-topic gh-mastery-drill \
  --squash-merge-commit-message pr-title

# 3. Verify the full profile
gh repo view "$REPO" \
  --json squashMergeAllowed,mergeCommitAllowed,rebaseMergeAllowed,deleteBranchOnMerge,hasIssuesEnabled,hasWikiEnabled,hasProjectsEnabled,repositoryTopics \
  --jq '{
    squashOnly: (.squashMergeAllowed and (.mergeCommitAllowed | not) and (.rebaseMergeAllowed | not)),
    deleteBranchOnMerge: .deleteBranchOnMerge,
    issues: .hasIssuesEnabled,
    wiki: .hasWikiEnabled,
    projects: .hasProjectsEnabled,
    topics: [.repositoryTopics[].name]
  }'

# 4. Cleanup
gh repo delete "$REPO" --yes
```
</details>

**Verify:** `squashOnly: true`, `deleteBranchOnMerge: true`, `issues: true`, `wiki: false`, `projects: true`, topics contains `"gh-mastery-drill"`. **Cleanup:** repo deleted. Confirm:
```bash
gh repo list borahanmirzaii --json name --jq '[.[].name] | map(select(startswith("zz-repo-"))) | length'
# Expected: 0
```
