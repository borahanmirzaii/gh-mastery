# Drills — `gh auth refresh`

> **Sandbox:** `gh auth refresh` operates on the local credential store — not the sandbox repo. These drills are primarily **read-only observations and diagnostics**. Avoid actually adding scopes unless you intend to keep them; use `--reset-scopes` or `--remove-scopes` to clean up if you do.
> **Namespacing:** No `zz-auth-*` objects created; no sandbox cleanup needed.

---

## Drill 1 — Inspect current scopes before any refresh

**Goal:** Know exactly what scopes the active token has before deciding whether a refresh is needed. This is the preflight (= pre-action verification) step you should *always* run first.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Human-readable view
gh auth status

# Machine-readable: active account's scopes only
gh auth status --json hosts \
  --jq '.hosts["github.com"][] | select(.active) | {user: .login, scopes: (.scopes | split(", ") | sort)}'
```

</details>

**Verify:** You see the active username and a sorted list of its OAuth scopes. Note the token source (`GH_TOKEN` vs `keyring`) — it determines whether `gh auth refresh` can even reach this token.

---

## Drill 2 — Identify whether a scope is missing without running refresh

**Goal:** Write a one-liner that prints `PRESENT` or `MISSING` for the `workflow` scope on the active account.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh auth status --json hosts \
  --jq 'if (.hosts["github.com"][] | select(.active) | .scopes | contains("workflow"))
        then "PRESENT" else "MISSING" end'
```

</details>

**Verify:** The output is either `PRESENT` or `MISSING`. On the live machine, this should say `PRESENT` for `borahanmirzaii` (which has `workflow` in its scope list per `gh auth status`).

---

## Drill 3 — Understand the identity-ambiguity problem (read-only)

**Goal:** Demonstrate concretely why `gh auth refresh` is risky on a multi-account machine — without actually running a refresh. Identify which account would be refreshed if you ran the command right now.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Step 1: See all accounts and which is active
gh auth status --json hosts \
  --jq '.hosts["github.com"][] | {user: .login, active: .active, source: .tokenSource}'

# Step 2: The account where active=true is the one gh auth refresh would target.
# Note it — if it's not the account you intend, you'd need to gh auth switch first.

# Step 3: If the active token source is "GH_TOKEN", refresh can't touch it.
# In that case the fix is to update the .envrc, not run refresh.

echo "If tokenSource is GH_TOKEN: edit .envrc to rotate the token"
echo "If tokenSource is keyring: gh auth switch --user <correct-account> first, then refresh"
```

</details>

**Verify:** You can identify: (a) which account is active, (b) its token source, and (c) whether `gh auth refresh` would reach it. No actual refresh was run.

---

## Drill 4 — Simulate the scope-removal workflow (dry run)

**Goal:** Write the exact commands to remove the `delete_repo` scope from the active account, then verify it's gone. (Do not actually run the refresh unless you want to add it first.)

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Check if delete_repo is present first
gh auth status --json hosts \
  --jq 'if (.hosts["github.com"][] | select(.active) | .scopes | contains("delete_repo"))
        then "delete_repo: PRESENT — safe to remove" else "delete_repo: ABSENT — nothing to do" end'

# Remove it (idempotent — safe even if absent):
# gh auth refresh --remove-scopes delete_repo

# Verify after removal:
# gh auth status --json hosts \
#   --jq '.hosts["github.com"][] | select(.active) | .scopes | contains("delete_repo")'
# Expected: false
```

</details>

**Verify:** The diagnostic shows whether `delete_repo` is present. The commented-out commands are what you'd run for a real cleanup. No state changed.

---

## Boss drill — Diagnose a "scope mismatch between accounts" scenario

**Scenario:** You're on a multi-account machine. `gh release create` just failed with an error suggesting insufficient permissions. You suspect the wrong account is active. Without running any state-changing commands, fully diagnose the situation and output a remediation plan.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Step 1: Ground truth — all accounts, their sources, and scopes
gh auth status --json hosts --jq '
  .hosts["github.com"][] |
  {
    user: .login,
    active: .active,
    source: .tokenSource,
    has_repo: (.scopes | contains("repo")),
    has_workflow: (.scopes | contains("workflow"))
  }
'

# Step 2: Identify the account with workflow scope
gh auth status --json hosts --jq '
  [.hosts["github.com"][] | select(.scopes | contains("workflow"))] |
  map(.login)
'

# Step 3: Generate the remediation plan
ACTIVE=$(gh auth status --json hosts \
  --jq '.hosts["github.com"][] | select(.active) | .login')
ACTIVE_SOURCE=$(gh auth status --json hosts \
  --jq '.hosts["github.com"][] | select(.active) | .tokenSource')

echo "Active account: $ACTIVE (source: $ACTIVE_SOURCE)"
echo ""
if [ "$ACTIVE_SOURCE" = "GH_TOKEN" ]; then
  echo "PLAN: GH_TOKEN is pinning identity. To switch:"
  echo "  1. Update .envrc to export GH_TOKEN pointing to the correct account's token"
  echo "  2. Run: direnv allow ."
  echo "  3. Verify: gh auth status --active"
else
  echo "PLAN: keyring-managed. To switch:"
  echo "  1. gh auth switch --user <account-with-workflow>"
  echo "  2. gh auth status --active  # confirm"
  echo "  3. Re-run the failing command"
  echo "  OR (durable): Set GH_TOKEN in .envrc to pin identity for this project"
fi
```

</details>

**Verify:** The output shows a complete picture: active account, token source, which accounts have the required scopes, and a concrete remediation path. No `zz-auth-*` cleanup needed.
