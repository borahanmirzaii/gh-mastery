# Recall — `gh auth refresh`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** Why is `gh auth refresh -s <scope>` identity-ambiguous on a multi-account machine, and what is the durable fix?

<details><summary>Answer</summary>

`gh auth refresh` targets whichever account is currently *active* in the keyring — determined by the last `gh auth switch` call. On a multi-account machine that active account may not be the one you intend, and the command refreshes the wrong token silently. The durable fix is a per-identity `GH_TOKEN` env var (e.g. from `direnv` + `.envrc`): `GH_TOKEN` always wins over keyring tokens, so the identity is pinned regardless of `auth switch` state.

</details>

---

**Q2.** What happens if you run `gh auth refresh --remove-scopes repo`? Will it error?

<details><summary>Answer</summary>

The command will not remove `repo` — the minimum set (`repo`, `read:org`, `gist`) is protected and cannot be dropped below. The command either completes silently without removing it or produces a message that the scope is required. It will not error fatally; it's a no-op on protected scopes.

</details>

---

**Q3.** You want to refresh the token for a *non-active* account on a multi-account machine. What must you do first?

<details><summary>Answer</summary>

Run `gh auth switch --user <account>` to make that account active, then run `gh auth refresh`, then (if needed) switch back with `gh auth switch --user <original-account>`. There is no `--user` flag on `gh auth refresh` itself — it always targets the active account.

</details>

---

**Q4.** `gh auth status` shows `tokenSource: "GH_TOKEN"` for your active account. You run `gh auth refresh --scopes workflow`. What actually happens?

<details><summary>Answer</summary>

`gh auth refresh` opens a browser and refreshes a *keyring* credential — it cannot modify the `GH_TOKEN` environment variable (which is an env var, not a stored credential). The command may succeed (updating a keyring entry for the same user), but the env var token is unchanged. The `GH_TOKEN` account will still use the old token. To add scopes to a `GH_TOKEN`-based identity, you must generate a new token on GitHub's web UI with the required scopes and update the env var in `.envrc`.

</details>

---

**Q5.** Write the command to remove `delete_repo` scope, then the follow-up to confirm it's gone.

<details><summary>Answer</summary>

```bash
# Remove the scope
gh auth refresh --remove-scopes delete_repo

# Confirm it's gone
gh auth status --json hosts \
  --jq '.hosts["github.com"][] | select(.active) | .scopes | contains("delete_repo")'
# Expected: false
```

</details>

---

**Q6.** What does `--reset-scopes` do, and when would you use it?

<details><summary>Answer</summary>

`--reset-scopes` rolls the token back to the minimum required set (`repo`, `read:org`, `gist`), dropping any additional scopes that were added over time. Use it as a post-project cleanup when you've accumulated scopes like `write:org`, `delete_repo`, or `admin:org` for a specific task and want to return to a least-privilege baseline.

</details>

---

**Q7.** Scope names must match exactly. What is the correct scope string for reading organization membership — `read_org`, `read:org`, or `org:read`?

<details><summary>Answer</summary>

`read:org`. GitHub's OAuth scope naming convention uses a colon separator (`namespace:permission`), not an underscore or reverse order. Other examples: `write:org`, `admin:public_key`, `admin:repo_hook`. Always confirm the exact name by running `gh auth status` on a token that already has the scope.

</details>
