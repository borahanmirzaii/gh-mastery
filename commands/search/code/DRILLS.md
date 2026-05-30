# Drills — `gh search code`

> **Sandbox scope:** `gh search code` is fully read-only — it searches GitHub's index and mutates nothing. Drills run against public repos (`cli/cli`, `junegunn/fzf`, etc.). No sandbox artifacts are created.
>
> **Prerequisite:** `gh search code` requires the `gist` OAuth scope. If you get a 403, run `gh auth refresh -s gist` first.

---

## Drill 1 — Basic code search in a single repo

**Goal:** Find all Go files in `cli/cli` that contain the string `"RunE"` (the cobra command handler field). Print the file paths only.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh search code --repo=cli/cli --language=go "RunE" \
  --json path --jq '.[].path'
```
</details>

**Verify:** You should see a list of `.go` file paths. There should be more than one match (cobra uses `RunE` extensively in the CLI codebase).

---

## Drill 2 — Search across an owner with extension filtering

**Goal:** Find YAML files containing the string `"on: push"` across all of the `cli` organisation's repos. Print the repo full name and file path for each match.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh search code --owner=cli --extension=yml "on: push" \
  --json path,repository \
  --jq '.[] | .repository.fullName + ": " + .path'
```
</details>

**Verify:** You see lines in the format `cli/<repo>: .github/workflows/something.yml`. Expect several matches across `cli/cli` and sibling repos.

---

## Drill 3 — Filename-exact search

**Goal:** Search all of GitHub for files named exactly `Makefile` that contain the target `"install"`. Limit to 5 results and print the full repository name alongside the file path.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh search code --filename=Makefile "install" --limit=5 \
  --json path,repository \
  --jq '.[] | .repository.fullName + "  " + .path'
```
</details>

**Verify:** Every `path` in the output is `Makefile` (exact match). Results come from a variety of repos.

---

## Drill 4 — Accessing `textMatches` for surrounding context

**Goal:** Find one Go file in `cli/cli` that contains `"cobra.Command"` and print the matching fragment (surrounding context) rather than just the path.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh search code --repo=cli/cli --language=go "cobra.Command" --limit=1 \
  --json path,textMatches \
  --jq '.[] | "File: " + .path + "\n" + (.textMatches[0].fragment // "(no fragment)")'
```
</details>

**Verify:** Output shows a `File:` header followed by several lines of Go source code containing `cobra.Command`. This is the same snippet you'd see highlighted in the browser.

---

## Drill 5 — The `match` flag: find files *named* something vs. files that *contain* something

**Goal:** Use `--match=path` to find files whose *path* contains `"auth"` and also contain the keyword `"token"` in the `cli/cli` repo — as opposed to files whose *contents* mention both words.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh search code --repo=cli/cli --match=path "auth" --limit=10 \
  --json path --jq '.[].path'
```

Then compare with `--match=file` (content match):

```bash
gh search code --repo=cli/cli --match=file "auth token" --limit=10 \
  --json path --jq '.[].path'
```
</details>

**Verify:** The `--match=path` query returns only files whose path contains `auth` (e.g. `pkg/cmd/auth/auth.go`). The `--match=file` query returns files whose *contents* mention `auth token`, which may live anywhere in the tree.

---

## Boss drill — Go CLI repos discovery pipeline

**Scenario:** You want to find the top Go CLI repos on GitHub and report where their `func main` entry-points live — all in one pipeline with structured JSON output at every step.

Chain:
1. `gh search repos` — find the top 5 Go repos tagged `cli`, sorted by stars.
2. `gh search code` — for each repo, locate `func main` Go files.
3. `--json --jq` — extract clean data at each step, no table parsing.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Step 1: get top 5 Go CLI repos
REPOS=$(gh search repos --language=go --topic=cli --sort=stars --order=desc \
  --limit=5 --json fullName --jq '.[].fullName')

echo -e "repo\tentry_point_path"

# Steps 2 + 3: code search per repo
for repo in $REPOS; do
  path=$(gh search code --repo="$repo" --language=go "func main" --limit=1 \
    --json path --jq '.[0].path // "(not found)"' 2>/dev/null || echo "(error)")
  echo -e "$repo\t$path"
done
```

Expected output (repos vary by current star counts):

```
repo                        entry_point_path
junegunn/fzf                main.go
jesseduffield/lazygit        main.go
wagoodman/dive               main.go
cli/cli                      cmd/gh/main.go
spf13/cobra                  main.go
```
</details>

**Verify:** Each row has a repo name and a `.go` file path separated by a tab. Repos whose entry-point is buried in a sub-directory (like `cli/cli`) show the full relative path. If you see `(error)` for any row, that repo may require the `gist` scope: run `gh auth refresh -s gist`.

**Cleanup:** Entirely read-only. No sandbox artifacts created.
