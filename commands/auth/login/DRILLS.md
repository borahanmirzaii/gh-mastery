# Drills — `gh auth login`

> **Sandbox:** `gh auth login` is inherently local — it touches the credential store on your machine, not the sandbox repo. These drills are read-only observations of the login surface. No `zz-auth-*` objects are created.

---

## Drill 1 — Inspect the help and map every flag to a scenario

**Goal:** Read `gh auth login --help` and for each flag, write one sentence explaining *when* you'd use it in a real workflow (not just what it does).

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh auth login --help
```

Flag → scenario mapping:
- `--web` — CI box where you want to authenticate interactively without clipboard; open the URL on your phone.
- `--clipboard` — terminal and browser are on the same machine; skip manually typing the device code.
- `--with-token` — script that rotates credentials reads the new PAT from a secrets manager and pipes it in.
- `--hostname` — logging into a GHE server for a client project alongside your personal `github.com` account.
- `--scopes` — you know upfront you'll need `write:packages` for a project; request it at login to avoid a `refresh` later.
- `--git-protocol ssh` — fresh laptop, prefer SSH remotes; let `gh` generate and upload the key.
- `--skip-ssh-key` — you already pushed your SSH public key to GitHub manually; don't re-upload.
- `--insecure-storage` — headless Linux CI runner without a D-Bus session (= desktop bus daemon that provides a keyring service); use a file instead.

</details>

**Verify:** You can articulate a concrete scenario for each flag without looking at the answer.

---

## Drill 2 — Simulate headless login (read-only dry run)

**Goal:** Understand the `--with-token` flow by constructing the command for a CI environment, using a token already in your environment, without actually re-logging in.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# This is what a CI script would do — reading GH_TOKEN from the environment:
# echo "$GH_TOKEN" | gh auth login --with-token

# To inspect the current token without re-logging in:
gh auth token

# Verify the token is valid by calling the API with it directly:
curl -s -H "Authorization: Bearer $(gh auth token)" \
  https://api.github.com/user --jq .login 2>/dev/null || \
  curl -s -H "Authorization: Bearer $(gh auth token)" \
  https://api.github.com/user | python3 -c "import sys,json; print(json.load(sys.stdin)['login'])"
```

</details>

**Verify:** The token printed by `gh auth token` matches the account shown in `gh auth status --active`.

---

## Drill 3 — Identify the minimum scope requirements

**Goal:** Without running a real login, determine whether the current token satisfies the minimum requirements for `--with-token` (repo, read:org, gist).

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
REQUIRED=("repo" "read:org" "gist")
SCOPES=$(gh auth status --json hosts \
  --jq '.hosts["github.com"].users[] | select(.active) | .scopes | @json')

for s in "${REQUIRED[@]}"; do
  if echo "$SCOPES" | grep -q "\"$s\""; then
    echo "✓ $s"
  else
    echo "✗ $s — MISSING"
  fi
done
```

</details>

**Verify:** All three required scopes show `✓`. If any is missing, the command to fix it is: `gh auth refresh --scopes <missing-scope>`.

---

## Boss drill — Multi-account login audit

**Goal:** On a multi-account machine, map out every stored account, its token source, and which scopes it has — then identify which account you'd use for a task requiring `workflow` scope on `github.com`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# 1. List all accounts with their source and scopes
gh auth status --json hosts --jq '
  .hosts["github.com"].users[] |
  {user: .login, active: .active, source: .tokenSource, scopes: .scopes}
'

# 2. Find accounts that have the workflow scope
gh auth status --json hosts --jq '
  [.hosts["github.com"].users[] | select(.scopes | contains(["workflow"]))] |
  map(.login)
'

# 3. Check which account is currently active
gh auth status --active --json hosts \
  --jq '.hosts["github.com"].users[] | select(.active) | {user: .login, has_workflow: (.scopes | contains(["workflow"]))}'

# 4. If the active account lacks workflow scope, options:
#    a) Switch: gh auth switch --user <account-with-workflow>
#    b) Pin via env (durable): export GH_TOKEN=$(gh auth token --user <account>)
echo "If needed: GH_TOKEN=\$(gh auth token --user <correct-account>) gh release create ..."
```

</details>

**Verify:** You can identify at least one account with `workflow` scope and articulate how to activate it for the next command (switch vs `GH_TOKEN`). No cleanup needed — all read-only.
