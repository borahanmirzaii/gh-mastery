# Drills — `gh run`

> **Sandbox:** run everything against `borahanmirzaii/gh-mastery-sandbox`
> (add `--repo borahanmirzaii/gh-mastery-sandbox` to every command) unless a drill says otherwise.
> **Namespacing:** the only sandbox object created here is the `zz-run-test` branch and the `zz-run-ci`
> workflow file it holds; delete them in the cleanup step.
>
> **Setup (do this once):** The sandbox needs a workflow file on a `zz-run-*` branch before any drill
> can produce a live run. The setup below creates one via the API (no local clone needed).

## Setup — create the sandbox workflow

```bash
# Step 1: create the zz-run-test branch from main
MAIN_SHA=$(gh api repos/borahanmirzaii/gh-mastery-sandbox/git/refs/heads/main \
  --jq '.object.sha')
gh api repos/borahanmirzaii/gh-mastery-sandbox/git/refs \
  -X POST -f ref="refs/heads/zz-run-test" -f sha="$MAIN_SHA"

# Step 2: add the workflow file (triggers a run immediately via the push event)
ENCODED=$(base64 < <(cat <<'YAML'
name: zz-run-ci
on:
  push:
    branches:
      - "zz-run-*"
  workflow_dispatch:
jobs:
  hello:
    runs-on: ubuntu-latest
    steps:
      - name: Greet
        run: echo "Hello from zz-run-ci"
      - name: Show date
        run: date
YAML
))
gh api repos/borahanmirzaii/gh-mastery-sandbox/contents/.github/workflows/zz-run-ci.yml \
  -X PUT \
  -f message="chore: add zz-run-ci workflow for gh run drills" \
  -f "content=$ENCODED" \
  -f branch="zz-run-test"

# Step 3: wait ~15 s, then grab the run ID for the drills below
sleep 15
export DRILL_RUN_ID=$(gh run list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --limit 1 --json databaseId --jq '.[0].databaseId')
echo "Drill run ID: $DRILL_RUN_ID"
```

---

## Drill 1 — List and filter runs

**Goal:** List the most recent 5 runs in the sandbox, then narrow to only completed ones using `--json` and `--jq`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Plain list
gh run list --repo borahanmirzaii/gh-mastery-sandbox --limit 5

# JSON + jq — filter to completed only
gh run list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --limit 10 \
  --json databaseId,workflowName,status,conclusion,headBranch \
  --jq '.[] | select(.status=="completed")'
```
</details>

**Verify:** the output includes your `$DRILL_RUN_ID` with `"status":"completed"`.

---

## Drill 2 — Structured introspection with `view --json --jq`

**Goal:** For the drill run, emit a single JSON object containing `workflow`, `status`, `conclusion`, and `url` — the core fields of a CI dashboard entry.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh run view "$DRILL_RUN_ID" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --json workflowName,status,conclusion,url \
  --jq '{workflow:.workflowName, status:.status, conclusion:.conclusion, url:.url}'
```
</details>

**Verify:** output is a JSON object with four keys; `conclusion` should be `"success"`.

---

## Drill 3 — Triage with `--log-failed`

**Goal:** Try to view only the failed-step logs for the drill run. Understand what happens when there are no failures.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# On a successful run --log-failed produces no output (no failed steps to show)
gh run view "$DRILL_RUN_ID" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --log-failed

# To see --log-failed in action on a real failure, try a failing run from cli/cli:
FAILED_ID=$(gh run list --repo cli/cli \
  --status failure --limit 1 --json databaseId --jq '.[0].databaseId')
gh run view "$FAILED_ID" --repo cli/cli --log-failed 2>&1 | head -40
```
</details>

**Verify:** on the successful sandbox run, the command exits 0 with no output (correct — nothing failed). On a `cli/cli` failure run, it prints only the failed-step log lines.

---

## Drill 4 — Rerun and get job IDs

**Goal:** Find the job `databaseId` inside the drill run (needed for `rerun --job`), then rerun only the failed jobs (there are none, so it's a no-op rerun, but practise the command).

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# List job IDs (use databaseId, NOT the browser URL number)
gh run view "$DRILL_RUN_ID" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --json jobs \
  --jq '.jobs[] | {name:.name, id:.databaseId}'

# Rerun failed jobs only (no-op on a successful run)
gh run rerun "$DRILL_RUN_ID" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --failed
```
</details>

**Verify:** `gh run view "$DRILL_RUN_ID" --repo borahanmirzaii/gh-mastery-sandbox` shows `attempt: 2` after the rerun.

---

## Drill 5 — Watch a live run with `--exit-status`

**Goal:** Trigger a new run via `workflow_dispatch`, then watch it with `--exit-status` so the shell command exits non-zero if CI fails. This is the canonical CI-gating pattern.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Trigger a new dispatch run
gh workflow run zz-run-ci \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --ref zz-run-test

# Grab the new run ID (give it 3 s to appear)
sleep 3
NEW_RUN=$(gh run list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --workflow zz-run-ci \
  --limit 1 --json databaseId --jq '.[0].databaseId')
echo "Watching run $NEW_RUN"

# Watch with exit-status — exits non-zero if run fails
gh run watch "$NEW_RUN" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --exit-status && echo "CI green — safe to ship"
```
</details>

**Verify:** `watch` streams step progress and exits 0; the final echo fires. Run `echo $?` immediately after to confirm exit code 0.

---

## Boss drill — Trigger → watch → introspect → rerun → clean up

Chain the full CI feedback loop in one script:

1. Trigger a `workflow_dispatch` run on `zz-run-test`.
2. Watch it to completion with `--exit-status`.
3. Pull a structured JSON summary (workflow, conclusion, url).
4. Rerun with `--failed` (no-op — good practise anyway).
5. Delete the run, then delete the `zz-run-test` branch and its workflow file.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# 1. Trigger
gh workflow run zz-run-ci \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --ref zz-run-test
sleep 5

# 2. Grab the run ID and watch it
BOSS_RUN=$(gh run list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --workflow zz-run-ci --limit 1 \
  --json databaseId --jq '.[0].databaseId')
gh run watch "$BOSS_RUN" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --exit-status

# 3. JSON summary
gh run view "$BOSS_RUN" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --json workflowName,conclusion,url \
  --jq '{workflow:.workflowName, conclusion:.conclusion, url:.url}'

# 4. Rerun failed (no-op)
gh run rerun "$BOSS_RUN" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --failed

# 5. Cleanup — delete runs, then workflow file, then branch
for RUN_ID in $(gh run list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --limit 20 --json databaseId --jq '.[].databaseId'); do
  gh run delete "$RUN_ID" --repo borahanmirzaii/gh-mastery-sandbox
done

# Delete workflow file (need its SHA)
FILE_SHA=$(gh api \
  repos/borahanmirzaii/gh-mastery-sandbox/contents/.github/workflows/zz-run-ci.yml \
  -f ref="zz-run-test" --jq '.sha')
gh api \
  repos/borahanmirzaii/gh-mastery-sandbox/contents/.github/workflows/zz-run-ci.yml \
  -X DELETE \
  -f message="chore: remove zz-run-ci workflow (drill cleanup)" \
  -f sha="$FILE_SHA" \
  -f branch="zz-run-test"

# Delete branch
gh api repos/borahanmirzaii/gh-mastery-sandbox/git/refs/heads/zz-run-test -X DELETE
```
</details>

**Verify:** `gh run list --repo borahanmirzaii/gh-mastery-sandbox` returns empty. `gh api repos/borahanmirzaii/gh-mastery-sandbox/git/refs/heads/zz-run-test` returns 404.

**Cleanup confirmation:** `gh api repos/borahanmirzaii/gh-mastery-sandbox/git/refs/heads 2>&1` should list only `main`.
