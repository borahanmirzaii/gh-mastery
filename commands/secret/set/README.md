# `gh secret set`

> **One-liner:** Create or update an encrypted secret for GitHub Actions, Agents, Codespaces, or Dependabot — at repo, environment, org, or user scope — from a prompt, flag, or dotenv file.

## When you reach for it

`gh secret set` is the write half of secret management. You reach for it:
- After creating a new repo, before writing any workflow that needs a token.
- When rotating a credential — overwrite the existing name; GitHub replaces the ciphertext atomically.
- In a setup script that bootstraps a repo's entire secret set from a `.env` file using `--env-file`.
- When promoting a secret from repo-level to org-level to share it across repos.

The counterpart `gh secret list` (read names, no values) and `gh secret delete` complete the lifecycle, but `set` is where you start.

## Key flags

- `-b, --body string` — provide the secret value as a CLI argument. Avoids the interactive prompt; useful in scripts. **Caution:** the value may appear in shell history if typed literally — prefer `--body "$ENV_VAR"` or `--env-file`.
- `-f, --env-file file` — load multiple secrets at once from a dotenv-formatted file (`KEY=value` lines; lines starting with `#` are ignored). Pass `-f -` to read from stdin. Bulk-set all secrets for a service in one invocation.
- `-e, --env string` — scope the secret to a **deployment environment** within the repo. The environment must exist before this call.
- `-o, --org string` — scope the secret to an **organization**. Requires org admin permissions.
- `-u, --user` — scope the secret to your **user account** (Codespaces only).
- `-a, --app string` — target a specific secret store: `actions` (default), `agents`, `codespaces`, or `dependabot`. **If omitted, the default is `actions`.** Setting the wrong app silently lands the secret in the wrong store.
- `-r, --repos list` — for org or user secrets: comma-separated list of repos that may access the secret. Sets visibility to `selected` automatically.
- `-v, --visibility string` — for org secrets: `all` (every repo), `private` (private repos only, default), or `selected` (only repos in `--repos`). No effect on repo-level secrets.
- `--no-repos-selected` — for org secrets: no repository can access the secret (useful to create a placeholder before assigning repos).
- `--no-store` — print the encrypted, base64-encoded value to stdout **without** storing it on GitHub. Useful for generating the encrypted payload to pass to the REST API directly.
- `-R, --repo [HOST/]OWNER/REPO` — target a different repo than the current directory infers.

## Examples

```bash
# Interactive prompt (hidden input — value not echoed, best for real credentials)
gh secret set DEPLOY_KEY --repo borahanmirzaii/gh-mastery-sandbox

# Non-interactive with --body (useful in CI scripts; avoid pasting real tokens literally)
gh secret set API_TOKEN --body "throwaway-value" --repo borahanmirzaii/gh-mastery-sandbox

# Set from an env var (the value never appears as a literal in history)
gh secret set API_TOKEN --body "$MY_API_TOKEN" --repo borahanmirzaii/gh-mastery-sandbox

# Pipe from stdin (another history-safe approach)
echo "throwaway-value" | gh secret set API_TOKEN --repo borahanmirzaii/gh-mastery-sandbox

# Load all secrets from a .env file (bulk set)
gh secret set -f .env --repo borahanmirzaii/gh-mastery-sandbox

# Load secrets from stdin in dotenv format
printf 'DB_URL=postgres://host/db\nREDIS_URL=redis://host:6379\n' | gh secret set -f - --repo borahanmirzaii/gh-mastery-sandbox

# Set for Dependabot (not Actions — explicit --app required)
gh secret set NPM_TOKEN --app dependabot --body "npm_token_value" --repo borahanmirzaii/gh-mastery-sandbox

# Set an environment secret for the "staging" deployment environment
gh secret set DB_URL --env staging --body "postgres://staging-host/db" --repo borahanmirzaii/gh-mastery-sandbox

# Set an org secret visible to all repos
gh secret set SHARED_SLACK_TOKEN --org myorg --visibility all --body "xoxb-..."

# Set an org secret restricted to two repos
gh secret set DEPLOY_KEY --org myorg --repos repo1,repo2 --body "ssh-rsa ..."

# Rotate a secret (same command, new value — atomic overwrite)
gh secret set API_TOKEN --body "new-rotated-value" --repo borahanmirzaii/gh-mastery-sandbox
```

## Gotchas

- **Three scopes: repo (default), org, and env.** `--org <name>` targets org-level; `--env <name>` targets a deployment environment; omitting both defaults to the current repo. Choosing the wrong scope silently sets the secret in the wrong place — the call succeeds, but the workflow finds nothing.

- **`--app` selects which secret store within a scope.** Default is `actions`. Setting `--app codespaces` when you meant `--app actions` (or vice versa) results in a silent no-op for the consumer — no error is raised by the CLI or GitHub.

- **Secret values are never echoed or retrievable after set.** `gh secret list` shows names and `updatedAt` only. Once a secret is set, its value cannot be read back by any CLI command or the API. Forgotten secret = rotate, not recover.

- **`--body` and shell history.** `gh secret set NAME --body "actual-token"` records the literal token in shell history. Use `--body "$ENV_VAR"`, the interactive prompt, or `--env-file` for real credentials.

- **`--env-file` is dotenv format, not shell syntax.** Lines must be `KEY=value` with no `export`, no quotes around the value (unless quotes are part of the value), and no variable substitution. Comments (`# ...`) are stripped.

- **Environment must exist before `--env`.** `gh secret set NAME --env missing-env` fails. Create the environment first: `gh api repos/OWNER/REPO/environments/missing-env -X PUT`.

- **Org secrets need org admin or secrets manager role.** A regular member gets a 403 with a generic permission error.

- **`-r, --repos` and `--visibility selected` are two sides of the same coin.** Passing `--repos repo1,repo2` automatically sets visibility to `selected`. If you set `--visibility selected` without `--repos`, the result is the same as `--no-repos-selected` — no repos can access the secret.

## Concepts

- None. (GitHub Actions deployment environments are covered in [`../../concepts/actions-model.md`](../../concepts/actions-model.md) — _(to be written)_.)

## Sources

- Manual: https://cli.github.com/manual/gh_secret_set
- Local: `gh secret set --help` (gh 2.92.0)
