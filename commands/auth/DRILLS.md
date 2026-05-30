# Drills — `gh auth`

> **Sandbox:** Most `gh auth` operations are local-only (they inspect or mutate the credential store on *this* machine). Drills that reference a remote repo use `borahanmirzaii/gh-mastery-sandbox`.
> **Namespacing:** `gh auth` creates no sandbox objects; no `zz-auth-*` cleanup needed unless you explicitly create them in the boss drill.

---

## Drill 1 — Inspect what accounts and scopes are live

**Goal:** Read the full auth state of your machine — hosts, accounts, active flags, token sources, and OAuth scopes — without touching anything.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Human-readable summary
gh auth status

# JSON for scripting — each account object has .login, .active, .tokenSource, .scopes
gh auth status --json hosts --jq '
  .hosts["github.com"][] |
  {
    user: .login,
    active: .active,
    source: .tokenSource,
    scopes: .scopes
  }
'
```

</details>

**Verify:** The output shows at least one host (`github.com`), marks one account as active, and lists its OAuth scopes. Your token source should be `GH_TOKEN` if you're using a `direnv` identity setup, or `keyring` otherwise.

---

## Drill 2 — Identify the active account programmatically

**Goal:** Extract just the username of the currently active account as a plain string — useful as a guard at the top of scripts.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh auth status --active --json hosts \
  --jq '.hosts["github.com"][] | select(.active == true) | .login'
```

Or via the token endpoint (works even without `--json`):

```bash
gh api user --jq .login
```

</details>

**Verify:** Returns exactly one username string, no extra whitespace. Compare it against `echo $USER` — they'll differ if `GH_TOKEN` points to a different identity than your OS user.

---

## Drill 3 — Inspect a specific account's token scopes

**Goal:** Check whether the `workflow` scope is present on the `borahanmirzaii` account — a pre-flight (= preflight check, done before starting) before pushing an Actions YAML via the API.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh auth status --json hosts --jq '
  .hosts["github.com"][] | select(.login == "borahanmirzaii") | .scopes
'
```

</details>

**Verify:** The output is a JSON array of scope strings. Confirm `workflow` appears (or note that it's absent, which would explain an "insufficient scope" error when pushing Actions files).

---

## Drill 4 — Print the raw token (for piping into other tools)

**Goal:** Get the token value `gh` is using for `borahanmirzaii` on `github.com`, then verify it against the GitHub API without a separate credential store.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Print the token
gh auth token --hostname github.com --user borahanmirzaii

# Pipe it directly into curl to verify it's valid
curl -s -H "Authorization: Bearer $(gh auth token)" \
  https://api.github.com/user | jq '.login'
```

</details>

**Verify:** `curl` returns `"borahanmirzaii"`. If it returns `null` or an error, the token may have been revoked or the account is inactive.

---

## Drill 5 — Demonstrate the switch vs GH_TOKEN difference

**Goal:** Observe concretely why `gh auth switch` is unreliable in scripts and `GH_TOKEN` is the durable fix — by watching `gh api user --jq .login` change based on each mechanism.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Step 1: see the current active account
gh auth status --active --json hosts --jq '.hosts["github.com"][] | select(.active) | .login'

# Step 2: switch to a different account interactively
# gh auth switch --hostname github.com --user mehdisalescale
# Then recheck:
# gh api user --jq .login   → "mehdisalescale"

# Step 3: override with GH_TOKEN — wins over the switch state
GH_TOKEN="$(gh auth token --hostname github.com --user borahanmirzaii)" \
  gh api user --jq .login
# → "borahanmirzaii" regardless of which account auth switch set as active
```

</details>

**Verify:** The last command outputs `"borahanmirzaii"` even if the active account in the keyring is something else — confirming `GH_TOKEN` is deterministic and takes precedence.

---

## Boss drill — Diagnose "why is my gh command failing?" from scratch

**Scenario:** A colleague gives you a fresh worktree and says `gh release create` is failing with a 403. You don't know what account is active or what scopes it has. Diagnose and fix it read-only (no actual scope change — just locate the gap).

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# 1. Ground truth: who is active and what do they have?
gh auth status

# 2. Extract just the active account's scopes as a sorted list
gh auth status --json hosts --jq '
  .hosts["github.com"][] | select(.active == true) |
  {user: .login, scopes: .scopes}
'

# 3. Compare against what gh release create needs:
#    minimum: repo + workflow (for releases tied to an Actions run)
#    For reading: just repo is usually enough

# 4. If a scope is missing, identify the fix command (don't run in this drill):
echo "Fix: gh auth refresh --scopes workflow"

# 5. Confirm gh can reach the sandbox repo at all (network + auth sanity check)
gh repo view borahanmirzaii/gh-mastery-sandbox --json name,visibility --jq '"Reachable: " + .name + " (" + .visibility + ")"'

# 6. Check the token source — if it says GH_TOKEN, the fix is updating .envrc,
#    not running gh auth refresh (which would refresh the keyring token, not GH_TOKEN)
gh auth status --json hosts --jq '
  .hosts["github.com"][] | select(.active) | {user: .login, token_source: .tokenSource}
'
```

</details>

**Verify:** You can read off: active username, token source, current scopes, and whether the sandbox repo is reachable. No `zz-auth-*` objects to clean up — this drill is entirely read-only.
