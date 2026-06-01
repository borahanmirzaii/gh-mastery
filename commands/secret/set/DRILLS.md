# Drills — `gh secret set`

> **Sandbox:** run everything against `borahanmirzaii/gh-mastery-sandbox`
> (add `--repo borahanmirzaii/gh-mastery-sandbox` to every command or set `export REPO=borahanmirzaii/gh-mastery-sandbox`).
> **Namespacing:** create only secrets prefixed `ZZ_SECRET_` and delete them at the end.

---

## Drill 1 — Set and verify a simple repo secret

**Goal:** Set `ZZ_SECRET_HELLO` with value `drill-value-1` in the sandbox using `--body`, then confirm it appears in `gh secret list`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO=borahanmirzaii/gh-mastery-sandbox

gh secret set ZZ_SECRET_HELLO \
  --body "drill-value-1" \
  --repo $REPO

gh secret list --repo $REPO
```

</details>

**Verify:** `ZZ_SECRET_HELLO` appears in the list. No value column is shown — only the name and last-updated time.

---

## Drill 2 — Set from a dotenv file

**Goal:** Create a temporary `.env` file with two entries (`ZZ_SECRET_ALPHA` and `ZZ_SECRET_BETA`), bulk-load them with `--env-file`, and verify both appear.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO=borahanmirzaii/gh-mastery-sandbox

# Write a throwaway dotenv file
cat > /tmp/zz-secret-drill.env << 'EOF'
ZZ_SECRET_ALPHA=drill-alpha-value
ZZ_SECRET_BETA=drill-beta-value
EOF

# Bulk-load from the file
gh secret set -f /tmp/zz-secret-drill.env --repo $REPO

# Verify
gh secret list --repo $REPO

# Clean up the local file
rm /tmp/zz-secret-drill.env
```

</details>

**Verify:** Both `ZZ_SECRET_ALPHA` and `ZZ_SECRET_BETA` appear in `gh secret list`. The dotenv file is deleted locally — it was only a temporary fixture.

---

## Drill 3 — Set a Dependabot secret and confirm store isolation

**Goal:** Set `ZZ_SECRET_DEP_TOKEN` with `--app dependabot`, then verify it does **not** appear in the default `gh secret list` (Actions store) but **does** appear in `gh secret list --app dependabot`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO=borahanmirzaii/gh-mastery-sandbox

gh secret set ZZ_SECRET_DEP_TOKEN \
  --app dependabot \
  --body "drill-dep-token-value" \
  --repo $REPO

echo "=== Actions store (should NOT contain ZZ_SECRET_DEP_TOKEN) ==="
gh secret list --repo $REPO

echo "=== Dependabot store (SHOULD contain ZZ_SECRET_DEP_TOKEN) ==="
gh secret list --app dependabot --repo $REPO
```

</details>

**Verify:** `ZZ_SECRET_DEP_TOKEN` is absent from the Actions listing and present in the Dependabot listing. This is the `--app` store isolation in action.

---

## Drill 4 — Set an environment secret

**Goal:** Create the `zz-secret-env-drill` deployment environment in the sandbox, set `ZZ_SECRET_ENV_DB` scoped to it, and confirm it appears only under environment listing.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO=borahanmirzaii/gh-mastery-sandbox
ENV_NAME=zz-secret-env-drill

# Create the deployment environment (required first)
gh api "repos/$REPO/environments/$ENV_NAME" -X PUT --silent

# Set the environment-scoped secret
gh secret set ZZ_SECRET_ENV_DB \
  --env $ENV_NAME \
  --body "drill-env-db-value" \
  --repo $REPO

# Verify: env listing shows it
echo "=== Env secrets for $ENV_NAME ==="
gh secret list --env $ENV_NAME --repo $REPO

# Verify: repo-level Actions listing does NOT show it
echo "=== Repo Actions secrets (should NOT contain ZZ_SECRET_ENV_DB) ==="
gh secret list --repo $REPO
```

</details>

**Verify:** `ZZ_SECRET_ENV_DB` appears under `--env zz-secret-env-drill` and is absent from the plain repo listing. Environment secrets are scoped to their environment context in Actions workflows.

---

## Boss drill — Chain: Actions secret + env secret + list filter + delete all

**Goal:** Wire up a simulated staged deployment scenario: set one repo-level Actions secret (CI token), one environment secret (staging DB URL), verify each is in the right store using appropriate `--app` and `--env` filters, then delete all created objects.

> **Key insight:** Each call to `gh secret set` requires you to be explicit about scope (`--env`, `--org`) and app (`--app`). Without those flags, everything defaults to the repo-level Actions store. Verify immediately after each `set` — there's no value to read back later.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO=borahanmirzaii/gh-mastery-sandbox
ENV_NAME=zz-secret-staging-boss

# 1. Set a repo-level Actions secret
gh secret set ZZ_SECRET_CI_TOKEN \
  --app actions \
  --body "boss-drill-ci-token" \
  --repo $REPO

# 2. Create environment, then set env-scoped secret
gh api "repos/$REPO/environments/$ENV_NAME" -X PUT --silent

gh secret set ZZ_SECRET_STAGING_DB \
  --env $ENV_NAME \
  --body "boss-drill-staging-db" \
  --repo $REPO

# 3. Verify each store in isolation
echo "=== Repo Actions secrets (expect ZZ_SECRET_CI_TOKEN) ==="
gh secret list --app actions --repo $REPO

echo "=== Staging env secrets (expect ZZ_SECRET_STAGING_DB) ==="
gh secret list --env $ENV_NAME --repo $REPO

echo "=== Dependabot secrets (expect empty) ==="
gh secret list --app dependabot --repo $REPO

# 4. Cleanup
gh secret delete ZZ_SECRET_CI_TOKEN --repo $REPO
gh secret delete ZZ_SECRET_STAGING_DB --env $ENV_NAME --repo $REPO
gh api "repos/$REPO/environments/$ENV_NAME" -X DELETE --silent

# Also clean up any leftovers from earlier drills
gh secret delete ZZ_SECRET_HELLO   --repo $REPO 2>/dev/null || true
gh secret delete ZZ_SECRET_ALPHA   --repo $REPO 2>/dev/null || true
gh secret delete ZZ_SECRET_BETA    --repo $REPO 2>/dev/null || true
gh secret delete ZZ_SECRET_DEP_TOKEN --app dependabot --repo $REPO 2>/dev/null || true
gh secret delete ZZ_SECRET_ENV_DB  --env zz-secret-env-drill --repo $REPO 2>/dev/null || true
gh api "repos/$REPO/environments/zz-secret-env-drill" -X DELETE --silent 2>/dev/null || true

echo "=== Final state (all should be empty) ==="
gh secret list --repo $REPO
gh secret list --app dependabot --repo $REPO
```

</details>

**Verify:** After cleanup:
- `gh secret list --repo borahanmirzaii/gh-mastery-sandbox` → empty.
- `gh secret list --app dependabot --repo borahanmirzaii/gh-mastery-sandbox` → empty.
- `gh api repos/borahanmirzaii/gh-mastery-sandbox/environments --jq '.total_count'` → `0`.

**Cleanup checklist:**
- [ ] `ZZ_SECRET_CI_TOKEN` deleted (Actions store).
- [ ] `ZZ_SECRET_STAGING_DB` deleted (env `zz-secret-staging-boss`).
- [ ] `ZZ_SECRET_HELLO`, `ZZ_SECRET_ALPHA`, `ZZ_SECRET_BETA` deleted (Actions store).
- [ ] `ZZ_SECRET_DEP_TOKEN` deleted (Dependabot store).
- [ ] `ZZ_SECRET_ENV_DB` deleted (env `zz-secret-env-drill`).
- [ ] Both environments deleted (`zz-secret-staging-boss`, `zz-secret-env-drill`).
- [ ] No `ZZ_SECRET_*` entries remain in any store.
