# Drills — `gh workflow`

> **Sandbox:** run everything against `borahanmirzaii/gh-mastery-sandbox`
> (add `--repo borahanmirzaii/gh-mastery-sandbox` or set `export REPO=borahanmirzaii/gh-mastery-sandbox` as a shorthand).
> **Namespacing:** any workflow files created are named `zz-workflow-*.yml`. Delete them at the end of the boss drill.
> **Read-only drills** (1, 2) target `cli/cli` so you can run them without touching the sandbox.

---

## Drill 1 — Discover workflows in a public repo (read-only)

**Goal:** List all workflows in `cli/cli`, including any that are disabled, and identify their states.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh workflow list --all --repo cli/cli --json id,name,state --jq '.[] | "\(.state)\t\(.name)"'
```

You can also use the plain table form:
```bash
gh workflow list --all --repo cli/cli
```

</details>

**Verify:** Output includes at least one row. Any disabled workflow has `disabled_manually` (or `disabled_inactivity`) in its `state` field when using `--json`. Without `--all`, disabled workflows are absent from the list.

---

## Drill 2 — Inspect a workflow's YAML on a specific ref (read-only)

**Goal:** In `cli/cli`, view the raw YAML of the `codeql-analysis.yml` (or any workflow you found in Drill 1) as it exists on the `trunk` branch.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Pick any workflow name from Drill 1's output, e.g. "CodeQL"
gh workflow view "CodeQL" --yaml --ref trunk --repo cli/cli

# Or use the filename directly
gh workflow view codeql-analysis.yml --yaml --ref trunk --repo cli/cli 2>/dev/null || \
  gh workflow view --yaml --repo cli/cli   # interactive picker if the name differs
```

</details>

**Verify:** Raw YAML prints to stdout. Look for the `on:` block to confirm the triggers defined for that ref.

---

## Drill 3 — Add a `workflow_dispatch` workflow to the sandbox

**Goal:** Add a minimal workflow file that supports `workflow_dispatch` with two string inputs (`greeting` and `name`) to the sandbox's **default branch** (`main`) and verify it appears in `gh workflow list`.

> **Key insight:** The workflow YAML must be on the default branch to be dispatchable. Adding it only to a feature branch is not enough — GitHub looks for the YAML on `main` (or the configured default branch) when you call `gh workflow run`. The `--ref` flag controls what code the runner checks out, not where the YAML lives.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO=borahanmirzaii/gh-mastery-sandbox

# Option A: via a local clone
git clone git@github.com:$REPO.git /tmp/gh-mastery-sandbox 2>/dev/null || true
cd /tmp/gh-mastery-sandbox && git checkout main && git pull

mkdir -p .github/workflows
cat > .github/workflows/zz-workflow-demo.yml << 'EOF'
name: zz-workflow-demo
on:
  workflow_dispatch:
    inputs:
      greeting:
        description: "Greeting word"
        required: true
        default: "Hello"
      name:
        description: "Name to greet"
        required: false
        default: "World"
jobs:
  greet:
    runs-on: ubuntu-latest
    steps:
      - run: echo "${{ github.event.inputs.greeting }}, ${{ github.event.inputs.name }}!"
EOF

git add .github/workflows/zz-workflow-demo.yml
git commit -m "chore: add zz-workflow-demo for gh workflow drills"
git push origin main

# Option B: via gh api (no clone needed)
# 1. Compute the base64 of the YAML, then:
gh api repos/$REPO/contents/.github/workflows/zz-workflow-demo.yml \
  -X PUT \
  -f "content=<base64-of-yaml>" \
  -f "message=chore: add zz-workflow-demo for gh workflow drills" \
  -f "branch=main"
```

Also create a `zz-workflow-demo` feature branch for the `--ref` drill:
```bash
SHA=$(gh api repos/$REPO/git/refs/heads/main --jq '.object.sha')
gh api repos/$REPO/git/refs -f ref="refs/heads/zz-workflow-demo" -f sha="$SHA"
```

</details>

**Verify:** `zz-workflow-demo` appears in `gh workflow list --repo borahanmirzaii/gh-mastery-sandbox` (default list, no `--all` needed) with state `active`.

---

## Drill 4 — Disable and re-enable a workflow

**Goal:** Disable the `zz-workflow-demo` workflow you created in Drill 3, confirm it disappears from the default listing, then re-enable it and confirm it reappears.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO=borahanmirzaii/gh-mastery-sandbox

# Disable
gh workflow disable zz-workflow-demo --repo $REPO

# Default list: zz-workflow-demo should be absent
gh workflow list --repo $REPO

# Full list: should show it as disabled
gh workflow list --all --repo $REPO --json name,state

# Re-enable
gh workflow enable zz-workflow-demo --repo $REPO

# Confirm it's active again
gh workflow list --repo $REPO
```

</details>

**Verify:** After disable, `zz-workflow-demo` is absent from plain `workflow list` but visible with `--all`. After enable, it reappears in the default listing with state `active`.

---

## Boss drill — Full dispatch-and-observe cycle

**Goal:** Using the sandbox workflow from Drill 3 (file on `main`), dispatch `zz-workflow-demo` with inputs `greeting=Howdy name=Partner` targeted at the `zz-workflow-demo` feature branch using `--ref`. Watch the run complete, verify the log, then clean up.

> **Reminder:** `--ref` controls which code the runner checks out. The YAML must already be on `main` (Drill 3) for dispatch to succeed — `--ref` does not change where GitHub finds the YAML.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO=borahanmirzaii/gh-mastery-sandbox
BRANCH=zz-workflow-demo

# 1. Dispatch with inputs targeting the feature branch ref
gh workflow run zz-workflow-demo.yml \
  -f greeting=Howdy \
  -f name=Partner \
  --ref $BRANCH \
  --repo $REPO

# 2. Give GitHub a moment to register the run, then find the run ID
sleep 4
RUN_ID=$(gh run list \
  --workflow zz-workflow-demo.yml \
  --repo $REPO \
  --limit 1 \
  --json databaseId \
  --jq '.[0].databaseId')
echo "Run ID: $RUN_ID"

# 3. Watch it to completion (exits non-zero if the run fails)
gh run watch "$RUN_ID" --exit-status --repo $REPO

# 4. View the log to confirm the echo step printed the expected string
gh run view "$RUN_ID" --log --repo $REPO | grep "Howdy, Partner"

# 5. Cleanup: delete the workflow file from main via the API or a local clone
gh api repos/$REPO/contents/.github/workflows/zz-workflow-demo.yml \
  --jq '"sha=\(.sha)"'
# Then delete with the sha:
SHA=$(gh api "repos/$REPO/contents/.github/workflows/zz-workflow-demo.yml" --jq '.sha')
gh api repos/$REPO/contents/.github/workflows/zz-workflow-demo.yml \
  -X DELETE \
  -f message="chore: remove zz-workflow-demo drill fixture" \
  -f sha="$SHA" \
  -f branch="main"

# 6. Delete the feature branch
gh api -X DELETE "repos/$REPO/git/refs/heads/$BRANCH"
```

</details>

**Verify:** Step 4 outputs a line containing `Howdy, Partner!`. After cleanup:
- `gh workflow list --all --repo borahanmirzaii/gh-mastery-sandbox` shows no `zz-workflow-*` entries.
- `gh api repos/borahanmirzaii/gh-mastery-sandbox/git/refs/heads/zz-workflow-demo` returns 404.

**Cleanup checklist:**
- [ ] `zz-workflow-demo.yml` deleted from `main` (and any other branch).
- [ ] `zz-workflow-demo` branch deleted.
- [ ] `gh workflow list --all --repo borahanmirzaii/gh-mastery-sandbox` shows no `zz-workflow-*` entries.
