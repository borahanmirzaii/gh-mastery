# `gh secret`

> **One-liner:** Create, list, and delete encrypted secrets for GitHub Actions, Agents, Codespaces, and Dependabot — scoped to a repo, deployment environment, organization, or your user account.

## When you reach for it

You reach for `gh secret` any time you need to **inject sensitive values into GitHub's execution contexts without exposing them in YAML or logs**: API tokens for CI workflows, cloud credentials for deployment environments, npm auth tokens for Dependabot auto-updates, Codespaces personal tokens, and so on. In the solo-builder loop this typically happens right after scaffolding a new repo or before writing a workflow that needs a token — you set the secret once via the CLI, reference it as `${{ secrets.NAME }}` in Actions, and it never touches plain text again.

Concrete moments:
- You've written a release workflow that calls the PyPI API; before merging you run `gh secret set PYPI_TOKEN --body "..."` to drop the token into the repo's Actions store.
- A staging deployment environment needs its own `DB_URL` separate from production — `gh secret set DB_URL --env staging --body "..."` scopes it to that environment only.
- Your org uses a shared NPM_TOKEN for all private packages; `gh secret set NPM_TOKEN --org myorg --visibility all --body "..."` makes it available across every repo without per-repo repetition.

## Subcommands

| Subcommand | Purpose | Node? |
|---|---|---|
| `gh secret set` | Create or update an encrypted secret (repo, env, org, or user scope) | [→ `set/`](./set/) |
| `gh secret list` | List secret names and last-updated timestamps (values are never shown) | inline |
| `gh secret delete` | Delete a secret by name from the chosen scope | inline |

## Key flags

All subcommands share:
- `-R, --repo [HOST/]OWNER/REPO` — target a specific repo instead of the one inferred from the current directory.

`gh secret set` (see also [`set/`](./set/) for full coverage):
- `-b, --body string` — provide the secret value inline (avoid shell history exposure — prefer stdin or `--env-file` for real credentials).
- `-e, --env string` — set an **environment-level** secret (scoped to a specific deployment environment, Actions only).
- `-o, --org string` — set an **org-level** secret (available to multiple repos in the org).
- `-u, --user` — set a **user-level** secret (Codespaces for your user account).
- `-a, --app string` — choose the secret store: `actions` (default), `agents`, `codespaces`, or `dependabot`. **Choosing the wrong app silently sets the secret in the wrong store.**
- `-f, --env-file file` — load multiple secrets from a dotenv file in one command.
- `-r, --repos list` — restrict an org or user secret to specific repositories.
- `-v, --visibility string` — for org secrets: `all`, `private`, or `selected` (default `private`).

`gh secret list`:
- `-a, --app string` — filter the listing by app store (`actions`, `agents`, `codespaces`, `dependabot`).
- `-e, --env string` — list secrets for a specific deployment environment.
- `-o, --org string` — list secrets for an organization.
- `-u, --user` — list your user Codespaces secrets.
- `--json fields` — machine-readable output with fields `name`, `updatedAt`, `visibility`, `numSelectedRepos`, `selectedReposURL`.

`gh secret delete`:
- `-a, --app string` — delete from a specific app store.
- `-e, --env string` — delete an environment secret.
- `-o, --org string` — delete an org secret.
- `-u, --user` — delete a user Codespaces secret.

## Examples

```bash
# Set a repo secret from stdin (interactive prompt — value is hidden)
gh secret set API_TOKEN --repo borahanmirzaii/gh-mastery-sandbox

# Set a repo secret non-interactively with --body (for scripting; be mindful of shell history)
gh secret set API_TOKEN --body "my-token-value" --repo borahanmirzaii/gh-mastery-sandbox

# Set a secret for the Dependabot app instead of Actions
gh secret set NPM_TOKEN --app dependabot --body "npm_token_value" --repo borahanmirzaii/gh-mastery-sandbox

# Set an environment-level secret (scoped to the "staging" deployment environment)
gh secret set DB_URL --env staging --body "postgres://staging-host/db" --repo borahanmirzaii/gh-mastery-sandbox

# Set an org-level secret visible to all repos in the org
gh secret set SHARED_TOKEN --org myorg --visibility all --body "shared-value"

# Set an org-level secret restricted to specific repos
gh secret set SHARED_TOKEN --org myorg --repos repo1,repo2,repo3 --body "shared-value"

# Load multiple secrets from a .env file at once
gh secret set -f .env --repo borahanmirzaii/gh-mastery-sandbox

# List all Actions secrets for a repo
gh secret list --repo borahanmirzaii/gh-mastery-sandbox

# List Dependabot secrets specifically
gh secret list --app dependabot --repo borahanmirzaii/gh-mastery-sandbox

# List secrets for a deployment environment
gh secret list --env staging --repo borahanmirzaii/gh-mastery-sandbox

# Delete a repo secret
gh secret delete OLD_TOKEN --repo borahanmirzaii/gh-mastery-sandbox
```

## Gotchas

- **Three scopes: repo (default), org, and env.** Using `--org <name>` targets org-level secrets; `--env <name>` targets a deployment environment within the current repo; omitting both defaults to the current repo. **Choosing the wrong scope silently sets the secret in the wrong place** — the command succeeds, the secret exists, but the workflow that needs it finds nothing.

- **`--app` selects which secret store within a scope.** The default is `actions`. If you mean to set a Dependabot secret and omit `--app dependabot`, the secret ends up in the Actions store where Dependabot cannot read it — no error, no warning. The four app targets are: `actions`, `agents`, `codespaces`, `dependabot`.

- **Secret values are never echoed or retrievable.** `gh secret list` shows only names and `updatedAt` timestamps — no values. `gh secret set` reads from a hidden prompt or `--body`; once stored, the value cannot be read back via any CLI command or the GitHub UI. **Forgotten secret = rotate, not recover.** Design your rotation procedure before you need it.

- **`--body` and shell history.** Passing a real token as `--body "actual-secret"` may appear in your shell history (`.bash_history`, `.zsh_history`). For real credentials prefer the interactive prompt (`gh secret set NAME` with no `--body`) or load from an env var: `--body "$MY_TOKEN"` (the variable expands before the shell records the command, but the token lands in the process table briefly). The safest option for automation is `--env-file` pointing to a gitignored `.env`.

- **Environment secrets require the environment to exist first.** `gh secret set NAME --env myenv` fails if `myenv` doesn't exist as a deployment environment in the repo. Create it first via the GitHub web UI or `gh api repos/OWNER/REPO/environments/myenv -X PUT`.

- **Org secrets need org admin rights.** `gh secret set NAME --org myorg` requires you to be an organization owner or have the `admin:org_hook` + secrets management permissions. A regular member gets a 403 with no explanation.

- **`--repos` and `--visibility selected` are linked.** To restrict an org secret to specific repos, pass `-r repo1,repo2` — this automatically sets visibility to `selected`. Passing `--visibility selected` without `--repos` means no repos can access the secret (same as `--no-repos-selected`). These two flags encode the same state from different angles.

- **gh secret list output is paginated for orgs with many secrets** — add `--json name,updatedAt | jq 'length'` to count them or pipe to a pager.

## Concepts

- None. (GitHub Actions deployment environments are documented in the [Actions model concept](../../concepts/actions-model.md) — _(to be written)_.)

## Sources

- Manual: https://cli.github.com/manual/gh_secret
- Manual (set): https://cli.github.com/manual/gh_secret_set
- Local: `gh secret --help` (gh 2.92.0)
- Local: `gh secret set --help` (gh 2.92.0)
