# Drills — `gh repo create`

> **Sandbox:** mutating drills create repos named `zz-repo-*` under your own account and delete them at the end.
> **Namespacing:** only use the `zz-repo-*` prefix so parallel drills never collide.

## Drill 1 — Create a public repo with a README (remote-first)

**Goal:** Create a public repo named `zz-repo-create-1`, seed it with a README, verify it exists, then delete it.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Create
gh repo create borahanmirzaii/zz-repo-create-1 \
  --public \
  --description "Drill 1 — remote-first" \
  --add-readme

# Verify
gh repo view borahanmirzaii/zz-repo-create-1 --json name,visibility --jq '"\(.name) \(.visibility)"'
# Expected: zz-repo-create-1 PUBLIC

# Cleanup
gh repo delete borahanmirzaii/zz-repo-create-1 --yes
```
</details>

**Verify:** `view` prints `zz-repo-create-1 PUBLIC`. After deletion, a second `view` call returns a 404. **Cleanup:** done in drill.

---

## Drill 2 — Create from an existing local directory (local-first)

**Goal:** Initialise a local git repo in a temp directory, then publish it to GitHub using `--source=. --push`. Verify the commit appears on GitHub. Delete the repo at the end.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Set up a local repo
TMPDIR=$(mktemp -d)
cd "$TMPDIR"
git init
echo "# zz-repo-create-2" > README.md
git add README.md
git commit -m "init"

# Publish it (local-first: --source=. --push)
gh repo create borahanmirzaii/zz-repo-create-2 \
  --public \
  --source=. \
  --push

# Verify the commit landed on GitHub
gh repo view borahanmirzaii/zz-repo-create-2 --json name,isEmpty --jq '"\(.name) isEmpty=\(.isEmpty)"'
# Expected: zz-repo-create-2 isEmpty=false

# Cleanup
gh repo delete borahanmirzaii/zz-repo-create-2 --yes
cd - && rm -rf "$TMPDIR"
```
</details>

**Verify:** `isEmpty` is `false`, confirming the commit was pushed. **Cleanup:** repo deleted, temp dir removed.

---

## Drill 3 — Observe the --source and --clone exclusion

**Goal:** Confirm that using `--source` and `--clone` together produces an error, solidifying the mental model of which flag goes in which direction.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh repo create borahanmirzaii/zz-repo-create-should-fail \
  --public \
  --source=. \
  --clone 2>&1 | head -5
# Expected: an error message about --source and --clone being mutually exclusive
```
</details>

**Verify:** The command fails with an error — no repo is created. Run `gh repo list borahanmirzaii --json name --jq '[.[].name] | map(select(. == "zz-repo-create-should-fail"))' ` — should return `[]`. **Cleanup:** nothing to clean up (command errored before creating anything).

---

## Drill 4 — Create with a .gitignore template and license

**Goal:** Create a private repo seeded with the `Node` gitignore template and the `mit` license. Verify both files exist in the repo.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh repo create borahanmirzaii/zz-repo-create-4 \
  --private \
  --gitignore Node \
  --license mit \
  --add-readme

# Verify the files are present
gh api repos/borahanmirzaii/zz-repo-create-4/contents/ --jq '[.[].name] | sort | join(", ")'
# Expected: .gitignore, LICENSE, README.md

# Cleanup
gh repo delete borahanmirzaii/zz-repo-create-4 --yes
```
</details>

**Verify:** `.gitignore`, `LICENSE`, and `README.md` all appear in the contents listing. **Cleanup:** done in drill.

---

## Boss drill — Full create → configure → verify workflow

**Goal:** Chain `repo create` with `repo edit` to simulate a realistic project setup: create with defaults, then layer on settings that can't be set at create time (e.g., enabling squash-merge-only, adding topics).

<details><summary>Answer</summary>

```bash
REPO="borahanmirzaii/zz-repo-create-boss"

# 1. Create the repo (minimal flags — features configured in step 2)
gh repo create "$REPO" \
  --public \
  --description "Boss drill — repo create + edit workflow" \
  --add-readme

# 2. Configure: squash-merge only, delete branch on merge, add topic
gh repo edit "$REPO" \
  --enable-squash-merge \
  --enable-merge-commit=false \
  --enable-rebase-merge=false \
  --delete-branch-on-merge \
  --add-topic gh-mastery-drill

# 3. Verify everything landed
gh repo view "$REPO" \
  --json squashMergeAllowed,mergeCommitAllowed,rebaseMergeAllowed,deleteBranchOnMerge,repositoryTopics \
  --jq '{
    squash: .squashMergeAllowed,
    mergeCommit: .mergeCommitAllowed,
    rebase: .rebaseMergeAllowed,
    deleteBranchOnMerge: .deleteBranchOnMerge,
    topics: [.repositoryTopics[].name]
  }'
# Expected: squash=true, mergeCommit=false, rebase=false, deleteBranchOnMerge=true, topics=["gh-mastery-drill"]

# 4. Cleanup
gh repo delete "$REPO" --yes
```
</details>

**Verify:** JSON output shows only squash-merge enabled, `deleteBranchOnMerge` true, and topic `gh-mastery-drill` present. **Cleanup:** repo deleted in step 4. Confirm no `zz-repo-*` linger:
```bash
gh repo list borahanmirzaii --json name --jq '[.[].name] | map(select(startswith("zz-repo-"))) | length'
# Expected: 0
```
