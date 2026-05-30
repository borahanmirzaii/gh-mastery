# `gh search`

> **One-liner:** Search across all of GitHub — code, commits, issues, pull requests, and repositories — from the terminal.

## When you reach for it

Any time you need to find something that spans repositories or organisations: hunt for a function signature across all public Go repos, find which projects adopted a particular dependency, track down a stale issue by keyword across your org, or wire a CI step that fails if a forbidden string appears anywhere in the codebase. Because every subcommand supports `--json` + `--jq`, results drop cleanly into scripts without screen-scraping.

## Subcommands

| Subcommand | Purpose | Node? |
|---|---|---|
| `gh search code` | Search within file contents across repos | [→ `code/`](./code/) |
| `gh search commits` | Search for commits by message, author, hash, date | inline |
| `gh search issues` | Search for issues across repos and organisations | inline |
| `gh search prs` | Search for pull requests across repos and organisations | inline |
| `gh search repos` | Search for repositories by language, topic, stars, etc. | inline |

## Key flags

Flags common to most (or all) subcommands — each subcommand also has its own dedicated qualifiers (see below).

- `--limit int` / `-L` — cap the number of results (default 30; max varies by search type). Lower this to 5 or 10 when prototyping a query.
- `--json fields` — return structured JSON for the fields you name. Without this, output is a human-readable table; with it, the result is machine-parseable.
- `--jq expression` / `-q` — pass a `jq` expression to filter or reshape the JSON inline. Pair it with `--json` whenever you're scripting.
- `--web` / `-w` — open the query in the browser instead of printing results. Handy to inspect full context of a hit.
- `--order {asc|desc}` — reverse result ordering (only respected when `--sort` is also set).
- `--sort <field>` — sort by a subcommand-specific field (e.g. `stars`, `updated`, `comments`).
- `--owner strings` — restrict to a specific owner or organisation (can repeat).
- `--repo strings` / `-R` — restrict to specific repositories (can repeat, e.g. `-R owner/a -R owner/b`).
- `--visibility {public|private|internal}` — scope to one visibility tier (where applicable).

### `gh search commits` extras

- `--author string` — filter by author login.
- `--author-date date` — relative date filter, e.g. `>2024-01-01` or `<2023-06-01`.
- `--committer string` — filter by committer login.
- `--hash string` — match a specific commit hash.
- `--merge` — include only merge commits.

### `gh search issues` / `gh search prs` extras

- `--state {open|closed}` — filter by issue/PR state.
- `--label strings` — filter by label(s).
- `--assignee string` — filter by assignee login (use `@me` for yourself).
- `--author string` — filter by opener login.
- `--milestone title` — filter by milestone title.
- `--created date`, `--updated date`, `--closed date` — date filters; prefix with `>`, `<`, or a range `2024-01-01..2024-06-01`.
- `--match strings` — restrict keyword match to `title`, `body`, or `comments` fields.
- `--include-prs` (`issues` only) — fold PR results into an issue search.

### `gh search repos` extras

- `--language string` — filter by primary language.
- `--topic strings` — filter by topic tag(s).
- `--stars number` — e.g. `>=1000`.
- `--forks number` — e.g. `>=100`.
- `--archived {true|false}` — include or exclude archived repos.
- `--license strings` — filter by SPDX license ID.
- `--good-first-issues number` — great for finding repos welcoming newcomers.

## Examples

```bash
# Find all open issues mentioning "panic" in the cli/cli repo
gh search issues panic --repo=cli/cli --state=open

# Search for merged PRs that you authored in the last month
gh search prs --author=@me --merged --created=">2024-04-01"

# Find Go repositories tagged with topic "cli" that have ≥500 stars, return JSON
gh search repos --language=go --topic=cli --stars=">=500" \
  --json fullName,stargazersCount,url \
  --jq '.[] | "\(.stargazersCount)\t\(.fullName)"' \
  | sort -rn | head -10

# Search for commits mentioning "security fix" across the github organisation
gh search commits "security fix" --owner=github --order=desc --limit=20

# Find issues labelled "good first issue" in the cli org, open only
gh search issues --owner=cli --label="good first issue" --state=open \
  --json number,title,url --jq '.[] | "\(.number)\t\(.title)"'

# Exclude results with a qualifier: find prs WITHOUT label "bug"
gh search prs -- -label:bug --state=open --limit=5
```

## Gotchas

- **Qualifier syntax in `--query` vs dedicated flags.** GitHub's raw search syntax (`language:go`, `repo:owner/name`, `user:alice`) works inside the positional query string — e.g. `gh search code "foo language:go"`. But the **dedicated flags** (`--language go`, `--repo owner/name`, `--owner alice`) are preferred: they have tab-completion, are more readable, and avoid quoting pitfalls. Most qualifiers have a corresponding flag; mixing is allowed when a qualifier has no flag equivalent (e.g. `extension:` in code search, or `is:public`).

- **`--json` + `--jq` for structured output.** Search result tables are formatted for humans and are awkward to parse. In scripts, always add `--json fields --jq 'expression'`. The `--jq` filter runs on the raw JSON array so you can reshape, reduce, or extract individual fields before they hit stdout — no external `jq` invocation needed.

- **`gh search code` requires the `gist` OAuth scope.** Code search uses a different API scope than the other search subcommands. With a fresh `gh auth login`, you get `repo` and `read:org` by default, which is enough for `commits`, `issues`, `prs`, and `repos`. But `gh search code` calls the code-search endpoint that requires the `gist` scope. Without it you'll get a 403. Fix: `gh auth refresh -s gist`. The other `search` subcommands work with default scopes.

- **Excluding qualifiers requires `--` on Unix-like systems.** GitHub search syntax uses a hyphen prefix to negate a qualifier (e.g. `-label:bug`). A bare hyphen looks like a flag to the shell argument parser and is rejected. Use `--` before the negated qualifier: `gh search issues -- "-label:bug"`. On PowerShell you also need `--%`: `gh --% search issues -- "-label:bug"`.

- **Code search is powered by a legacy engine.** The `gh search code` help warns explicitly that results are powered by what is now GitHub's *legacy* code search engine. Results may differ from `github.com`'s modern search (Blackbird engine), and newer features like regex search are not yet available via the API.

- **Search text is required for `gh search commits`.** Unlike the other subcommands, `gh search commits` rejects queries that contain only qualifiers. You must supply at least one keyword: `gh search commits "merge" --repo=owner/repo` not just `gh search commits --repo=owner/repo`.

- **Rate limits apply.** The GitHub Search API caps unauthenticated requests at 10/minute and authenticated at 30/minute. `gh search` authenticates automatically, but high-frequency scripting loops can hit the ceiling. Add `--limit` to keep result sets small.

## Concepts

None.

## Sources

- Manual: https://cli.github.com/manual/gh_search
- Local: `gh search --help` (gh 2.92.0)
