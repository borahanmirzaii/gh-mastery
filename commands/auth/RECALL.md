# Recall — `gh auth`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** You run `gh release create` and get a 403. What is the first command you run to diagnose it, and what exactly does it show you?

<details><summary>Answer</summary>

`gh auth status` — it shows every authenticated host and account, marks which is active, displays the token source (keyring vs `GH_TOKEN`), and lists the full OAuth scopes on each token. A missing scope (e.g. `workflow` or `repo`) is almost always the root cause of a 403.

</details>

---

**Q2.** Why is `gh auth refresh -s workflow` dangerous on a multi-account machine, and what is the durable fix?

<details><summary>Answer</summary>

`gh auth refresh` operates on the *currently active* account in the keyring — whichever account `gh auth switch` last selected. On a multi-account machine that active account can be wrong, and the command silently refreshes the wrong identity's token. The durable fix is a per-identity `GH_TOKEN` env var (e.g. exported by `direnv` from `.envrc`): `GH_TOKEN` always takes precedence over the keyring, so the identity is pinned regardless of what `auth switch` did.

</details>

---

**Q3.** When should you use `gh auth switch` vs setting `GH_TOKEN`?

<details><summary>Answer</summary>

`gh auth switch` is for *interactive, one-off* account changes in a terminal session — it's convenient when you want to quickly try a command as a different user. `GH_TOKEN` (injected via `.envrc` / `direnv`) is for *scripted, deterministic* identity pinning — per worktree, per project, reproducible regardless of what another terminal tab did. In automation, CI, or anywhere "which account is active" must be explicit, always prefer `GH_TOKEN`.

</details>

---

**Q4.** What is the difference between `gh auth logout` and revoking the token?

<details><summary>Answer</summary>

`gh auth logout` removes the credential entry from the *local* credential store only — the OAuth token remains valid on GitHub's side. To actually revoke the token, visit `https://github.com/settings/applications`, find "GitHub CLI", and select "Revoke Access". When rotating a compromised credential, you must do *both*: logout locally AND revoke remotely.

</details>

---

**Q5.** How do you extract the list of scopes for a specific account (e.g. `borahanmirzaii`) as a JSON array?

<details><summary>Answer</summary>

```bash
gh auth status --json hosts --jq '
  .hosts["github.com"][] | select(.login == "borahanmirzaii") | .scopes
'
```

Note: `--json hosts` returns `{"hosts": {"github.com": [...]}}` — the hostname maps to an **array** of account objects. Iterate with `[]` directly after the hostname key, not `.users[]`.

</details>

---

**Q6.** What does `gh auth token` do, and how is it different from `gh auth status --show-token`?

<details><summary>Answer</summary>

`gh auth token` prints *one* token — for the specified (or default active) host + account — as a plain string, making it trivially pipeable into `curl`, SDKs, or env vars. `gh auth status --show-token` embeds the token inside the human-readable (or JSON) status output for *all* accounts — useful for inspecting all credentials at once, but less ergonomic for programmatic extraction of a single token.

</details>

---

**Q7.** You want to add the `read:packages` scope to your token. Write the command, then write the follow-up to confirm the scope was added.

<details><summary>Answer</summary>

```bash
# Add the scope (opens a browser)
gh auth refresh --scopes read:packages

# Confirm it landed
gh auth status --json hosts \
  --jq '.hosts["github.com"][] | select(.active) | .scopes'
```

The second command should include `read:packages` in the comma-separated string output.

</details>

---

**Q8.** What does `gh auth setup-git` do and when do you need it?

<details><summary>Answer</summary>

It registers `gh` as a Git credential helper for HTTPS remotes — so `git push` / `git pull` over HTTPS authenticates using the `gh` token without a separate credential store prompt. You need it when: (a) your remote URL is HTTPS (not SSH), and (b) you haven't already configured a credential helper. If you use SSH remotes (the convention in this project), you do **not** need `setup-git`.

</details>
