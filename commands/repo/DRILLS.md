# Drills — `gh repo`

> **Sandbox:** read-only drills target `borahanmirzaii/gh-mastery-sandbox`.
> Mutating drills create disposable repos prefixed `zz-repo-*` under your own account and delete them at the end.
> **Namespacing:** create only repos named `zz-repo-*` so parallel drills never collide.

## Drill 1 — Inspect a repo without cloning it

**Goal:** Print the description and homepage URL of `borahanmirzaii/gh-mastery-sandbox` as JSON, then open it in the browser.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# JSON output piped to jq
gh repo view borahanmirzaii/gh-mastery-sandbox --json description,homepageUrl

# Open in browser
gh repo view borahanmirzaii/gh-mastery-sandbox --web
```
</details>

**Verify:** The JSON output includes `"description"` and `"homepageUrl"` keys. The browser opens the repo page.

---

## Drill 2 — List your own repositories, filtered by visibility

**Goal:** List all public repos owned by `borahanmirzaii`, outputting only their names using `--jq`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh repo list borahanmirzaii --visibility public --json name --jq '.[].name'
```
</details>

**Verify:** Each line is a repo name (no URL, no extra fields). `gh-mastery` and `gh-mastery-sandbox` should appear.

---

## Drill 3 — Create a sandbox repo, check its settings, then delete it

**Goal:** Create a public repo named `zz-repo-drill-3` with a README, verify it exists, then delete it cleanly.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Create it
gh repo create borahanmirzaii/zz-repo-drill-3 \
  --public \
  --description "Sandbox drill — safe to delete" \
  --add-readme

# Verify it exists and is public
gh repo view borahanmirzaii/zz-repo-drill-3 --json name,visibility --jq '"\(.name) \(.visibility)"'

# Delete it (requires delete_repo scope — run gh auth refresh -s delete_repo once if needed)
gh repo delete borahanmirzaii/zz-repo-drill-3 --yes
```
</details>

**Verify:** The `view` command prints `zz-repo-drill-3 PUBLIC`. After deletion, `gh repo view borahanmirzaii/zz-repo-drill-3` returns a 404. **Cleanup:** repo is deleted in the drill itself.

---

## Drill 4 — Fork a repo and inspect the upstream remote wiring

**Goal:** Fork `borahanmirzaii/gh-mastery-sandbox` to your own account and confirm that `gh repo fork` automatically names the fork's remote `origin` and sets `upstream` to the parent.

> Note: this drill is read-only at the API level after the fork exists — delete the fork at the end.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Fork without cloning (API-only)
gh repo fork borahanmirzaii/gh-mastery-sandbox --fork-name zz-repo-fork-drill-4

# Verify the fork exists and its parent is correct
gh repo view borahanmirzaii/zz-repo-fork-drill-4 --json name,parent --jq '"\(.name) forked from \(.parent.nameWithOwner)"'

# Cleanup
gh repo delete borahanmirzaii/zz-repo-fork-drill-4 --yes
```
</details>

**Verify:** Output reads `zz-repo-fork-drill-4 forked from borahanmirzaii/gh-mastery-sandbox`. **Cleanup:** fork is deleted.

---

## Drill 5 — Attempt a repo transfer via gh api (since gh repo transfer doesn't exist)

**Goal:** Confirm that `gh repo transfer` is not a valid command, then demonstrate the correct REST API approach (dry-run — we won't actually transfer anything, just show the command shape).

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Step 1: Confirm gh repo transfer doesn't exist
gh repo transfer 2>&1 | head -3
# Expected: "unknown command "transfer" for "gh repo""

# Step 2: The correct approach — REST API (dry-run; do NOT run this on a real repo without intent)
# gh api -X POST repos/{owner}/{repo}/transfer -f new_owner=<target>
# Example shape:
echo 'gh api -X POST repos/borahanmirzaii/zz-repo-drill-3/transfer -f new_owner=some-org'
```
</details>

**Verify:** `gh repo transfer` prints an "unknown command" error. The `gh api` command shape is correct per the GitHub REST docs. **Cleanup:** nothing was mutated.

---

## Boss drill — Full repo lifecycle: create → edit → archive → delete

**Goal:** Chain `repo create`, `repo edit`, `repo archive`, and `repo delete` into a mini-workflow that mirrors what you'd do when spinning up and retiring a sandbox project.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO="borahanmirzaii/zz-repo-boss-drill"

# 1. Create the repo
gh repo create "$REPO" \
  --public \
  --description "Boss drill — lifecycle test" \
  --add-readme

# 2. Edit it — enable delete-branch-on-merge, add a topic
gh repo edit "$REPO" \
  --delete-branch-on-merge \
  --add-topic gh-mastery-drill

# 3. Verify settings took effect
gh repo view "$REPO" --json deleteBranchOnMerge,repositoryTopics \
  --jq '{"deleteBranchOnMerge": .deleteBranchOnMerge, "topics": [.repositoryTopics[].name]}'

# 4. Archive it
gh repo archive "$REPO" --yes

# 5. Verify archived
gh repo view "$REPO" --json isArchived --jq .isArchived
# Expected: true

# 6. Unarchive (so we can delete — archived repos can still be deleted but let's be tidy)
gh repo unarchive "$REPO" --yes

# 7. Delete
gh repo delete "$REPO" --yes
```
</details>

**Verify:** After step 3, `deleteBranchOnMerge` is `true` and topics include `"gh-mastery-drill"`. After step 5, `isArchived` is `true`. **Cleanup:** repo is deleted in step 7. Confirm with:
```bash
gh repo list borahanmirzaii --json name --jq '[.[].name] | map(select(startswith("zz-repo-"))) | length'
# Expected: 0
```
