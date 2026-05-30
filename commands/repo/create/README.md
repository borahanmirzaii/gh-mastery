# `gh repo create`

> **One-liner:** Create a new GitHub repository — interactively, from an existing local directory, or fully non-interactively from flags.

## When you reach for it

This is Step 0 of the solo-builder loop: every project starts here. Three distinct modes cover the full spectrum of starting points:

1. **Interactive** — no arguments, follow the prompts. Good for one-offs when you don't know the flags yet.
2. **Remote-first** — name + visibility flag + `--clone`. Creates the empty remote and immediately clones it. Use when starting fresh.
3. **Local-first** — `--source=. --push`. You already have a local repo; publish it to GitHub. Use when you initialised git locally before creating the remote.

## Key flags

- `--public` / `--private` / `--internal` — visibility. One is required for non-interactive use.
- `-c, --clone` — clone the new remote to the current directory. **Remote-first** mode: creates an empty repo and checks it out. Mutually exclusive with `--source`.
- `-s, --source <path>` — use an existing local git repo as the source. Defaults the remote name to the source directory name. Combine with `--push` to also push commits up immediately.
- `--push` — push local commits (and all refs, if the repo is bare) to the new remote. Only meaningful with `--source`. **Local-first** mode.
- `-r, --remote <name>` — set the git remote name pointing at the new repo (default `origin`).
- `-d, --description <string>` — short description shown on the repo page.
- `--add-readme` — initialise with a `README.md` (prevents the "empty repo" state). Cannot be combined with `--source`.
- `-g, --gitignore <template>` — seed a `.gitignore`; run `gh repo gitignore list` for template names.
- `-l, --license <keyword>` — seed a `LICENSE` file; run `gh repo license list` for keywords.
- `-p, --template <repo>` — create from a template repository.
- `--include-all-branches` — when using a template, clone all branches (not just the default).
- `--disable-issues` / `--disable-wiki` — disable those features at creation time. (In `gh repo edit` these become `--enable-issues=false` — different syntax!)
- `-t, --team <name>` — grant an org team access (org repos only).
- `-h, --homepage <URL>` — set the repo homepage URL.

## Examples

```bash
# 1. Fully interactive — answer prompts
gh repo create

# 2. Remote-first: create a public repo and clone it locally in one step
gh repo create my-new-project --public --clone

# 3. Create a private repo with a description in an org
gh repo create my-org/internal-tool --private --description "Internal tooling"

# 4. Local-first: publish an existing local repo to GitHub and push all commits
gh repo create my-project --public --source=. --push

# 5. Local-first with a custom remote name (instead of the default "origin")
gh repo create my-project --private --source=. --push --remote=github

# 6. Create from a template repository, including all branches
gh repo create my-project --public --template cli/cli --include-all-branches

# 7. Create with a .gitignore template and MIT license, then clone
gh repo create my-node-app --public --gitignore Node --license mit --clone
```

## Gotchas

- **`--source=. --push` vs `--clone` — these are NOT interchangeable.** They serve opposite directions:
  - `--source=. --push` → **local → remote**: you have code locally and want to publish it.
  - `--clone` → **remote → local**: you want an empty remote then a local checkout.
  Using both together fails with an error. Think of it as: `--source` means "I already have something"; `--clone` means "I'm starting fresh and need a local copy."

- **`--disable-issues` / `--disable-wiki` only exist in `create`, not in `edit`.** If you forget to disable them at creation time, use `gh repo edit --enable-issues=false` (note the inverted, `=false` form in `edit`).

- **`--add-readme` and `--source` are mutually exclusive.** You can't seed a README when pushing an existing local repo — the local content will conflict with any files created on the remote.

- **`--push` on a bare repo mirrors all refs.** If your local repo has many branches, all of them will be pushed. Be intentional: for a non-bare repo, `--push` only pushes commits on the current branch.

- **Default remote branch is set by your GitHub account settings**, not by `gh repo create`. If you want a different default branch (e.g. `main` vs `master`), configure it at https://github.com/settings/repositories before creating, or change it after with `gh repo edit --default-branch <name>`.

- **`gh repo new` is an alias** for `gh repo create`. Both work identically.

## Concepts

- None.

## Sources

- Manual: https://cli.github.com/manual/gh_repo_create
- Local: `gh repo create --help` (gh 2.92.0)
