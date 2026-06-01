# `gh variable`

> **One-liner:** Manage GitHub Actions and Dependabot variables — non-encrypted key/value pairs — at the repository, environment, or organization level.

## When you reach for it

Reach for `gh variable` when you need to store **non-sensitive configuration** that GitHub Actions or Dependabot workflows read at runtime: AWS region, feature flags, deployment targets, image registries, timeout values. Because the values are not encrypted, they are readable by anyone with repo access — that visibility is a feature (auditability) and a constraint (no passwords).

In the solo-builder loop this typically appears at repo setup time: before you write your first workflow, you stash the environment-specific knobs here so the YAML stays clean and environment-agnostic.

## Subcommands

All four subcommands are shallow (no promoted nodes); everything is documented inline below.

| Subcommand | Purpose | Node? |
|---|---|---|
| `gh variable set <NAME>` | Create or update a variable | inline |
| `gh variable get <NAME>` | Print a variable's current value | inline |
| `gh variable list` | List all variables (with their **visible** values) | inline |
| `gh variable delete <NAME>` | Delete a variable | inline |

### `gh variable set`

Create or update a variable at repo, environment, or org scope.

```
gh variable set <variable-name> [flags]

  -b, --body string          The value (reads from stdin if omitted)
  -e, --env environment      Target a deployment environment
  -f, --env-file file        Bulk-load from a dotenv-formatted file
  -o, --org organization     Target an organisation
  -r, --repos repositories   Restrict an org variable to specific repos
  -v, --visibility string    Org variable visibility: all|private|selected (default "private")
  -R, --repo OWNER/REPO      Target a different repo
```

### `gh variable get`

Print the value of a single named variable. Unlike `gh secret get` (which does not exist — secrets are write-only), `gh variable get` returns the **plaintext value** — confirming that variables are never encrypted.

```
gh variable get <variable-name> [flags]

  -e, --env string    Read from a deployment environment
  -o, --org string    Read from an organisation
  --json fields       Output JSON (fields: name, value, visibility, createdAt, updatedAt, ...)
  -q, --jq expr       Filter JSON with a jq expression
```

### `gh variable list`

List all variables at a given scope. Values appear in plain text in the output — there is no masking.

```
gh variable list [flags]  (alias: gh variable ls)

  -e, --env string    List for a deployment environment
  -o, --org string    List for an organisation
  --json fields       JSON output
  -q, --jq expr       jq filter
```

### `gh variable delete`

Delete a named variable.

```
gh variable delete <variable-name> [flags]  (alias: gh variable remove)

  -e, --env string    Delete from a deployment environment
  -o, --org string    Delete from an organisation
```

## Key flags

- `--body` / `-b` — Supply the value inline. Omit it to read from stdin (useful in scripts: `echo "$MY_VAL" | gh variable set NAME`).
- `--env` / `-e` — Scope the variable to a deployment environment. The environment must already exist in the repo. Same flag name across `set`, `get`, `list`, `delete`.
- `--org` / `-o` — Scope to an organization. Requires the caller to be an org admin. Combine with `--repos` to restrict which repos can read the variable, and `--visibility` to control whether all repos or only private ones can see it.
- `--env-file` / `-f` — Bulk-import from a `.env` file; each `KEY=VALUE` line becomes a separate variable. The fastest way to hydrate a fresh repo.
- `--json` + `--jq` / `-q` — Machine-readable output from `list` and `get`; pipe into `jq` for scripted checks.

## Examples

```bash
# 1. Set a repo-level variable (value inline)
gh variable set AWS_REGION --body "us-east-1" --repo borahanmirzaii/gh-mastery-sandbox

# 2. Read it back — value is plaintext, no masking
gh variable get AWS_REGION --repo borahanmirzaii/gh-mastery-sandbox
# Output: us-east-1   (no asterisks, no masking)

# 3. List all repo variables — values appear in the table
gh variable list --repo borahanmirzaii/gh-mastery-sandbox

# 4. Set an environment-scoped variable (environment must exist first)
gh api -X PUT repos/borahanmirzaii/gh-mastery-sandbox/environments/staging  # create env
gh variable set DEPLOY_TARGET --body "staging-cluster" \
  --env staging --repo borahanmirzaii/gh-mastery-sandbox

# 5. List only variables for the staging environment
gh variable list --env staging --repo borahanmirzaii/gh-mastery-sandbox

# 6. Get a single env-scoped variable value in a script
VAL=$(gh variable get DEPLOY_TARGET --env staging \
      --repo borahanmirzaii/gh-mastery-sandbox)
echo "Deploy target is: $VAL"

# 7. Bulk-load from a .env file (names must be alphanumeric + underscore only)
cat <<'EOF' > /tmp/build.env
CACHE_TTL=3600
LOG_LEVEL=info
RETRY_COUNT=3
EOF
gh variable set -f /tmp/build.env --repo borahanmirzaii/gh-mastery-sandbox

# 8. Delete a variable
gh variable delete AWS_REGION --repo borahanmirzaii/gh-mastery-sandbox
```

## Gotchas

- **Variable names are restricted to `[A-Za-z0-9_]` and must start with a letter or underscore — no hyphens.** Attempting to create `my-var` fails with HTTP 422. Use `MY_VAR` or `my_var` instead. This catches people who try to follow the `zz-<group>-*` drill namespace literally — for variables, the namespace must use underscores: `ZZ_VARIABLE_*`.

- **Variables are NOT encrypted — values are fully visible.** `gh variable list` and `gh variable get` print plaintext values. Anyone with read access to the repo can see them via the GitHub UI, the API, or these CLI commands. Never put tokens, passwords, private keys, or any credential in a variable. Use `gh secret set` (→ [`commands/secret/`](../secret/)) for anything sensitive.

- **`gh variable` vs `gh secret` — same shape, different trust model.** Both have `set`/`list`/`delete`/`get` and support `--repo`/`--env`/`--org` scoping. The critical difference: secrets are encrypted at rest, their values are never returned by `gh secret list` or any API, and there is no `gh secret get`. Variables are transparent — the `get` subcommand exists precisely because visibility is expected. Choose by asking: "would I be comfortable with this value showing up in plain text in a `gh variable list` printout?"

- **Environment must exist before you can set env-scoped variables.** If you run `gh variable set FOO --env staging` and the `staging` environment does not exist, the call fails. Create the environment first: `gh api -X PUT repos/OWNER/REPO/environments/staging` (no body required for a basic environment).

- **Org variables require admin access and explicit visibility.** `--visibility` defaults to `"private"` (only private repos in the org). Set `--visibility all` to include public repos, or `--visibility selected` combined with `--repos repo1,repo2` for surgical (= precise, targeted) scoping.

- **`--env-file` does not support comments or quoted values.** The parser is strict dotenv: `KEY=VALUE` only. Lines starting with `#` and lines with `export KEY=VALUE` are silently skipped or may error — test your file first with `cat .env` before bulk-importing.

- **`list` has an alias `ls`; `delete` has an alias `remove`.** Both are baked into the CLI — no configuration needed.

- **Variables are available to both GitHub Actions and Dependabot** at the repo level, unlike some secret apps (e.g., `--app codespaces`). Org and environment scoping works identically to secrets.

## Concepts

None. (Variables are a straightforward key/value store; the relevant concept is the Actions environment model, covered when the `run` and `workflow` groups are authored.)

## Sources

- Manual: https://cli.github.com/manual/gh_variable
- Local: `gh variable --help` (gh 2.92.0)
- Encrypted counterpart: [`commands/secret/`](../secret/)
