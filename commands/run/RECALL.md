# Recall — `gh run`

Spaced-repetition self-test. Cover the answer, recall it, then check.

---

**Q1.** You push a commit and want a shell script to block until CI finishes, then only continue if the run *succeeded*. What single `gh run` command achieves this?

<details><summary>Answer</summary>

```bash
gh run watch "$RUN_ID" --exit-status && next_step.sh
```

`--exit-status` makes `watch` exit non-zero when the run fails. Without it, `watch` always exits 0 — the gate would silently pass even on failure.
</details>

---

**Q2.** A workflow run failed. You want to see only the log lines from the steps that failed — not the full run log. Which flag do you use, and on which subcommand?

<details><summary>Answer</summary>

`gh run view <run-id> --log-failed`

`--log-failed` prints only the failed-step logs. The alternative `--log` dumps the entire run log (often megabytes), making triage far slower.
</details>

---

**Q3.** You want to script a CI dashboard that shows, for each run, the workflow name, status, conclusion, and HTML URL. Which command and flags give you a structured JSON object per run?

<details><summary>Answer</summary>

```bash
gh run view <run-id> \
  --json workflowName,status,conclusion,url \
  --jq '{workflow:.workflowName, status:.status, conclusion:.conclusion, url:.url}'
```

Or for a list view across runs:

```bash
gh run list --limit 20 \
  --json workflowName,status,conclusion,url \
  --jq '.[] | {workflow:.workflowName, conclusion:.conclusion, url:.url}'
```

The available fields on `view` include `jobs`, `headBranch`, `headSha`, `databaseId`, `createdAt`, `startedAt`, `updatedAt`.
</details>

---

**Q4.** You want to rerun only the failed jobs in run `12345`, not the whole run. What is the command? What is the difference from `gh run rerun 12345` (no flag)?

<details><summary>Answer</summary>

```bash
gh run rerun 12345 --failed
```

Without `--failed`, the *entire* run restarts from scratch (all jobs). `--failed` restarts only the failed jobs and their dependencies — faster and cheaper for flaky-test situations.
</details>

---

**Q5.** You want to rerun a specific job within a run. You open the run in the browser; the URL ends in `/jobs/3`. Can you pass `3` to `gh run rerun --job 3`?

<details><summary>Answer</summary>

No. The number in the browser URL is a display index, not the `databaseId`. Passing it to `--job` returns `404 NOT FOUND`.

Get the correct ID first:

```bash
gh run view <run-id> --json jobs --jq '.jobs[] | {name:.name, id:.databaseId}'
```

Then use the `databaseId` value with `--job`.
</details>

---

**Q6.** You filter runs with `gh run list -w my-workflow`. It returns nothing even though you know runs exist. What is the likely cause and the fix?

<details><summary>Answer</summary>

The workflow is probably **disabled**. By default `gh run list` does not fetch runs from disabled workflows. Fix: add `-a` / `--all`:

```bash
gh run list -w my-workflow --all
```
</details>

---

**Q7.** `gh run watch` fails with an authentication error even though other `gh` commands work fine. What is the most likely cause?

<details><summary>Answer</summary>

`gh run watch` requires the `checks:read` permission, which **cannot be granted to fine-grained PATs**. The token in use is likely a fine-grained PAT. Switch to a classic token (or use `gh auth refresh` with a classic scope set) to fix it.
</details>

---

**Q8.** You want to list only failed runs on the `main` branch across the last 100 runs. Write the command.

<details><summary>Answer</summary>

```bash
gh run list --branch main --status failure --limit 100
```

Or, using JSON to filter on `conclusion` (which is more precise than `--status failure` for completed runs):

```bash
gh run list --branch main --limit 100 \
  --json databaseId,workflowName,conclusion,url \
  --jq '.[] | select(.conclusion=="failure")'
```
</details>
