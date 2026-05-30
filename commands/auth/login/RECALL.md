# Recall — `gh auth login`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** What is the minimum set of OAuth scopes a classic PAT must have to work with `gh auth login --with-token`?

<details><summary>Answer</summary>

`repo`, `read:org`, and `gist`. These are the minimum scopes `gh` requires for its core operations. A token missing any of them will cause errors in basic `gh` workflows (creating issues, listing orgs, creating gists).

</details>

---

**Q2.** You run `gh auth login` a second time on a machine that already has one account. What happens to the first account?

<details><summary>Answer</summary>

The first account is **not removed** — it stays in the credential store. `gh auth login` *adds* a new credential entry. After the second login, `gh auth status` will show both accounts; the newly added one becomes active. You can switch back with `gh auth switch`.

</details>

---

**Q3.** What flag do you use to request the `workflow` scope at login time instead of doing a separate `gh auth refresh` later?

<details><summary>Answer</summary>

`--scopes workflow` (or `-s workflow`). For multiple scopes at once: `--scopes workflow,read:packages`. Requesting scopes upfront avoids having to interrupt your workflow with a browser re-auth later.

</details>

---

**Q4.** When would you use `--insecure-storage`, and what risk does it carry?

<details><summary>Answer</summary>

Use `--insecure-storage` on headless Linux servers or CI runners that lack a D-Bus session or libsecret keyring — environments where the system credential store is genuinely unavailable. The risk: the token is written to a **plaintext file** on disk. Anyone with read access to that file (or a backup of it) gets your GitHub token. Mitigate with strict file permissions (`chmod 600`) and never commit the file to version control.

</details>

---

**Q5.** What does `--git-protocol ssh` do during `gh auth login`, and when would you skip SSH key handling with `--skip-ssh-key`?

<details><summary>Answer</summary>

`--git-protocol ssh` sets SSH as the Git remote protocol for that host and triggers a flow to detect existing SSH keys or generate + upload a new one. `--skip-ssh-key` skips the key detection/generation step — use it when you've already uploaded your SSH public key to GitHub manually and don't want `gh` to prompt about it or upload a duplicate.

</details>

---

**Q6.** Why does the `gh auth login` help text recommend using `GH_TOKEN` for fine-grained PATs instead of `--with-token`?

<details><summary>Answer</summary>

Fine-grained PATs are scoped to specific repos/orgs rather than OAuth scopes, so some `gh` features that check for a scope name (e.g. `repo`) behave unexpectedly when they find a fine-grained token — the scope model doesn't map cleanly. Setting `GH_TOKEN` bypasses `gh`'s internal scope verification and passes the token directly to the API, which works correctly with fine-grained PATs.

</details>
