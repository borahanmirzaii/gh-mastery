# Drills — `gh workflow run`

> **Sandbox:** run everything against `borahanmirzaii/gh-mastery-sandbox`
> (add `--repo borahanmirzaii/gh-mastery-sandbox`).
> **Namespacing:** any workflow files or branches created use the `zz-workflow-*` prefix and are deleted after the boss drill.
> **Prerequisite:** the boss drill in `../DRILLS.md` walks through creating the `zz-workflow-demo.yml` fixture used here. If it doesn't exist yet in the sandbox, run that drill first.

---

## Drill 1 — Dispatch a workflow without inputs (read-only against cli/cli)

**Goal:** Identify a workflow in `cli/cli` that supports `workflow_dispatch`, then observe what `gh workflow run` would require to fire it (without actually dispatching into someone else's repo).

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Find workflows that have workflow_dispatch in their YAML
gh workflow list --repo cli/cli --json name,path | \
  jq -r '.[].path' | \
  while read p; do
    name=$(basename "$p")
    gh workflow view "$name" --yaml --repo cli/cli 2>/dev/null | grep -q 'workflow_dispatch' \
      && echo "dispatch-capable: $name"
  done
```

This surfaces which workflow files declare the trigger. Note the `inputs:` section (if any) to understand what `-f` flags would be required.
</details>

**Verify:** At least a few workflow files print `dispatch-capable: <name>`. The ones without `workflow_dispatch` are silently skipped.

---

## Drill 2 — Dispatch with `-f` inputs

**Goal:** Using the `zz-workflow-demo` workflow in the sandbox (on the `zz-workflow-demo` branch), dispatch it with `greeting=Hi` and `name=Driller` using `-f` flags.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh workflow run zz-workflow-demo \
  -f greeting=Hi \
  -f name=Driller \
  --ref zz-workflow-demo \
  --repo borahanmirzaii/gh-mastery-sandbox
```

</details>

**Verify:** Command exits 0 and prints a URL (or a "Created workflow_dispatch event" message). Then:
```bash
gh run list --workflow zz-workflow-demo --repo borahanmirzaii/gh-mastery-sandbox --limit 3
```
The run appears with status `queued` or `in_progress`.

---

## Drill 3 — Dispatch via JSON stdin

**Goal:** Dispatch `zz-workflow-demo` passing the same two inputs but using `--json` and a JSON object piped from `echo`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
echo '{"greeting":"Bonjour","name":"Monde"}' | \
  gh workflow run zz-workflow-demo \
    --json \
    --ref zz-workflow-demo \
    --repo borahanmirzaii/gh-mastery-sandbox
```

</details>

**Verify:** A new run appears in `gh run list --workflow zz-workflow-demo --repo borahanmirzaii/gh-mastery-sandbox --limit 3` triggered after the previous one.

---

## Boss drill — Dispatch, watch, and verify output end-to-end

**Goal:** Fire `zz-workflow-demo` with inputs `greeting=Ciao` and `name=Mondo` on the `zz-workflow-demo` ref, watch it complete with `gh run watch --exit-status`, then confirm the log contains the expected greeting string.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO=borahanmirzaii/gh-mastery-sandbox
REF=zz-workflow-demo

# 1. Dispatch
gh workflow run zz-workflow-demo \
  -f greeting=Ciao \
  -f name=Mondo \
  --ref $REF \
  --repo $REPO

# 2. Wait for the run to register, then grab the run ID
sleep 4
RUN_ID=$(gh run list \
  --workflow zz-workflow-demo \
  --repo $REPO \
  --limit 1 \
  --json databaseId \
  --jq '.[0].databaseId')
echo "Watching run $RUN_ID"

# 3. Stream the run to completion
gh run watch "$RUN_ID" --exit-status --repo $REPO

# 4. Confirm the log output
gh run view "$RUN_ID" --log --repo $REPO | grep "Ciao, Mondo"
```

</details>

**Verify:** Step 4 prints a line containing `Ciao, Mondo!`. The run exits with status `completed / success`.

**Cleanup:** The `zz-workflow-demo` fixture is shared with the parent DRILLS; clean it up there (remove the workflow file via a commit, as described in the parent boss drill). No additional cleanup is needed for this node's drills — they only dispatch runs, they don't create persistent objects.
