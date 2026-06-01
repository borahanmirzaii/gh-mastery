# Recall — `gh secret set`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** You run `gh secret set DEPLOY_TOKEN --body "abc123"` in a repo directory. Which app store and which scope does the secret land in?

<details><summary>Answer</summary>

- **App store:** `actions` (the default when `--app` is omitted).
- **Scope:** repository-level (the default when `--env`, `--org`, and `--user` are all omitted).

The full equivalent command is:
```bash
gh secret set DEPLOY_TOKEN --app actions --body "abc123" --repo OWNER/REPO
```

</details>

---

**Q2.** How do you set multiple secrets from a `.env` file in a single `gh` invocation?

<details><summary>Answer</summary>

```bash
gh secret set -f .env --repo OWNER/REPO
```

`-f` / `--env-file` reads a dotenv-formatted file (`KEY=value` lines). Pass `-f -` to read from stdin instead of a file. Notes: no `export` keyword, no shell variable expansion, `#` lines are treated as comments.
</details>

---

**Q3.** What happens if you run `gh secret set DB_URL --env staging` but the `staging` deployment environment does not yet exist?

<details><summary>Answer</summary>

The command fails — GitHub returns an error because the environment must exist before secrets can be scoped to it.

Fix: create the environment first:
```bash
gh api repos/OWNER/REPO/environments/staging -X PUT --silent
```
Then re-run `gh secret set DB_URL --env staging --body "..."`.
</details>

---

**Q4.** What is the safest way to pass a real token to `gh secret set` without it appearing in your shell history?

<details><summary>Answer</summary>

Three options, safest first:

1. **Interactive prompt** — run `gh secret set NAME` with no `--body`. The CLI prompts for the value with hidden input (not echoed, not recorded in history).
2. **Environment variable** — `gh secret set NAME --body "$MY_TOKEN"`. The shell expands `$MY_TOKEN` before recording the command, so the literal token doesn't appear in history (though it does appear briefly in the process table).
3. **`--env-file`** — store secrets in a gitignored `.env` file and use `gh secret set -f .env`. The file path (not the values) appears in history.

Avoid: `gh secret set NAME --body "actual-token-here"` — this records the literal token in `.bash_history` / `.zsh_history`.
</details>

---

**Q5.** You want to share a `SLACK_WEBHOOK` secret across all private repos in your org, but not public ones. What command achieves this?

<details><summary>Answer</summary>

```bash
gh secret set SLACK_WEBHOOK \
  --org myorg \
  --visibility private \
  --body "$SLACK_WEBHOOK_URL"
```

`--visibility private` (which is the default for org secrets) makes the secret available to all private repos in the org. Use `--visibility all` to include public repos too, or `--repos repo1,repo2` to restrict to specific repos.
</details>

---

**Q6.** What does `--no-store` do, and when would you use it?

<details><summary>Answer</summary>

`--no-store` prints the **encrypted, base64-encoded ciphertext** of the secret value to stdout without storing it on GitHub.

Use case: when you want to pre-encrypt a secret locally and pass the payload to the GitHub REST API directly (e.g., in a script that uses `gh api` or `curl` rather than `gh secret set`). The encrypted value is produced using the repo's public key, so it can only be decrypted by GitHub's secret store for that repo.
</details>

---

**Q7.** You set `ZZ_SECRET_API_TOKEN` two weeks ago. A new workflow needs the same token. Can you read the stored value with `gh secret list` to copy it to another repo?

<details><summary>Answer</summary>

No. `gh secret list` shows **names and `updatedAt` timestamps only** — the value is never returned. There is no CLI command or API endpoint that retrieves a stored secret value; GitHub only ever stores the ciphertext.

To copy the secret to another repo, you must re-obtain the original credential from its source (e.g., the API provider's dashboard) and run `gh secret set` against the target repo. This is by design — it prevents accidental exposure and enforces explicit rotation.
</details>
