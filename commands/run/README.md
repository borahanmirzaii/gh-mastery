# `gh run`

> **One-liner:** Inspect, watch, and manage GitHub Actions workflow runs from the terminal.

## When you reach for it

Any time you need CI feedback without opening a browser: after a push, before a deploy, or when triaging a red build. In the solo-builder loop, `gh run` sits between "code pushed" and "PR ready" — it's how you gate the next step on CI green.

## Subcommands

| Subcommand | Purpose | Node? |
|---|---|---|
| `gh run list` | List recent runs (filter by branch, status, workflow, user) | inline |
| `gh run view` | Show a run summary; view logs; emit JSON for scripting | inline |
| `gh run watch` | Stream live progress until a run completes | [→ `watch/`](./watch/) |
| `gh run rerun` | Retry a run, only its failed jobs, or one specific job | inline |
| `gh run cancel` | Cancel an in-progress run | inline |
| `gh run delete` | Delete a run record | inline |
| `gh run download` | Download artifacts produced by a run | inline |

## Key flags

### Shared
- `-R, --repo [HOST/]OWNER/REPO` — operate on a different repo without changing your working directory.
- `--json <fields>` — emit JSON; combine with `--jq` for one-liner queries or `--template` for Go templates.
- `-q, --jq <expr>` — apply a jq expression to JSON output (available on `list` and `view`).

### `gh run list`
- `-L, --limit <n>` — fetch up to `n` runs (default 20, max 1000).
- `-b, --branch <branch>` — filter by branch.
- `-s, --status <status>` — filter by status: `queued`, `in_progress`, `completed`, `failure`, `success`, `cancelled`, etc.
- `-w, --workflow <name|file|id>` — filter by workflow name, filename, or ID.
- `-e, --event <event>` — filter by trigger event (`push`, `pull_request`, `workflow_dispatch`, …).
- `-a, --all` — include runs from disabled workflows.

### `gh run view`
- `--log` — print the full log for a run or job.
- `--log-failed` — print **only** the logs of failed steps — much faster than `--log` for triage.
- `-j, --job <id>` — focus on a specific job ID (use `--json jobs --jq` to find IDs).
- `--exit-status` — exit non-zero if the run failed (useful in scripts).
- `-v, --verbose` — show individual job steps in the summary.
- `-w, --web` — open the run in the browser.

### `gh run watch`
- `--exit-status` — exit non-zero when the run fails (the canonical CI-gating pattern).
- `--compact` — show only relevant/failed steps instead of all steps.
- `-i, --interval <sec>` — polling interval (default 3 s).

### `gh run rerun`
- `--failed` — rerun only the failed jobs (plus their dependencies) instead of the whole run.
- `-j, --job <id>` — rerun a single job. **Gotcha:** the job `<number>` in the browser URL is *not* the `databaseId` — see Gotchas below.
- `-d, --debug` — rerun with debug logging enabled.

### `gh run cancel`
- `--force` — force-cancel a run that is stuck.

### `gh run download`
- `-n, --name <name>` — download a named artifact (repeatable).
- `-p, --pattern <glob>` — download artifacts matching a glob (repeatable).
- `-D, --dir <dir>` — target directory (default: current dir).

## Examples

```bash
# 1. List the last 10 runs on the current repo, across all workflows
gh run list --limit 10

# 2. Filter to failed runs on the main branch
gh run list --branch main --status failure

# 3. View a run summary (interactive picker if run-id omitted)
gh run view 26679159041 --repo borahanmirzaii/gh-mastery-sandbox

# 4. Structured introspection — get workflow name, status, conclusion, and URL
gh run view 26679159041 \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --json workflowName,status,conclusion,url \
  --jq '{workflow:.workflowName, status:.status, conclusion:.conclusion, url:.url}'

# 5. List all job IDs for a run (needed before rerunning a specific job)
gh run view 26679159041 \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --json jobs \
  --jq '.jobs[] | {name:.name, id:.databaseId}'

# 6. Show only failed-step logs — fastest triage path
gh run view 26679159041 \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --log-failed

# 7. Watch a run live and exit non-zero if it fails (CI-gating pattern)
RUN_ID=$(gh run list --repo borahanmirzaii/gh-mastery-sandbox \
  --limit 1 --json databaseId --jq '.[0].databaseId')
gh run watch "$RUN_ID" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --exit-status && echo "CI green — safe to proceed"

# 8. Rerun only the failed jobs
gh run rerun 26679159041 \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --failed

# 9. Cancel an in-progress run
gh run cancel <run-id> --repo borahanmirzaii/gh-mastery-sandbox

# 10. Download all artifacts from a run into ./artifacts/
gh run download 26679159041 \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --dir ./artifacts/

# 11. Build a minimal CI dashboard (latest run per workflow)
gh run list --repo cli/cli --limit 50 \
  --json workflowName,status,conclusion,url \
  --jq 'group_by(.workflowName) | map({workflow:.[0].workflowName, conclusion:.[0].conclusion, url:.[0].url})'
```

## Gotchas

1. **`gh run watch --exit-status` is the canonical CI-gating pattern.** When you push a commit and want a script to block until CI finishes — succeeding or failing — chain it like:
   ```bash
   gh run watch "$RUN_ID" --exit-status && deploy.sh
   ```
   Without `--exit-status`, `watch` always exits 0 (even if the run failed), so the gate silently passes on failure.

2. **`--log-failed` vs `--log`:** `--log` dumps every step's log for the entire run — often megabytes of output. `--log-failed` prints only the logs of failed steps, making triage dramatically faster. Always reach for `--log-failed` first.

3. **`view --json` + `--jq` is the foundation of any CI dashboard script.** Available JSON fields include `workflowName`, `status`, `conclusion`, `headBranch`, `headSha`, `jobs`, `url`, `databaseId`, `createdAt`, `startedAt`, `updatedAt`. Pipe through `jq` to filter, reshape, or group across runs.

4. **`rerun --job` requires `databaseId`, not the browser job number.** When you click a job in `https://github.com/<owner>/<repo>/actions/runs/<run-id>/jobs/<number>`, that `<number>` is a display index — passing it to `--job` returns `404 NOT FOUND`. Get the real ID first:
   ```bash
   gh run view <run-id> --json jobs --jq '.jobs[] | {name:.name, id:.databaseId}'
   ```

5. **`gh run watch` does not support fine-grained PATs.** Fine-grained tokens cannot be granted `checks:read` permission, so `watch` requires a classic token. If `watch` fails with an auth error, check your token type.

6. **`gh run list` skips disabled workflows by default.** Pass `-a` / `--all` to include runs from disabled workflows. This also applies when filtering by `-w workflow_name`.

7. **Run IDs are repository-scoped.** A `databaseId` (e.g. `26679159041`) is valid only within its repo. Always pass `-R` when scripting across repos.

8. **`gh run rerun` vs `gh run rerun --failed`:** without `--failed`, the *entire* run restarts (all jobs). `--failed` restarts only failed jobs plus their dependencies — cheaper and faster for flaky-test reruns.

## Concepts

- [GitHub Actions model](../../concepts/actions-model.md) _(to be written)_ — how workflows, jobs, steps, triggers, and run IDs relate.

## Sources

- Manual: https://cli.github.com/manual/gh_run
- Local: `gh run --help` (gh 2.93.0)
