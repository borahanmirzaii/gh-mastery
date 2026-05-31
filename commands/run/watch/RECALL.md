# Recall — `gh run watch`

Spaced-repetition self-test. Cover the answer, recall it, then check.

---

**Q1.** What does `gh run watch` exit with by default when a run *fails*?

<details><summary>Answer</summary>

Exit **0**. By default `watch` exits 0 regardless of the run outcome. You must add `--exit-status` to get a non-zero exit on failure. This is the most common footgun with `watch`.
</details>

---

**Q2.** Write the one-liner CI-gating pattern: watch run `$RUN_ID` in repo `owner/repo` and only run `./ship.sh` if it passes.

<details><summary>Answer</summary>

```bash
gh run watch "$RUN_ID" --repo owner/repo --exit-status && ./ship.sh
```

`&&` short-circuits — `./ship.sh` is skipped when `watch` exits non-zero (run failed).
</details>

---

**Q3.** You have a large matrix workflow with 30 steps. Most pass. What flag shows only the relevant/failed steps to keep output readable?

<details><summary>Answer</summary>

`--compact`

```bash
gh run watch "$RUN_ID" --compact --exit-status
```
</details>

---

**Q4.** `gh run watch` fails with a 403 error even though `gh run list` works fine on the same repo. What is the likely cause?

<details><summary>Answer</summary>

The token is a **fine-grained PAT**. Fine-grained PATs cannot be granted `checks:read` permission, which `watch` requires. Switch to a classic token (or use a `GH_TOKEN` set to a classic token value).
</details>

---

**Q5.** You trigger a workflow via `gh workflow run` and immediately call `gh run watch`. It fails with "run not found". Why, and how do you fix it?

<details><summary>Answer</summary>

The run has not appeared in the API yet — it typically takes 2–5 seconds after dispatch. Fix: add `sleep 5` (or poll with `gh run list --limit 1`) before calling `watch`.

```bash
gh workflow run my-workflow --ref main
sleep 5
RUN_ID=$(gh run list --limit 1 --json databaseId --jq '.[0].databaseId')
gh run watch "$RUN_ID" --exit-status
```
</details>

---

**Q6.** What is the default polling interval for `gh run watch`, and how would you change it to poll every 10 seconds?

<details><summary>Answer</summary>

Default: **3 seconds**. To change it:

```bash
gh run watch "$RUN_ID" --interval 10
```

Use a higher interval (`--interval 10` or more) in automated scripts to reduce API call rate on long-running workflows.
</details>
