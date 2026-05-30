# Drills — `gh run watch`

> **Sandbox:** run everything against `borahanmirzaii/gh-mastery-sandbox`
> (add `--repo borahanmirzaii/gh-mastery-sandbox` to every command) unless a drill says otherwise.
> **Namespacing:** the sandbox objects used here (`zz-run-test` branch, `zz-run-ci` workflow) are
> shared with the parent `run/` drills. If you have not run the `run/` setup yet, complete it first.
>
> **Prerequisite:** the `zz-run-test` branch with `.github/workflows/zz-run-ci.yml` must exist in the sandbox.
> See `commands/run/DRILLS.md` → Setup section if it does not.

---

## Drill 1 — Watch a run with `--exit-status` (CI-gating pattern)

**Goal:** Trigger a `workflow_dispatch` run on `zz-run-test`, then watch it to completion with `--exit-status`. Verify that the shell exit code reflects the run outcome.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Trigger a dispatch run
gh workflow run zz-run-ci \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --ref zz-run-test

# Give it 5 s to appear, then grab the ID
sleep 5
RUN_ID=$(gh run list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --workflow zz-run-ci --limit 1 \
  --json databaseId --jq '.[0].databaseId')
echo "Watching run $RUN_ID"

# Watch with exit-status — exits 0 if success, non-zero if fail
gh run watch "$RUN_ID" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --exit-status
echo "Exit code: $?"
```
</details>

**Verify:** `watch` streams the "hello" and "Show date" steps, then exits. `echo $?` immediately after returns `0` (success). If you see the steps streaming live, the command is working correctly.

---

## Drill 2 — Watch in compact mode

**Goal:** Trigger another dispatch run, watch it with `--compact`, and observe that only in-progress or failed steps appear (not all passing steps).

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh workflow run zz-run-ci \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --ref zz-run-test
sleep 5
RUN_ID=$(gh run list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --workflow zz-run-ci --limit 1 \
  --json databaseId --jq '.[0].databaseId')

gh run watch "$RUN_ID" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --compact \
  --exit-status
```
</details>

**Verify:** the output is shorter than a non-compact watch — in a two-step passing run you may see only the current step or a minimal summary. Exit code 0.

---

## Boss drill — Push-to-deploy gate

Simulate the real-world CI-gating pattern: trigger a run, watch it with `--exit-status`, and only proceed to "deploy" if it passes. Then clean up.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# 1. Trigger a run
gh workflow run zz-run-ci \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --ref zz-run-test
sleep 5

# 2. Grab the new run ID
GATE_RUN=$(gh run list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --workflow zz-run-ci --limit 1 \
  --json databaseId --jq '.[0].databaseId')

# 3. Gate: watch --exit-status then conditionally deploy
if gh run watch "$GATE_RUN" \
    --repo borahanmirzaii/gh-mastery-sandbox \
    --exit-status; then
  echo "CI GREEN (run $GATE_RUN) — would deploy here"
else
  echo "CI RED (run $GATE_RUN) — aborting deploy" >&2
  exit 1
fi

# 4. Cleanup — delete all sandbox runs
for RUN_ID in $(gh run list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --limit 20 --json databaseId --jq '.[].databaseId'); do
  gh run delete "$RUN_ID" --repo borahanmirzaii/gh-mastery-sandbox
done

# Delete the workflow file, then the branch (only needed for full teardown)
FILE_SHA=$(gh api \
  "repos/borahanmirzaii/gh-mastery-sandbox/contents/.github/workflows/zz-run-ci.yml?ref=zz-run-test" \
  --jq '.sha')
gh api \
  repos/borahanmirzaii/gh-mastery-sandbox/contents/.github/workflows/zz-run-ci.yml \
  -X DELETE \
  -f message="chore: remove zz-run-ci (drill cleanup)" \
  -f sha="$FILE_SHA" \
  -f branch="zz-run-test"
gh api repos/borahanmirzaii/gh-mastery-sandbox/git/refs/heads/zz-run-test -X DELETE
echo "Sandbox clean"
```
</details>

**Verify:** `gh run list --repo borahanmirzaii/gh-mastery-sandbox` returns empty. `gh api repos/borahanmirzaii/gh-mastery-sandbox/git/refs/heads/zz-run-test` returns 404.

**Cleanup confirmation:** `gh api repos/borahanmirzaii/gh-mastery-sandbox/git/refs/heads --jq '.[].ref'` shows only `refs/heads/main`.
