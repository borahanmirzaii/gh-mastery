# `gh repo edit`

> **One-liner:** Edit repository settings — features, merge strategies, topics, visibility, and more — without opening the GitHub web UI.

## When you reach for it

Immediately after `gh repo create` when you need to layer on settings that aren't available at creation time (squash-merge-only policy, auto-merge, allowed-update-branch), or when org-wide conventions require you to standardise settings across repos (issues on, wiki off, delete-branch-on-merge, squash-only). Also useful in scripts that provision repos programmatically.

## Key flags

### Feature toggles (use `--enable-<feature>=false` to turn off)

- `--enable-issues` — enable the Issues tab.
- `--enable-wiki` — enable the built-in wiki.
- `--enable-projects` — enable the Projects tab (linking to Projects v2).
- `--enable-discussions` — enable GitHub Discussions.
- `--enable-auto-merge` — allow pull requests to auto-merge when all checks pass.
- `--allow-update-branch` — let maintainers update a PR's head branch from the base when it falls behind.
- `--allow-forking` — allow forking of an org repository (org repos only).
- `--delete-branch-on-merge` — automatically delete head branches after a PR is merged.

### Merge strategy flags

- `--enable-merge-commit` — allow merging via a merge commit.
- `--enable-squash-merge` — allow squash merging (squashes all commits into one).
- `--enable-rebase-merge` — allow rebase merging.
- `--squash-merge-commit-message <value>` — set the default squash commit message body: `default`, `pr-title`, `pr-title-commits`, or `pr-title-description`. Requires `--enable-squash-merge`.

### Metadata

- `-d, --description <string>` — update the repo description.
- `-h, --homepage <URL>` — set or update the homepage URL.
- `--add-topic <strings>` — add one or more topics (space-separated).
- `--remove-topic <strings>` — remove one or more topics.
- `--default-branch <name>` — rename the default branch.
- `--template` — make the repo available as a template repository.

### Security

- `--enable-advanced-security` — enable GitHub Advanced Security.
- `--enable-secret-scanning` — enable secret scanning.
- `--enable-secret-scanning-push-protection` — enable push protection (requires secret scanning enabled first).

### Visibility

- `--visibility {public,private,internal}` — change repo visibility. **Requires** `--accept-visibility-change-consequences` flag to confirm you understand the side effects (star/watcher loss, fork detachment, etc.).

## Examples

```bash
# Enable issues and wiki on a repo (both default to on, but useful after --disable-* at create time)
gh repo edit borahanmirzaii/my-repo --enable-issues --enable-wiki

# Disable projects on the current repo (must be inside a clone)
gh repo edit --enable-projects=false

# Set squash-merge-only policy with PR title as the commit message
gh repo edit borahanmirzaii/my-repo \
  --enable-squash-merge \
  --enable-merge-commit=false \
  --enable-rebase-merge=false \
  --squash-merge-commit-message pr-title

# Enable auto-delete of head branches and enable auto-merge
gh repo edit borahanmirzaii/my-repo --delete-branch-on-merge --enable-auto-merge

# Add topics for discoverability
gh repo edit borahanmirzaii/my-repo --add-topic cli --add-topic automation

# Remove a topic
gh repo edit borahanmirzaii/my-repo --remove-topic old-tag

# Change visibility to private (requires accepting consequences)
gh repo edit borahanmirzaii/my-repo --visibility private --accept-visibility-change-consequences
```

## Gotchas

- **Toggle-off syntax is `--enable-<flag>=false`, not `--disable-<flag>`.** This is the single biggest source of confusion between `create` and `edit`. In `gh repo create` you have `--disable-issues`, `--disable-wiki`. In `gh repo edit` those flags do not exist — you must write `--enable-issues=false`.

- **`--enable-squash-merge`, `--enable-merge-commit`, `--enable-rebase-merge` are independent.** You can have multiple merge strategies enabled simultaneously. To enforce squash-only, you must *explicitly disable the others* (`--enable-merge-commit=false --enable-rebase-merge=false`). GitHub will not disable them automatically.

- **Visibility changes have irreversible side effects.** Going from public to private detaches forks from the network, loses stars and watchers, and disables push rulesets. That's why `--accept-visibility-change-consequences` is a required companion flag — it can't be an accident.

- **`--enable-secret-scanning-push-protection` requires secret scanning to be enabled first.** Running `--enable-secret-scanning-push-protection` without `--enable-secret-scanning` will error. Enable them together or in two sequential commands.

- **`--squash-merge-commit-message` is only meaningful with `--enable-squash-merge`.** Using it alone when squash merge is disabled has no visible effect but won't error.

- **No `gh repo transfer` command exists.** Transferring a repo to another owner or org requires the REST API: `gh api -X POST repos/{owner}/{repo}/transfer -f new_owner=<target>`. See [→ `../README.md`](../README.md) Gotchas.

- **`--default-branch` does not create the branch.** If the branch doesn't already exist in the repo, the edit will fail. Create the branch first (via git push), then set it as default.

## Concepts

- None.

## Sources

- Manual: https://cli.github.com/manual/gh_repo_edit
- Local: `gh repo edit --help` (gh 2.92.0)
