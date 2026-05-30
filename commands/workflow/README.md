# `gh workflow`

> **One-liner:** List, view, enable, disable, and manually trigger GitHub Actions workflows from the command line.

## When you reach for it

You reach for `gh workflow` any time you want to **drive GitHub Actions without touching the web UI**: firing a parametric workflow on demand, toggling a workflow on or off at the repo level, or inspecting what workflows exist and what their current state is. In the solo-builder loop it pairs with `gh run` — `workflow run` dispatches the job, `gh run watch` babysits it.

Concrete moments:
- You've merged a workflow change on a feature branch and want to test it before merging to `main` — use `workflow run --ref your-branch`.
- A scheduled workflow is hammering rate limits during an incident — disable it instantly without deleting the YAML.
- You need to trigger a release build manually, passing version inputs from the CLI instead of the UI.

## Subcommands

| Subcommand | Purpose | Node? |
|---|---|---|
| `gh workflow list` | List all workflow files (enabled by default; add `--all` for disabled) | inline |
| `gh workflow view` | Show a workflow's summary, status, or its raw YAML | inline |
| `gh workflow run` | Dispatch a `workflow_dispatch` event to trigger a workflow | [→ `run/`](./run/) |
| `gh workflow enable` | Re-enable a previously disabled workflow | inline |
| `gh workflow disable` | Disable a workflow, stopping scheduled and manual triggers | inline |

## Key flags

Global (all subcommands):
- `-R, --repo [HOST/]OWNER/REPO` — target a repo other than the one inferred from the current directory.

`workflow list`:
- `-a, --all` — include disabled workflows in the listing (disabled ones are hidden by default). **This is the most commonly missed flag** — if a workflow vanishes from `list` output, it was disabled, not deleted; add `-a` to confirm.
- `-L, --limit int` — cap results (default 50).
- `--json fields` — emit JSON with fields `id`, `name`, `path`, `state`; combine with `--jq` for scripting.

`workflow view`:
- `-r, --ref string` — view the YAML as it exists on a specific branch or tag.
- `-y, --yaml` — print the raw workflow YAML instead of the summary.
- `-w, --web` — open the workflow in the browser.

`workflow run` — see [`run/`](./run/) for full coverage; key flags here:
- `-f, --raw-field key=value` — pass a `workflow_dispatch` input as a string. Repeat for multiple inputs.
- `-F, --field key=value` — same, but respects `@`-file syntax (loads file contents as the value).
- `-r, --ref string` — **which branch or SHA to run the workflow on** (defaults to the repo's default branch). Critical when testing a workflow change that lives only on a feature branch.
- `--json` — read inputs as JSON from stdin instead of `-f` flags.

`workflow enable` / `workflow disable`:
- No meaningful extra flags beyond `--repo`. Accept `<workflow-id>`, `<workflow-name>`, or `<filename>` (e.g. `ci.yml`) as the positional argument.

## Examples

```bash
# List all workflows, including disabled ones
gh workflow list --all --repo borahanmirzaii/gh-mastery-sandbox

# View the YAML source of a specific workflow on a feature branch
gh workflow view ci.yml --yaml --ref my-feature-branch --repo borahanmirzaii/gh-mastery-sandbox

# Trigger a workflow_dispatch event with two string inputs
gh workflow run deploy.yml -f environment=staging -f version=1.2.3 --repo borahanmirzaii/gh-mastery-sandbox

# Fire the workflow as it exists on a feature branch (not main)
gh workflow run ci.yml --ref my-feature-branch --repo borahanmirzaii/gh-mastery-sandbox

# Pass complex inputs via JSON stdin
echo '{"environment":"production","version":"2.0.0"}' | gh workflow run deploy.yml --json --repo borahanmirzaii/gh-mastery-sandbox

# Disable a workflow to stop its scheduled and on-push triggers without deleting the YAML
gh workflow disable nightly.yml --repo borahanmirzaii/gh-mastery-sandbox

# Re-enable it later
gh workflow enable nightly.yml --repo borahanmirzaii/gh-mastery-sandbox

# Confirm a workflow's current state in machine-readable form
gh workflow list --all --json id,name,state --jq '.[] | select(.name == "Nightly")' --repo borahanmirzaii/gh-mastery-sandbox
```

## Gotchas

- **`workflow run` requires `on: workflow_dispatch:` in the YAML.** If the trigger is absent, the API rejects the dispatch with a 422. Check with `gh workflow view <name> --yaml` before attempting to run.

- **`--ref` controls which code the runner checks out — not where GitHub looks for the workflow YAML.** The YAML must be present on the repo's **default branch** for the workflow to be dispatchable at all. Once it is, `--ref` lets you run against any branch's code (useful for integration testing). If you want to test a workflow YAML change, you must merge (or at minimum push the YAML to default-branch-compatible location) before dispatching.

- **Disabled workflows disappear from `gh workflow list` by default.** Add `-a` / `--all` to see them. A vanished workflow almost always means someone called `gh workflow disable`, not that the file was deleted.

- **`gh workflow enable/disable` acts at the repo-level API layer, not the YAML.** The YAML file is untouched; you can re-enable without any git commit. This is different from commenting out the trigger in the file.

- **Workflow identity: name vs filename vs numeric ID.** All subcommands accept any of the three. The numeric ID is the most stable (use `--json id,name` to capture it). Filenames are convenient but must match exactly (including path-case on Linux). Workflow display names (from the `name:` field in YAML) differ from filenames.

- **`-f` vs `-F` for inputs.** `-f` (`--raw-field`) always passes a plain string. `-F` (`--field`) supports `@filename` to load file contents and attempts type coercion (booleans, numbers) — use `-f` when you want predictable string passing.

- **The run URL is returned on success, but only when the API resolves the run quickly.** For large repos the run may not be listed immediately; use `gh run list --workflow <name> --limit 5` to find it.

- **`workflow view` without `--yaml` shows the web summary, not the trigger log.** Use `gh run list --workflow <name>` to see actual execution history.

## Concepts

- [`../../concepts/actions-model.md`](../../concepts/actions-model.md) — covers the GitHub Actions event model, `workflow_dispatch`, trigger types, and how refs map to runner checkouts. _(to be written)_

## Sources

- Manual: https://cli.github.com/manual/gh_workflow
- Manual (run): https://cli.github.com/manual/gh_workflow_run
- Local: `gh workflow --help` (gh 2.92.0)
