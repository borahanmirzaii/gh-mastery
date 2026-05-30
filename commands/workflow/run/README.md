# `gh workflow run`

> **One-liner:** Dispatch a `workflow_dispatch` event to manually trigger a GitHub Actions workflow, optionally passing typed inputs and targeting a specific branch or SHA.

## When you reach for it

Any time you want to fire a workflow **on demand from the terminal** — a release build, a deployment, a data-migration job, a one-off report — without clicking through the GitHub UI. It is the CLI equivalent of the "Run workflow" button on the Actions tab, and it supports every feature that button does: input fields, branch selection, and JSON bulk-input.

In the solo-builder loop it appears at the boundary between code changes and CI verification: after you push a workflow change on a feature branch, `gh workflow run --ref your-branch` lets you confirm the new logic works before you merge to `main`.

## Key flags

- `-f, --raw-field key=value` — pass a `workflow_dispatch` input as a plain string. Repeat for each input. This is the standard flag for parametric dispatch: `gh workflow run deploy.yml -f env=staging -f version=1.2.3`.

- `-F, --field key=value` — like `-f` but supports `@filename` to load file contents as the value, and attempts type coercion (booleans, numbers) matching `gh api -F` semantics. Use when the input value lives in a file.

- `-r, --ref string` — **the branch or SHA on which to run the workflow** (defaults to the repo's default branch). This is the most critical flag when testing workflow changes that aren't yet merged: the workflow YAML must exist at the specified ref.

- `--json` — read all workflow inputs as a JSON object from stdin instead of repeating `-f` flags. The JSON keys must match the `inputs:` keys declared in the workflow YAML.

- `-R, --repo [HOST/]OWNER/REPO` — target a specific repo rather than inferring from the current directory.

## Examples

```bash
# Interactive: gh prompts you to pick a workflow and fill its inputs
gh workflow run

# Dispatch a specific workflow on the default branch (no inputs)
gh workflow run ci.yml --repo borahanmirzaii/gh-mastery-sandbox

# Dispatch with two string inputs (the canonical parametric form)
gh workflow run deploy.yml -f environment=staging -f version=1.2.3 --repo borahanmirzaii/gh-mastery-sandbox

# Dispatch on a feature branch — runs the version of the workflow at that ref
gh workflow run ci.yml --ref my-feature-branch --repo borahanmirzaii/gh-mastery-sandbox

# Pass all inputs at once via JSON stdin (useful when inputs come from a script)
echo '{"environment":"production","version":"2.0.0"}' | gh workflow run deploy.yml --json --repo borahanmirzaii/gh-mastery-sandbox

# Load a large input from a file using -F
gh workflow run process.yml -F payload=@./data/payload.json --repo borahanmirzaii/gh-mastery-sandbox

# Dispatch and immediately tail the resulting run
gh workflow run ci.yml --ref my-feature-branch --repo borahanmirzaii/gh-mastery-sandbox
sleep 3
gh run list --workflow ci.yml --repo borahanmirzaii/gh-mastery-sandbox --limit 1 --json databaseId --jq '.[0].databaseId' \
  | xargs -I{} gh run watch {} --exit-status --repo borahanmirzaii/gh-mastery-sandbox
```

## Gotchas

- **The workflow YAML must declare `on: workflow_dispatch:`** — without it, the API returns a 422 "Workflow does not have a `workflow_dispatch` trigger." Check with `gh workflow view <name> --yaml` before dispatching.

- **The workflow YAML must exist on the default branch, regardless of `--ref`.** `--ref` controls which code the runner checks out — it does not tell GitHub where to find the YAML. If you push a new workflow file to a feature branch but haven't merged it to `main` (or whatever the default branch is), `gh workflow run --ref feature-branch` returns a 404 "workflow not found on the default branch." Merge the YAML first (even as a draft/empty commit), then use `--ref` to run the workflow against your feature branch's application code.

- **`-f` always passes a string; `-F` coerces types.** `workflow_dispatch` inputs are always strings in the Actions model, so `-f` is generally safer for workflow inputs. Use `-F` specifically when you need `@filename` loading.

- **The command returns a run URL, but not immediately for all repos.** On a busy repo the run may not be registered by the time `gh workflow run` exits. Use `gh run list --workflow <name> --limit 3` to find the run ID, then `gh run watch <id>`.

- **Workflow identity is flexible but case-sensitive on some systems.** You can pass a numeric ID, the display name (from the `name:` field in YAML), or the filename (e.g. `ci.yml`). On Linux, the filename match is case-sensitive; on macOS it may not be. Use numeric ID for scripting: `gh workflow list --json id,name --jq '.[] | select(.name=="My Workflow") | .id'`.

- **`--json` from stdin and `-f` flags are mutually exclusive.** Mixing them causes an error. Choose one input method per invocation.

- **Inputs not declared in `on.workflow_dispatch.inputs:` are silently ignored.** If you misspell an input key, `gh workflow run` succeeds but the runner never receives that input value.

## Concepts

- [`../../../concepts/actions-model.md`](../../../concepts/actions-model.md) — covers `workflow_dispatch`, the GitHub Actions event model, how refs map to runner checkouts, and input types. _(to be written)_

## Sources

- Manual: https://cli.github.com/manual/gh_workflow_run
- Local: `gh workflow run --help` (gh 2.92.0)
