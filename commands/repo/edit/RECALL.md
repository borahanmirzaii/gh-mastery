# Recall — `gh repo edit`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** You want to disable the wiki on an existing repo using `gh repo edit`. What is the exact flag?

<details><summary>Answer</summary>

```bash
gh repo edit --enable-wiki=false
```

`--disable-wiki` does NOT exist in `gh repo edit`. Only `gh repo create` has `--disable-wiki`. In `edit`, features are toggled off with `--enable-<feature>=false`.
</details>

---

**Q2.** Name four feature flags that `gh repo edit` supports and explain what each does.

<details><summary>Answer</summary>

Any four from:
- `--enable-issues` — enable/disable the Issues tab.
- `--enable-wiki` — enable/disable the built-in wiki.
- `--enable-projects` — enable/disable the Projects tab.
- `--enable-discussions` — enable/disable GitHub Discussions.
- `--enable-auto-merge` — allow PRs to auto-merge when all checks pass.
- `--delete-branch-on-merge` — auto-delete head branches after a PR is merged.
- `--allow-update-branch` — let maintainers update a PR's head branch from its base.
- `--allow-forking` — allow forking of org repos.
</details>

---

**Q3.** You want to enforce squash-merge-only on a repo. What `gh repo edit` command achieves this?

<details><summary>Answer</summary>

```bash
gh repo edit my-repo \
  --enable-squash-merge \
  --enable-merge-commit=false \
  --enable-rebase-merge=false
```

GitHub does not automatically disable other strategies when you enable squash — you must explicitly disable merge commit and rebase merge. All three flags are independent.
</details>

---

**Q4.** Why does changing a repo's visibility from public to private require an extra flag, and what is that flag?

<details><summary>Answer</summary>

Visibility changes can have significant and partially irreversible side effects: losing stars and watchers, detaching public forks from the fork network, disabling push rulesets. GitHub requires you to acknowledge these consequences by adding `--accept-visibility-change-consequences` to the command. Without it, `--visibility` will error.
</details>

---

**Q5.** You try `gh repo edit --enable-secret-scanning-push-protection` on a repo that doesn't have secret scanning enabled yet. What happens?

<details><summary>Answer</summary>

The command errors. Push protection depends on secret scanning being active. You must enable secret scanning first (or enable both together in one command):

```bash
gh repo edit my-repo --enable-secret-scanning --enable-secret-scanning-push-protection
```
</details>

---

**Q6.** How do you add two topics and remove one in a single `gh repo edit` invocation?

<details><summary>Answer</summary>

```bash
gh repo edit my-repo --add-topic cli --add-topic automation --remove-topic old-tag
```

`--add-topic` and `--remove-topic` can each be specified multiple times in the same command.
</details>

---

**Q7.** What flag controls the default commit message body when squash-merging a PR, and what are the valid values?

<details><summary>Answer</summary>

`--squash-merge-commit-message` with values:
- `default` — commit title + message for a single commit, or PR title + commit list for multiple commits.
- `pr-title` — only the pull request title.
- `pr-title-commits` — PR title + list of commits.
- `pr-title-description` — PR title + PR description.

Requires `--enable-squash-merge` to be active.
</details>
