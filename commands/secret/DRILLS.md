# Drills — `gh secret`

> **Sandbox:** run everything against `borahanmirzaii/gh-mastery-sandbox`
> (add `--repo borahanmirzaii/gh-mastery-sandbox` to every command or set `export REPO=borahanmirzaii/gh-mastery-sandbox`).
> **Namespacing:** create only secrets prefixed `ZZ_SECRET_` (GitHub secret names are uppercase by convention) and delete them at the end. No `zz-secret-*` names should linger after cleanup.

---

## Drill 1 — Set a repo secret from stdin

**Goal:** Set a single Actions secret named `ZZ_SECRET_API_TOKEN` in the sandbox using the `--body` flag (non-interactive), then verify it appears in `gh secret list`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO=borahanmirzaii/gh-mastery-sandbox

gh secret set ZZ_SECRET_API_TOKEN \
  --body "drill-throwaway-value-1" \
  --repo $REPO
```

</details>

**Verify:**
```bash
gh secret list --repo borahanmirzaii/gh-mastery-sandbox
```
Expected: `ZZ_SECRET_API_TOKEN` appears in the list with a recent `updatedAt` timestamp. No value column — only the name.

---

## Drill 2 — Set a Dependabot secret and confirm the Actions store is unaffected

**Goal:** Set a secret named `ZZ_SECRET_NPM_TOKEN` for the **Dependabot** app, then list both the Actions secrets and the Dependabot secrets to see that they live in separate stores.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO=borahanmirzaii/gh-mastery-sandbox

gh secret set ZZ_SECRET_NPM_TOKEN \
  --app dependabot \
  --body "drill-throwaway-npm-value" \
  --repo $REPO

# List Actions secrets (should NOT contain ZZ_SECRET_NPM_TOKEN)
echo "=== Actions secrets ==="
gh secret list --repo $REPO

# List Dependabot secrets (SHOULD contain ZZ_SECRET_NPM_TOKEN)
echo "=== Dependabot secrets ==="
gh secret list --app dependabot --repo $REPO
```

</details>

**Verify:** `ZZ_SECRET_NPM_TOKEN` appears under Dependabot but **not** in the default Actions listing. This demonstrates that `--app dependabot` and the default (`actions`) are separate stores — setting one does not affect the other.

---

## Drill 3 — Set an environment-level secret

**Goal:** Create a deployment environment named `zz-secret-staging` in the sandbox, then set a secret `ZZ_SECRET_DB_URL` scoped to that environment. Verify it appears under environment secrets.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO=borahanmirzaii/gh-mastery-sandbox
ENV_NAME=zz-secret-staging

# Step 1: Create the deployment environment (required before setting env secrets)
gh api "repos/$REPO/environments/$ENV_NAME" -X PUT --silent

# Step 2: Set the environment secret
gh secret set ZZ_SECRET_DB_URL \
  --env $ENV_NAME \
  --body "drill-throwaway-db-url" \
  --repo $REPO

# Step 3: List environment secrets
gh secret list --env $ENV_NAME --repo $REPO
```

</details>

**Verify:** `ZZ_SECRET_DB_URL` appears when listing secrets for `zz-secret-staging`. It does **not** appear in the plain `gh secret list` output (which shows repo-level Actions secrets only).

---

## Drill 4 — Delete secrets and confirm cleanup

**Goal:** Delete all `ZZ_SECRET_*` secrets created in Drills 1–3 and verify none remain.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO=borahanmirzaii/gh-mastery-sandbox
ENV_NAME=zz-secret-staging

# Delete repo-level Actions secret (from Drill 1)
gh secret delete ZZ_SECRET_API_TOKEN --repo $REPO

# Delete Dependabot secret (from Drill 2)
gh secret delete ZZ_SECRET_NPM_TOKEN --app dependabot --repo $REPO

# Delete environment secret (from Drill 3)
gh secret delete ZZ_SECRET_DB_URL --env $ENV_NAME --repo $REPO

# Delete the environment itself
gh api "repos/$REPO/environments/$ENV_NAME" -X DELETE --silent

# Verify all gone
echo "=== Actions secrets (expect empty) ==="
gh secret list --repo $REPO
echo "=== Dependabot secrets (expect empty) ==="
gh secret list --app dependabot --repo $REPO
```

</details>

**Verify:** Both listings are empty (no `ZZ_SECRET_*` entries). The `zz-secret-staging` environment no longer exists: `gh api repos/$REPO/environments` should show `"total_count": 0`.

---

## Boss drill — Full secret lifecycle: Actions + env + list filter + delete

**Goal:** Simulate the real workflow of wiring up a repo for a staged deployment: set an Actions secret (CI token), set an environment secret (staging DB URL), list each store to confirm placement, then clean everything up.

> **Key insight:** Each scope (repo Actions, environment, Dependabot) is a **separate store**. Setting in the wrong scope — or the wrong `--app` — is silent. Always verify with `gh secret list` immediately after setting.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REPO=borahanmirzaii/gh-mastery-sandbox
ENV_NAME=zz-secret-staging

# 1. Set a repo-level Actions secret (CI token available to all workflows)
gh secret set ZZ_SECRET_CI_TOKEN \
  --app actions \
  --body "drill-throwaway-ci-token" \
  --repo $REPO

# 2. Create the staging environment, then set an environment-scoped secret
gh api "repos/$REPO/environments/$ENV_NAME" -X PUT --silent
gh secret set ZZ_SECRET_STAGING_DB \
  --env $ENV_NAME \
  --body "drill-throwaway-staging-db" \
  --repo $REPO

# 3. Verify placement: each scope shows only its own secrets
echo "=== Repo Actions secrets (expect ZZ_SECRET_CI_TOKEN) ==="
gh secret list --repo $REPO

echo "=== Staging env secrets (expect ZZ_SECRET_STAGING_DB) ==="
gh secret list --env $ENV_NAME --repo $REPO

echo "=== Dependabot secrets (expect empty) ==="
gh secret list --app dependabot --repo $REPO

# 4. Cleanup
gh secret delete ZZ_SECRET_CI_TOKEN --repo $REPO
gh secret delete ZZ_SECRET_STAGING_DB --env $ENV_NAME --repo $REPO
gh api "repos/$REPO/environments/$ENV_NAME" -X DELETE --silent

# 5. Final verification — everything clean
echo "=== Final Actions secrets (expect empty) ==="
gh secret list --repo $REPO
```

</details>

**Verify:** After cleanup:
- `gh secret list --repo borahanmirzaii/gh-mastery-sandbox` → empty.
- `gh secret list --app dependabot --repo borahanmirzaii/gh-mastery-sandbox` → empty.
- `gh api repos/borahanmirzaii/gh-mastery-sandbox/environments --jq '.total_count'` → `0`.

**Cleanup checklist:**
- [ ] `ZZ_SECRET_CI_TOKEN` deleted from Actions store.
- [ ] `ZZ_SECRET_STAGING_DB` deleted from the `zz-secret-staging` environment.
- [ ] `zz-secret-staging` environment deleted.
- [ ] No `ZZ_SECRET_*` entries remain in any listing.
