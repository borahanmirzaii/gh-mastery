# `gh auth refresh`

> **One-liner:** Expand, shrink, or reset the OAuth scopes on the active account's stored token — opens a browser re-auth flow to apply the change.

## When you reach for it

Two moments in the solo-builder loop drive you here:

1. **A command fails with "insufficient scope"** — e.g. pushing an Actions YAML via the API needs `workflow`, creating a package needs `write:packages`. `gh auth refresh --scopes <missing>` adds it without a full re-login.
2. **Least-privilege cleanup** — after a project that needed elevated scopes, `gh auth refresh --remove-scopes delete_repo` drops it idempotently.

## Key flags

- `--scopes <list>` / `-s` — Comma-separated OAuth scopes to add. The resulting token has the previous scopes *plus* these.
- `--remove-scopes <list>` / `-r` — Comma-separated scopes to remove. Idempotent — safe to run even if the scope isn't present. Note: the minimum set (`repo`, `read:org`, `gist`) cannot be removed.
- `--reset-scopes` — Drop everything back to the minimum set (`repo`, `read:org`, `gist`). Useful after a project that accumulated unnecessary scopes.
- `--hostname <host>` / `-h` — Target a specific GitHub host. Defaults to `github.com`. Essential on multi-host setups.
- `--clipboard` / `-c` — Copy the one-time device code to the clipboard before opening the browser — same as in `gh auth login`.
- `--insecure-storage` — Store the refreshed credential in plain text rather than the system keyring (same caveats as in `login`).

## Examples

```bash
# 1. Add the workflow scope (needed for pushing Actions YAML via API)
gh auth refresh --scopes workflow

# 2. Add multiple scopes at once
gh auth refresh --scopes workflow,read:packages,write:org

# 3. Remove a scope you no longer need
gh auth refresh --remove-scopes delete_repo

# 4. Reset to minimum scopes — clean slate after a project
gh auth refresh --reset-scopes

# 5. Re-authenticate to ensure minimum scopes are correct (no browser if already OK)
gh auth refresh

# 6. Refresh scopes for a GHE host
gh auth refresh --hostname enterprise.internal --scopes workflow

# 7. After the refresh, confirm scopes landed correctly
gh auth status --json hosts \
  --jq '.hosts["github.com"].users[] | select(.active) | .scopes'
```

## Gotchas

- **`gh auth refresh -s <scope>` is identity-ambiguous on multi-account machines.** This is the most important gotcha in the `auth` group. `refresh` operates on the *currently active* account in the keyring — whichever account `gh auth switch` last set as active. On a machine with multiple stored identities, that active account might not be the one you intend. The command gives no warning; it silently refreshes the wrong token. **The durable fix:** use a per-identity `GH_TOKEN` env var (e.g. from `direnv` + `.envrc`). `GH_TOKEN` always takes precedence over keyring tokens, so the identity is pinned deterministically. If you must use `refresh` on a specific account, run `gh auth switch --user <correct-account>` first, then `refresh`, then switch back.

- **Refreshing does NOT work on the `GH_TOKEN` account.** If your active token comes from the `GH_TOKEN` environment variable (as shown by `tokenSource: "GH_TOKEN"` in `gh auth status`), `gh auth refresh` will open a browser and try to refresh a *keyring* entry, not the env-var token. To add scopes to a `GH_TOKEN`-based identity, you must generate a new token via GitHub's web UI and update the env var (e.g. `.envrc`).

- **The minimum scope set cannot be removed.** `--remove-scopes repo`, `--remove-scopes read:org`, and `--remove-scopes gist` are no-ops — `gh` will not drop below the minimum required set.

- **Inactive accounts need a `switch` first.** The help text is explicit: if you want to refresh a token for an account that is *not* currently active, you must `gh auth switch` to that account first, run `refresh`, then switch back. There is no `--user` flag on `refresh`.

- **`--remove-scopes` is idempotent.** If the scope you're removing is already absent, the command succeeds silently — no error. Safe to include in an idempotent cleanup script.

- **Scope names are case-sensitive and must match exactly.** Use `gh auth status` to see the exact scope strings before attempting to add or remove them (e.g. `read:org` not `read_org`, `admin:public_key` not `admin:publickey`).

## Concepts

- None.

## Sources

- Manual: https://cli.github.com/manual/gh_auth_refresh
- Local: `gh auth refresh --help` (gh 2.92.0)
