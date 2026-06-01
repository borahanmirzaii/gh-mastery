# Drills — `gh variable`

> **Sandbox:** run everything against `borahanmirzaii/gh-mastery-sandbox`
> (pass `--repo borahanmirzaii/gh-mastery-sandbox` to every command).
> **Namespacing:** create only variables prefixed `ZZ_VARIABLE_*` and delete them at the end,
> so parallel drills never collide.
> **Naming constraint:** GitHub variable names are `[A-Za-z0-9_]` only — no hyphens.
> The drill namespace therefore uses underscores: `ZZ_VARIABLE_*` (not `zz-variable-*`).

---

## Drill 1 — Set and retrieve a repo-level variable

**Goal:** Create a repo variable, then read it back with `get` to confirm the value is returned in plaintext (proving it is not encrypted, unlike a secret).

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Create the variable
gh variable set ZZ_VARIABLE_REGION \
  --body "us-east-1" \
  --repo borahanmirzaii/gh-mastery-sandbox

# Read it back — value is plaintext, no masking
gh variable get ZZ_VARIABLE_REGION \
  --repo borahanmirzaii/gh-mastery-sandbox
```

Expected output from `get`: `us-east-1` (no asterisks, no masking — this would be impossible with a secret).

</details>

**Verify:** `gh variable list --repo borahanmirzaii/gh-mastery-sandbox` → table includes `ZZ_VARIABLE_REGION` with value `us-east-1` visible in the `VALUE` column.

---

## Drill 2 — Bulk-load variables from a .env file

**Goal:** Hydrate (= populate from scratch) multiple variables in one shot using `--env-file`, then confirm all appear in `list`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Write a small dotenv file (names must be alphanumeric + underscore only)
cat <<'EOF' > /tmp/zz_variable_bulk.env
ZZ_VARIABLE_LOG_LEVEL=info
ZZ_VARIABLE_RETRY_COUNT=3
ZZ_VARIABLE_CACHE_TTL=3600
EOF

# Bulk-import all at once
gh variable set -f /tmp/zz_variable_bulk.env \
  --repo borahanmirzaii/gh-mastery-sandbox
```

</details>

**Verify:** `gh variable list --repo borahanmirzaii/gh-mastery-sandbox --json name,value --jq '.[] | select(.name | startswith("ZZ_VARIABLE")) | "\(.name)=\(.value)"'` → three lines, one per variable, values visible in plaintext.

---

## Drill 3 — Set and inspect an environment-scoped variable

**Goal:** Create a `ZZ_VARIABLE_STAGING` deployment environment in the sandbox, set a variable scoped to it, then confirm scoping is independent of the repo-level list.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Create the environment first (required before setting env-scoped variables)
gh api -X PUT repos/borahanmirzaii/gh-mastery-sandbox/environments/ZZ_VARIABLE_STAGING

# Set the variable scoped to that environment
gh variable set ZZ_VARIABLE_DEPLOY_TARGET \
  --body "staging-cluster" \
  --env ZZ_VARIABLE_STAGING \
  --repo borahanmirzaii/gh-mastery-sandbox

# Get it back — plaintext, as expected
gh variable get ZZ_VARIABLE_DEPLOY_TARGET \
  --env ZZ_VARIABLE_STAGING \
  --repo borahanmirzaii/gh-mastery-sandbox
```

Expected `get` output: `staging-cluster`

</details>

**Verify (two-part):**
1. `gh variable list --env ZZ_VARIABLE_STAGING --repo borahanmirzaii/gh-mastery-sandbox` → shows `ZZ_VARIABLE_DEPLOY_TARGET`.
2. `gh variable list --repo borahanmirzaii/gh-mastery-sandbox` → does **not** show `ZZ_VARIABLE_DEPLOY_TARGET` (env scope is a separate bucket from repo scope).

---

## Drill 4 — Delete a specific variable

**Goal:** Delete a single variable and confirm it no longer appears in `list`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh variable delete ZZ_VARIABLE_REGION \
  --repo borahanmirzaii/gh-mastery-sandbox
```

</details>

**Verify:** `gh variable list --repo borahanmirzaii/gh-mastery-sandbox --json name --jq '.[].name'` → `ZZ_VARIABLE_REGION` is gone; other variables remain.

---

## Drill 5 — JSON output + jq filtering

**Goal:** Use `--json` and `--jq` to extract variable data in machine-readable form — the pattern you'd use in a shell script to audit variables before a deployment.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# List all repo variables as JSON, extract names only
gh variable list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --json name,value \
  --jq '.[] | .name'

# Get a specific variable as JSON including timestamps
gh variable get ZZ_VARIABLE_CACHE_TTL \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --json name,value,createdAt,updatedAt
```

</details>

**Verify:** First command outputs newline-separated names with no table chrome (= decorative table borders/headers). Second command outputs a JSON object with `value` field visible in plaintext.

---

## Boss drill — Full lifecycle: set (repo) → set (env) → list → get → delete all `ZZ_VARIABLE_*`

**Goal:** Chain set (repo scope) → set (env scope) → list → get one value → delete all `ZZ_VARIABLE_*` objects. Mirrors standing up a new repo's config before the first deploy.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
SANDBOX="borahanmirzaii/gh-mastery-sandbox"

# 1. Set a repo-level variable
gh variable set ZZ_VARIABLE_APP_NAME \
  --body "gh-mastery-demo" \
  --repo "$SANDBOX"

# 2. Ensure the staging environment exists
gh api -X PUT "repos/$SANDBOX/environments/ZZ_VARIABLE_STAGING"

# 3. Set an environment-scoped variable
gh variable set ZZ_VARIABLE_DEPLOY_URL \
  --body "https://staging.example.com" \
  --env ZZ_VARIABLE_STAGING \
  --repo "$SANDBOX"

# 4. List repo-level variables (confirms APP_NAME is there; DEPLOY_URL is NOT)
echo "=== Repo variables ==="
gh variable list --repo "$SANDBOX"

# 5. List env-level variables (confirms DEPLOY_URL is there)
echo "=== Staging env variables ==="
gh variable list --env ZZ_VARIABLE_STAGING --repo "$SANDBOX"

# 6. Get a single value and capture it in a shell variable
APP=$(gh variable get ZZ_VARIABLE_APP_NAME --repo "$SANDBOX")
echo "App name is: $APP"

# 7. Cleanup — delete repo-level ZZ_VARIABLE_* variables
gh variable delete ZZ_VARIABLE_APP_NAME --repo "$SANDBOX"

# 8. Cleanup — delete env-level variables and the environment
gh variable delete ZZ_VARIABLE_DEPLOY_URL --env ZZ_VARIABLE_STAGING --repo "$SANDBOX"
gh api -X DELETE "repos/$SANDBOX/environments/ZZ_VARIABLE_STAGING"
```

</details>

**Verify:** `gh variable list --repo borahanmirzaii/gh-mastery-sandbox` → empty (no `ZZ_VARIABLE_*` rows).

**Cleanup confirmation:** `gh api repos/borahanmirzaii/gh-mastery-sandbox/environments --jq '.environments[].name'` → `ZZ_VARIABLE_STAGING` is gone (command returns empty or no matching line).
