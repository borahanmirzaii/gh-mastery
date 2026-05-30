# Drills — `gh search`

> **Sandbox scope:** `gh search` is read-only across all of GitHub — no state is mutated. Run drills against the real CLI repository (`cli/cli`) or all of GitHub at large unless a drill calls for a sandbox-specific artifact. If you create an artifact specifically to search for it (e.g. an issue titled `zz-search-test-*`), create it in `borahanmirzaii/gh-mastery-sandbox` and clean it up at the end.
>
> **Namespacing:** Any sandbox artifacts you create must be prefixed `zz-search-*`.

---

## Drill 1 — Find open issues mentioning a keyword

**Goal:** Search all open issues in the `cli/cli` repository that mention the word "panic" and print their numbers and titles.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh search issues panic --repo=cli/cli --state=open --json number,title \
  --jq '.[] | "#\(.number) \(.title)"'
```
</details>

**Verify:** You should see one or more lines starting with `#<number> <title>`. If the query returns 0 results, try `"error"` instead — it's almost always present.

---

## Drill 2 — Find repositories by language and topic

**Goal:** Find the top 5 public Go repositories tagged with the topic `cli`, sorted by star count (descending), and print `stars  full-name`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh search repos --language=go --topic=cli --sort=stars --order=desc --limit=5 \
  --json fullName,stargazersCount \
  --jq '.[] | "\(.stargazersCount)\t\(.fullName)"'
```
</details>

**Verify:** Output is 5 tab-separated lines; star counts should be high (think hundreds of thousands for top results) and decrease line by line.

---

## Drill 3 — Search for commits by keyword with date filtering

**Goal:** Find the 10 most recent commits in `cli/cli` whose messages contain the word "fix", committed after 2024-01-01, and print the short SHA and first 70 characters of each commit message.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh search commits fix --repo=cli/cli --committer-date=">2024-01-01" \
  --sort=committer-date --order=desc --limit=10 \
  --json sha,commit \
  --jq '.[] | (.sha[:8]) + "  " + (.commit.message | split("\n")[0] | .[0:70])'
```
</details>

**Verify:** You should see 10 lines, each with an 8-character SHA followed by a commit subject line.

---

## Drill 4 — Exclude results with a negated qualifier

**Goal:** Search for open pull requests in `cli/cli` that do NOT have the label `"bug"` and are not drafts. Print their titles.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh search prs --repo=cli/cli --state=open -- -label:bug --json isDraft,title \
  --jq '.[] | select(.isDraft == false) | .title'
```

Alternatively, combine the qualifier negation with the `--draft` flag absence (the `-- -label:bug` is required to pass the negated qualifier past the shell):

```bash
gh search prs -- "-label:bug -is:draft" --repo=cli/cli --state=open \
  --json title --jq '.[].title'
```
</details>

**Verify:** Results appear and none of them show `bug` in their label list (you can add `--json labels` to double-check).

---

## Drill 5 — Pipe search results into a follow-up command

**Goal:** Find the 3 most-starred Go repositories with topic `cli`, then for each one print the repo name and first result from `gh search code "func main" --language=go --limit=1` in that repo. (This previews the boss drill in a simpler form.)

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh search repos --language=go --topic=cli --sort=stars --order=desc --limit=3 \
  --json fullName --jq '.[].fullName' | \
while read -r repo; do
  echo "=== $repo ==="
  gh search code --repo="$repo" --language=go "func main" --limit=1 \
    --json path --jq '.[0].path // "(no results)"'
done
```
</details>

**Verify:** You see three repo headers each followed by a file path (likely `main.go`) or `(no results)`.

---

## Boss drill — Cross-repo code archaeology pipeline

**Scenario:** You want to find actively-maintained Go CLI libraries, locate where their entry-points live, and produce a TSV report of `repo` + `entry-point path` that could feed a spreadsheet.

Chain:

1. `gh search repos` — find Go CLI repos (≥100 stars, topic `cli`).
2. `gh search code` — for each repo, find `func main` files.
3. `--json --jq` — extract clean paths throughout, no human-formatted tables.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Step 1: get the top 5 Go CLI repos by stars
REPOS=$(gh search repos --language=go --topic=cli --sort=stars --order=desc \
  --limit=5 --json fullName --jq '.[].fullName')

echo "repo	entry_point_path"

# Step 2 + 3: code search + JSON extraction per repo
for repo in $REPOS; do
  path=$(gh search code --repo="$repo" --language=go "func main" --limit=1 \
    --json path --jq '.[0].path // "(not found)"' 2>/dev/null || echo "(error)")
  echo "$repo	$path"
done
```

Example output (exact repos vary by star count at run time):

```
repo	entry_point_path
junegunn/fzf	main.go
jesseduffield/lazygit	main.go
wagoodman/dive	main.go
cli/cli	cmd/gh/main.go
spf13/cobra	main.go
```
</details>

**Verify:** The output is a TSV with a header row and 5 data rows. Each entry-point path is a `.go` file name (not an error). If `gh search code` returns a 403, you need to run `gh auth refresh -s gist` first.

**Cleanup:** This drill is entirely read-only. No sandbox artifacts were created.
