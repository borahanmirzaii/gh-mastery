# Recall — `gh secret`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** You run `gh secret set DATABASE_URL --body "postgres://..."` and the command succeeds. Your Actions workflow still fails with `DATABASE_URL is not set`. Name the two most likely scope/app mismatches and how to diagnose each.

<details><summary>Answer</summary>

Two common mismatches:

1. **Wrong app store.** The default is `--app actions`. If you accidentally set it for `--app dependabot` or `--app codespaces`, Actions workflows cannot read it. Diagnose: run `gh secret list --repo <repo>` (Actions) and `gh secret list --app dependabot --repo <repo>` (Dependabot) and compare where the name shows up.

2. **Wrong scope.** If you set an **environment secret** (`--env staging`) but the workflow step does not run in the `staging` deployment environment context, the secret is invisible. Diagnose: `gh secret list --env staging --repo <repo>` to confirm it exists there, then check the workflow YAML's `environment:` key matches the environment name exactly.

Rule of thumb: always run `gh secret list` in the relevant scope immediately after setting.
</details>

---

**Q2.** What are the three top-level scopes for `gh secret set`, and which flag enables each?

<details><summary>Answer</summary>

| Scope | Flag | Who reads it |
|---|---|---|
| Repository (default) | _(none — it's the default)_ | Actions runs, Agents sessions, Dependabot in this repo |
| Deployment environment | `--env <name>` | Actions runs for that environment in this repo |
| Organization | `--org <name>` | Actions, Agents, Dependabot, Codespaces across org repos |
| User | `--user` | Codespaces for your personal account |

Omitting all three scope flags defaults to **repository scope**.
</details>

---

**Q3.** You need to set the same `NPM_TOKEN` for Dependabot across 20 repos. What single command achieves this?

<details><summary>Answer</summary>

```bash
gh secret set NPM_TOKEN \
  --org myorg \
  --app dependabot \
  --visibility all \
  --body "$NPM_TOKEN_VALUE"
```

`--org` sets the secret at org level, `--app dependabot` targets the Dependabot store, and `--visibility all` makes it accessible to every repo in the org. Replace `--visibility all` with `--repos repo1,repo2` to restrict to specific repos.
</details>

---

**Q4.** You've forgotten the value of a secret you set last month. How do you recover it via `gh secret list`?

<details><summary>Answer</summary>

You cannot. `gh secret list` shows **names and `updatedAt` timestamps only** — values are never returned by any CLI command or the GitHub UI once stored. GitHub's secret storage is write-only from the retrieval perspective (the value is encrypted with the repo's public key before transmission; GitHub stores the ciphertext only).

**Forgotten secret = rotate, not recover.** Generate a new credential from the issuing service and overwrite with `gh secret set NAME --body "new-value"`.
</details>

---

**Q5.** What does `--app` control, and what are the four valid values?

<details><summary>Answer</summary>

`--app` selects **which secret store** to target within a given scope. The four valid values are:

- `actions` — GitHub Actions (default if `--app` is omitted)
- `agents` — GitHub Agents (AI agent sessions)
- `codespaces` — GitHub Codespaces
- `dependabot` — Dependabot auto-updates

Each is an entirely separate store. Setting `--app codespaces` when you meant `--app actions` means the secret is stored in the Codespaces store and is invisible to Actions workflows — no error is raised.
</details>

---

**Q6.** What is the difference between `gh secret set NAME` (no `--body`) and `gh secret set NAME --body "value"`?

<details><summary>Answer</summary>

- `gh secret set NAME` (no `--body`) — opens an **interactive hidden prompt** where you type or paste the value. The value is not echoed and does not appear in your terminal scrollback or shell history. Best practice for setting real credentials manually.

- `gh secret set NAME --body "value"` — passes the value as a flag argument. Convenient for scripting, but the literal value may appear in your **shell history** (`.zsh_history`, `.bash_history`) if you type it directly. Safer alternative: `--body "$ENV_VAR"` (expands before the shell records the command) or `--env-file .env` for bulk loading.
</details>

---

**Q7.** You try `gh secret set DB_URL --env staging --body "..."` and get an error. What is the most likely cause?

<details><summary>Answer</summary>

The deployment environment `staging` does not exist in the repository. Environment secrets are scoped to a named **deployment environment**, and the environment must be created before you can add secrets to it.

Fix: create the environment first:
```bash
gh api repos/OWNER/REPO/environments/staging -X PUT --silent
```
Then re-run `gh secret set`. Alternatively, create the environment via the GitHub web UI under **Settings → Environments**.
</details>

---

**Q8.** You run `gh secret list --repo myorg/myrepo` and get zero results, but you know secrets exist. What `--app` values should you try next?

<details><summary>Answer</summary>

`gh secret list` without `--app` defaults to the **Actions** store. If secrets were set for Dependabot, Codespaces, or Agents, they live in separate stores and won't appear in the default listing.

Try:
```bash
gh secret list --app dependabot --repo myorg/myrepo
gh secret list --app codespaces --repo myorg/myrepo
gh secret list --app agents     --repo myorg/myrepo
```

Also check environment secrets:
```bash
gh secret list --env <env-name> --repo myorg/myrepo
```

Each store must be queried independently.
</details>
