# Recall — `gh workflow`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** How do you trigger a `workflow_dispatch` event from the CLI, passing two string inputs `environment=staging` and `version=1.2.3`?

<details><summary>Answer</summary>

```bash
gh workflow run deploy.yml -f environment=staging -f version=1.2.3
```

The `-f` / `--raw-field` flag (repeat once per input) dispatches a `workflow_dispatch` event. The workflow YAML must declare `on: workflow_dispatch:` with matching `inputs:` keys; otherwise the API returns a 422 error.
</details>

---

**Q2.** You push a workflow change to the branch `feat/new-ci`. How do you dispatch that new version of the workflow — not the one on `main`?

<details><summary>Answer</summary>

```bash
gh workflow run ci.yml --ref feat/new-ci
```

`--ref` controls which branch (or SHA) the workflow runs on. Without it, GitHub uses the repo's default branch. The branch must already exist on the remote; dispatching against an un-pushed ref fails silently or returns a 422.
</details>

---

**Q3.** A teammate says "the nightly workflow disappeared." You run `gh workflow list` and it's not there. What is the most likely explanation and how do you confirm it?

<details><summary>Answer</summary>

The workflow was disabled — `gh workflow list` **hides disabled workflows by default**. To confirm:

```bash
gh workflow list --all
# or with state field:
gh workflow list --all --json name,state --jq '.[] | select(.state != "active")'
```

A disabled workflow has state `disabled_manually` or `disabled_inactivity`. Re-enable with `gh workflow enable nightly.yml`.
</details>

---

**Q4.** What is the difference between `gh workflow disable <name>` and deleting the workflow YAML file from the repo?

<details><summary>Answer</summary>

- `gh workflow disable` acts at the **GitHub API layer**: the YAML file remains in the repo untouched; the workflow is toggled off (no scheduled runs, no manual triggers). It can be re-enabled with `gh workflow enable` without any git commit.
- Deleting the YAML file removes the workflow permanently from the repo's history (after a commit + push). GitHub will also automatically clean up the runs associated with a deleted workflow over time.

Use `disable` for temporary suppression (incident response, rate-limit emergencies). Use deletion for cleanup when the workflow is truly dead.
</details>

---

**Q5.** You want to pass a large JSON blob as a workflow input without writing a bunch of `-f key=value` flags. How do you do it?

<details><summary>Answer</summary>

```bash
echo '{"environment":"production","version":"2.0.0","dry_run":false}' | gh workflow run deploy.yml --json
```

`--json` reads workflow inputs as a JSON object from stdin. The keys in the JSON must match the `inputs:` keys declared in the workflow YAML. Useful when inputs are already available as structured data (e.g., from a script output).
</details>

---

**Q6.** What is the difference between `-f` and `-F` when passing workflow inputs to `gh workflow run`?

<details><summary>Answer</summary>

- `-f` / `--raw-field` — always passes a plain string, exactly as written.
- `-F` / `--field` — supports `@filename` syntax (loads file contents as the value) and attempts type coercion (booleans, numbers) similar to `gh api -F`.

For workflow inputs where the destination is always a string (which it is — `workflow_dispatch` inputs are strings in the Actions model), `-f` is safer and more predictable. Use `-F` when you need to load the value from a file: `-F payload=@payload.json`.
</details>

---

**Q8.** You push a new workflow file `ci-new.yml` to feature branch `feat/new-ci`. You then run `gh workflow run ci-new.yml --ref feat/new-ci` and get `HTTP 404: workflow ci-new.yml not found on the default branch`. What went wrong and how do you fix it?

<details><summary>Answer</summary>

`--ref` controls which branch the **runner checks out** (i.e., what application code runs), not where GitHub looks for the workflow YAML. GitHub always looks for dispatchable workflows on the **default branch** (typically `main`). If the YAML only exists on `feat/new-ci` and hasn't been merged to `main`, the dispatch fails with 404.

Fix: merge (or push) the workflow YAML to `main` first. Once it's on the default branch, you can use `--ref feat/new-ci` to run the workflow against your feature branch's code, which is useful for integration testing.
</details>

---

**Q7.** You run `gh workflow list --all --json name,state` and see `"state": "disabled_inactivity"` for one workflow. What caused this and how do you restore it?

<details><summary>Answer</summary>

GitHub **automatically disables** workflows that have had no activity (no runs triggered) for 60 days on a public repository — this is the `disabled_inactivity` state. It is not a manual disable; it happens server-side.

To restore:
```bash
gh workflow enable <workflow-name-or-file>
```

No git commit is required — it's a pure API toggle, same as re-enabling a manually disabled workflow.
</details>
