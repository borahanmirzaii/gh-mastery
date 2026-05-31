# Recall — `gh workflow run`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** What GitHub Actions trigger must a workflow declare before `gh workflow run` can dispatch it?

<details><summary>Answer</summary>

The workflow YAML must include `on: workflow_dispatch:` (at the top-level `on:` key). Without it the API returns HTTP 422 — "Workflow does not have a `workflow_dispatch` trigger." Confirm with `gh workflow view <name> --yaml`.
</details>

---

**Q2.** How do you dispatch `deploy.yml` with the input `environment=staging` and also pass a `version` value loaded from a local file `./version.txt`?

<details><summary>Answer</summary>

```bash
gh workflow run deploy.yml \
  -f environment=staging \
  -F version=@./version.txt
```

Mix `-f` (plain string) and `-F` (file-load via `@` syntax) freely. `-F` reads the file contents and passes them as the input value.
</details>

---

**Q3.** You changed `ci.yml` on branch `feat/faster-ci` and pushed it. How do you test that version of the workflow without merging to `main`?

<details><summary>Answer</summary>

This depends on what "test that version of the workflow" means:

- **Testing the workflow YAML change itself** (new steps, new triggers): You cannot dispatch it from `feat/faster-ci` alone — `gh workflow run ci.yml --ref feat/faster-ci` returns a 404 if `ci.yml` isn't on the default branch. The YAML change must reach the default branch first (merge it, or temporarily push to `main`) before it's dispatchable.

- **Testing your application code changes against the existing workflow**: `--ref` works here. If `ci.yml` is already on `main` and you just want the runner to check out `feat/faster-ci`'s application code:

```bash
gh workflow run ci.yml --ref feat/faster-ci
```

The runner uses the `ci.yml` from `main` but executes against `feat/faster-ci`'s checked-out code.
</details>

---

**Q4.** After running `gh workflow run deploy.yml -f env=prod`, you don't see a run URL in the output. How do you find the resulting run ID?

<details><summary>Answer</summary>

```bash
gh run list --workflow deploy.yml --limit 5
# or for just the ID:
gh run list --workflow deploy.yml --limit 1 --json databaseId --jq '.[0].databaseId'
```

The run URL isn't always returned immediately (especially on busy repos). `gh run list --workflow <name>` is the reliable fallback — look for the most recently created run.
</details>

---

**Q5.** What happens if you pass an input key via `-f` that is not declared in the workflow's `on.workflow_dispatch.inputs:` block?

<details><summary>Answer</summary>

`gh workflow run` succeeds (exit 0) and the dispatch event is created — GitHub does not reject undeclared inputs at the API level. However, the runner **silently ignores** any input key that isn't in `inputs:`. The workflow receives only the declared inputs; the extra key is discarded. This is a silent footgun — always verify input names against the YAML.
</details>

---

**Q6.** You want to pass five inputs to a workflow and they are already available as a JSON object in a shell variable. What is the most ergonomic dispatch command?

<details><summary>Answer</summary>

```bash
INPUTS='{"environment":"staging","version":"1.2.3","dry_run":"true","region":"us-east-1","notify":"true"}'
echo "$INPUTS" | gh workflow run deploy.yml --json
```

`--json` reads the entire input map from stdin in one shot. The JSON keys must match the `inputs:` keys in the workflow YAML exactly. This avoids repeating `-f key=value` five times and is easier to script.
</details>
