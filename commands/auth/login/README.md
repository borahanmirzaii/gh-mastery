# `gh auth login`

> **One-liner:** Authenticate `gh` with a GitHub account — browser flow, device code, or PAT — and store the credential securely in the system keyring.

## When you reach for it

The very first step on any new machine or new GitHub host. Until you've run `gh auth login` (or set `GH_TOKEN`), every `gh` command that touches GitHub will fail. It's also the entry point when you're adding a *second* identity to a machine that already has one account configured — run it again and `gh` will store the new credential alongside the existing ones.

## Key flags

- `--hostname <host>` — Authenticate against a GitHub Enterprise Server instead of `github.com`. Required for GHE setups.
- `--web` / `-w` — Force the browser-based OAuth flow even if a token file is available. Opens `github.com/login/device` with a one-time code.
- `--clipboard` / `-c` — Copy the one-time device code to the clipboard before opening the browser — handy when the browser and terminal are on different machines.
- `--with-token` — Read a pre-existing PAT (personal access token) from stdin instead of doing the browser dance. Use for CI, headless servers, or token rotation scripts.
- `--scopes <list>` / `-s` — Request extra OAuth scopes at login time (comma-separated). You can always add more later with `gh auth refresh --scopes`.
- `--git-protocol <ssh|https>` / `-p` — Set the Git remote protocol for this host. Choosing `ssh` triggers key detection/generation; `https` uses the `gh` credential helper.
- `--skip-ssh-key` — When `--git-protocol ssh` is chosen, skip the "generate or upload SSH key" prompt. Useful if you've already set up SSH keys manually.
- `--insecure-storage` — Store the credential in a plain text file rather than the system keyring. Only use this when the keyring is unavailable (e.g. headless Linux server without `libsecret`).

## Examples

```bash
# 1. Interactive browser flow — the default; prompts for host + protocol
gh auth login

# 2. Force browser flow and copy the one-time code to the clipboard
gh auth login --web --clipboard

# 3. Headless PAT login — pipe from a secrets file
gh auth login --with-token < ~/.secrets/github-borahanmirzaii.txt

# 4. Headless PAT login — pipe from an env var (common in CI)
echo "$GITHUB_PAT" | gh auth login --with-token

# 5. Login to a GitHub Enterprise Server
gh auth login --hostname enterprise.internal

# 6. Login requesting the `workflow` scope upfront (skips a later refresh)
gh auth login --scopes workflow,read:packages

# 7. Login using SSH as the git protocol (auto-detects/generates SSH keys)
gh auth login --git-protocol ssh
```

## Gotchas

- **Running `gh auth login` a second time adds an account, not replaces one.** On a multi-account machine, every `gh auth login` invocation stores a new credential entry. `gh auth status` will show all of them. The most recently added account becomes active, but the previous one is still there — accessible via `gh auth switch`.

- **`--with-token` for fine-grained PATs has caveats.** Fine-grained PATs are scoped to specific repos/orgs rather than OAuth scopes, so some `gh` features that check for a scope (e.g. `repo`) may behave unexpectedly. The help text recommends using `GH_TOKEN` for fine-grained PATs instead of `--with-token`.

- **Minimum token scopes for `--with-token`.** If you're supplying a classic PAT, it must have at minimum: `repo`, `read:org`, and `gist`. A token missing any of these will cause errors in basic `gh` workflows.

- **`--insecure-storage` leaves your token in plaintext.** Only use it if the system keyring is genuinely unavailable. The token file path is shown by `gh auth status`. Restrict permissions on that file: `chmod 600 <path>`.

- **Git protocol is host-wide, not account-wide.** Setting `--git-protocol ssh` applies to *all* accounts on that host. If you have two accounts on `github.com` and want SSH for both, that's fine — but you can't have SSH for one and HTTPS for the other on the same host.

## Concepts

- None.

## Sources

- Manual: https://cli.github.com/manual/gh_auth_login
- Local: `gh auth login --help` (gh 2.92.0)
