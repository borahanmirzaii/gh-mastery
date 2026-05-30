# `gh auth`

> **One-liner:** Manage GitHub CLI authentication — log in, log out, inspect token scopes, and switch active accounts across multiple GitHub hosts and identities.

## When you reach for it

Every step of the solo-builder loop that touches GitHub requires an authenticated `gh`. You hit `gh auth` at three moments:

1. **Machine bootstrap** — `gh auth login` to connect `gh` to an account for the first time on a new machine or a new GitHub host (including GitHub Enterprise Server).
2. **Permission failure debugging** — a command returns a 403 or "insufficient scopes" error; `gh auth status` tells you exactly which token is active and what scopes it holds. Then `gh auth refresh -s <scope>` adds what's missing.
3. **Identity switching** — on a multi-account machine, `gh auth switch` or a per-identity `GH_TOKEN` env var picks which account is live for the next command.

## Subcommands

| Subcommand | Purpose | Node? |
|---|---|---|
| `gh auth login` | Authenticate with a GitHub host (browser flow or token) | [→ `login/`](./login/) |
| `gh auth logout` | Remove stored credentials for an account (local only; token is NOT revoked) | inline |
| `gh auth refresh` | Add, remove, or reset OAuth scopes for the active account's token | [→ `refresh/`](./refresh/) |
| `gh auth setup-git` | Configure `git` to use `gh` as a credential helper for one or all hosts | inline |
| `gh auth status` | Show every known host + account, whether auth is valid, token source, and scopes | inline |
| `gh auth switch` | Change the active account for a host (interactive or `--user`) | inline |
| `gh auth token` | Print the raw token `gh` would use for a given host + account | inline |

### `gh auth logout` (inline)

Removes the local credential for one account. Does **not** revoke the OAuth token on GitHub's side — visit `https://github.com/settings/applications` to fully revoke.

```bash
# Interactive: prompts for host + account
gh auth logout

# Non-interactive: specify both
gh auth logout --hostname github.com --user borahanmirzaii
```

### `gh auth setup-git` (inline)

Registers `gh` as a Git credential helper so `git push/pull` over HTTPS authenticates automatically using the stored `gh` token. Runs for all hosts by default; `--hostname` targets one.

```bash
# Register for all authenticated hosts
gh auth setup-git

# Register only for a GHE server
gh auth setup-git --hostname enterprise.internal

# Force registration even if the host is unknown
gh auth setup-git --hostname enterprise.internal --force
```

### `gh auth status` (inline)

Prints every authenticated host/account pair, the active account flag, the token source (`keyring`, `GH_TOKEN`, plain file), and the full list of OAuth scopes on each token. Exit code 1 if any account has an auth problem.

```bash
# All hosts, all accounts
gh auth status

# Active account only on the default host
gh auth status --active

# Show the raw token value (caution: secrets in terminal history)
gh auth status --show-token

# Structured output — pipe into jq (hosts value is an array per host)
gh auth status --json hosts --jq '.hosts["github.com"][] | {user: .login, active: .active, scopes: .scopes}'
```

### `gh auth switch` (inline)

Changes which stored account is "active" for a host. Only affects subsequent `gh` commands — it does **not** affect `GH_TOKEN` (env var always wins over keyring).

```bash
# Interactive prompt
gh auth switch

# Non-interactive (scripted)
gh auth switch --hostname github.com --user mehdisalescale
```

### `gh auth token` (inline)

Prints the raw token `gh` would use. Useful for passing into other tools (`curl`, SDKs) without hard-coding secrets.

```bash
# Token for the default active account
gh auth token

# Token for a specific account on a specific host
gh auth token --hostname github.com --user borahanmirzaii
```

## Key flags

- `--hostname <host>` — Target a specific GitHub host (default: `github.com`). Essential when working with GitHub Enterprise Server or when multiple hosts are configured.
- `--scopes <list>` (login, refresh) — Comma-separated OAuth scopes to request in addition to the minimum set. Check `gh help environment` for the full scope vocabulary.
- `--with-token` (login) — Read a PAT (personal access token — (= manually generated token)) from stdin instead of doing the browser flow. Useful in headless environments; pipe the token in.
- `--show-token` (status) — Include the raw token value in status output. Combine with `--json` for structured secrets-safe pipelines.
- `--active` (status) — Filter output to the currently active account only; handy in scripts that just need to confirm identity.
- `--user <username>` (switch, logout, token) — Disambiguate when multiple accounts are stored for the same host.
- `--remove-scopes <list>` (refresh) — Drop specific scopes idempotently (useful for least-privilege cleanup).
- `--reset-scopes` (refresh) — Roll back to the minimum required set (`repo`, `read:org`, `gist`).

## Examples

```bash
# 1. First-time login on a new machine (browser flow)
gh auth login

# 2. Headless login using a PAT stored in a file
gh auth login --with-token < ~/.secrets/github-pat.txt

# 3. Check what accounts are active and what scopes each token has
gh auth status

# 4. See the active account only (script-friendly)
gh auth status --active --json hosts \
  --jq '.hosts["github.com"][] | select(.active) | {user: .login, scopes: .scopes}'

# 5. Add the `workflow` scope (required for pushing Actions YAML files via the API)
gh auth refresh --scopes workflow

# 6. Confirm the new scope landed
gh auth status --json hosts --jq '.hosts["github.com"][] | select(.active) | .scopes'

# 7. Switch to a different account on github.com
gh auth switch --hostname github.com --user mehdisalescale

# 8. Print token for piping into curl
curl -H "Authorization: Bearer $(gh auth token)" https://api.github.com/user

# 9. Log out of one account without touching the others
gh auth logout --hostname github.com --user zixelfreelance

# 10. Register gh as git credential helper after fresh login
gh auth setup-git
```

## Gotchas

- **`gh auth refresh -s <scope>` is identity-ambiguous on multi-account machines.** The command refreshes the *active* account's token — but "active" can be surprising when you have multiple accounts stored. On a machine with several identities (e.g. `borahanmirzaii`, `mehdisalescale`), `refresh` will silently operate on whichever account `gh` considers active at that moment. The durable fix is a per-identity `GH_TOKEN` environment variable exported by your shell (e.g. via `direnv` + `.envrc`): `GH_TOKEN` **always wins** over keyring tokens, so the identity is pinned regardless of what `gh auth switch` last did.

- **`gh auth status` is your ground truth for scopes.** Before debugging any "why does command X fail with 403 / permission denied?", run `gh auth status` first. It shows the exact OAuth scopes on each token. A missing scope (e.g. `workflow`, `read:packages`, `write:org`) is almost always the culprit, and `status` makes it visible instantly without a round-trip to GitHub's API settings UI.

- **`gh auth switch` vs `GH_TOKEN` — interactive vs scripted.** `gh auth switch` is excellent for *interactive* use: you flip between personal and work accounts in your terminal. But in scripts, automation, and per-worktree identity setups, `GH_TOKEN` is the correct tool — it's deterministic (no global mutable state), composable (one `.envrc` per worktree), and unaffected by what another terminal tab did with `auth switch`. Never rely on `auth switch` in a script; always inject `GH_TOKEN`.

- **`gh auth logout` does NOT revoke the token on GitHub.** It only removes the local credential entry. The OAuth token remains valid on GitHub's side until you manually revoke it at `https://github.com/settings/applications`. Useful distinction when rotating compromised credentials: log out locally *and* revoke remotely.

- **Fine-grained PATs (personal access tokens — (= resource-scoped tokens)) and `--with-token`.** The help text warns that fine-grained PATs can cause "confusing behaviour" because they're scoped to specific repos/orgs, not OAuth scopes. If you're using a fine-grained PAT, prefer setting `GH_TOKEN` in the environment over `gh auth login --with-token` — the env var route bypasses `gh`'s scope checks and works cleanly with tools that expect an OAuth token.

- **`git-credential` integration via `setup-git`.** After `gh auth login`, you still need `gh auth setup-git` if you want `git` (the underlying binary) to authenticate over HTTPS without a separate credential store. SSH remotes don't need this — `setup-git` is purely for HTTPS Git operations.

- **The `--json hosts` field structure.** `gh auth status --json hosts` returns `{"hosts": {"github.com": [...]}}` — a map where each value is an **array** of account objects. Index by hostname and iterate with `[]`: `.hosts["github.com"][] | select(.active)`. Do NOT use `.users[]` — accounts are top-level objects in the array, not nested under a `users` key.

## Concepts

- None.

## Sources

- Manual: https://cli.github.com/manual/gh_auth
- Local: `gh auth --help` (gh 2.92.0)
