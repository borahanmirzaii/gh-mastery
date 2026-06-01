# `gh api`

> **One-liner:** Make an authenticated HTTP request to the GitHub REST or GraphQL API and print the response.

## When you reach for it

`gh api` is the **escape hatch** — the tool you pick up whenever no higher-level `gh` subcommand covers what you need. In the solo-builder loop it shows up constantly:

- **REST gaps** — creating milestones (`gh` has no `milestone` command), converting repo transfers, toggling Pages source, reading fine-grained repo metadata.
- **GraphQL mutations** — creating Discussions, editing Project v2 field options, querying across multiple resources in one round trip.
- **Scripting / automation** — anything that needs `--paginate` to walk multi-page results, `--jq` to extract a specific field, or `--input` to POST a pre-built JSON payload.

If a higher-level command exists (e.g. `gh issue create`, `gh pr merge`), prefer it — `gh api` is deliberate plumbing, not a default.

## Subcommands

`gh api` has **no subcommands**. It is a single command that takes an endpoint path as its positional argument.

The string **`graphql`** is a special endpoint value, not a subcommand:

```bash
gh api graphql -f query='{ viewer { login } }'
```

This routes to the GitHub GraphQL API (v4) rather than the REST API (v3). Everything else about the command — flags, output formatting, pagination — works identically for both.

## Key flags

### Request construction

- `-X, --method <METHOD>` — HTTP method (`GET`, `POST`, `PATCH`, `PUT`, `DELETE`). Defaults to `GET`; automatically switches to `POST` when any `-f`/`-F` field is added. Use `-X GET` to send fields as query-string parameters instead of a request body.

- `-f, --raw-field key=value` — Add a **string** parameter. The value is always sent as a JSON string — no type coercion. Use for text fields where you're certain the value is a string (e.g. `-f title='My Issue'`, `-f body='Hello'`). In GraphQL, all fields except `query` and `operationName` become variables; `-f` always injects them as strings regardless of the schema type.

- `-F, --field key=value` — Add a **typed** parameter. Performs magic type conversion:
  - `true` / `false` / `null` → JSON booleans / null
  - Integer strings → JSON numbers
  - `@path` → reads file contents and sends as the value (`@-` reads stdin)
  - `{owner}` / `{repo}` / `{branch}` → substituted from current repo context
  
  **Use `-F` for GraphQL variables** that are not strings (booleans, integers, cursor values). Mixing up `-f` and `-F` is the single most common `gh api` mistake.

- `-H, --header key:value` — Add an HTTP request header. Common use: setting `Accept` for API previews or raw content (`-H 'Accept: application/vnd.github.v3.raw+json'`).

- `--input <file>` — Read the request body from a file (or `-` for stdin). When used, any `-f`/`-F` fields are appended to the **query string** instead. Useful for posting pre-built JSON payloads.

- `-p, --preview <name>` — Opt into a GitHub API preview (experimental endpoints). Sends the appropriate `Accept` header automatically. Multiple previews: `-p corsair,scarlet-witch`.

### Pagination

- `--paginate` — Follow all pagination links and emit every page sequentially. Without this flag, you silently receive only the first page (typically 30 items for issues/PRs, 100 for some endpoints). Each page is emitted as a separate JSON object/array.

- `--slurp` — Use with `--paginate`: wraps all pages into a single outer JSON array, so downstream `jq` sees one document instead of a stream. Essential for aggregate operations (e.g. computing percentages across all pages). **Note:** `--slurp` and `--jq` are mutually exclusive — use `--slurp` then pipe to a separate `jq` invocation instead.

  For **GraphQL** pagination, `--paginate` requires the query to accept `$endCursor: String` and return `pageInfo { hasNextPage endCursor }` on the collection. `gh api` injects the cursor automatically between pages.

### Output formatting

- `-q, --jq <expr>` — Filter the JSON response with a jq expression inline — no need to pipe to a separate `jq` process. The `jq` binary does not need to be installed. Runs after each page when combined with `--paginate`.

- `-t, --template <string>` — Format JSON output using a Go template. Supports custom helpers: `color`, `join`, `pluck`, `tablerow`/`tablerender`, `timeago`, `timefmt`, `truncate`, `hyperlink`. See `gh help formatting` for the full list.

- `--silent` — Suppress the response body (useful when you care only about the exit code or side-effect).

- `-i, --include` — Include HTTP status line and response headers in output. Useful for debugging unexpected responses.

- `--verbose` — Full HTTP request + response dump. The heaviest debugging mode.

### Targeting

- `--hostname <host>` — Direct the request to a GitHub Enterprise Server instance instead of `github.com`. Also settable via `GH_HOST` environment variable.

- `--cache <duration>` — Cache the response for the given duration (e.g. `"3600s"`, `"60m"`, `"1h"`). Useful in scripts that call the same read endpoint repeatedly. Cache is local to the machine.

### Placeholder substitution

Endpoint paths (and `-F` values) may contain `{owner}`, `{repo}`, and `{branch}` — these are **auto-substituted** from the current directory's repo context or from `GH_REPO`. This makes scripts portable across repos without hardcoding org/repo names:

```bash
# Works in any repo checkout — no hardcoding needed
gh api repos/{owner}/{repo}/releases --jq '.[0].tag_name'
```

Override with `GH_REPO=owner/repo gh api repos/{owner}/{repo}/...` to target a different repo without changing directory.

## Examples

### 1 — REST GET: list releases in the current repo

```bash
# Print the tag name of the latest release
gh api repos/{owner}/{repo}/releases --jq '.[0].tag_name'
```

### 2 — REST GET with `--paginate` + `--jq`: collect all open issue numbers

```bash
# Fetch every page of open issues and extract their numbers
# --jq runs per page, so each page's numbers are printed in sequence
gh api repos/borahanmirzaii/gh-mastery/issues \
  -X GET \
  -f state=open \
  --paginate \
  --jq '.[].number'
```

Without `--paginate`, only the first 30 issues would be returned (GitHub's default page size for issues).

To get the total count across all pages, use `--slurp` and pipe to `jq` separately (`--slurp` and `--jq` cannot be combined in gh 2.92.0):

```bash
gh api repos/borahanmirzaii/gh-mastery/issues \
  -X GET -f state=open \
  --paginate --slurp | jq 'map(.[]) | length'
```

### 3 — REST POST with `-f`: create a milestone (no `gh milestone` command exists)

```bash
gh api repos/borahanmirzaii/gh-mastery/milestones \
  -f title="M1 — Core (daily drivers)" \
  -f description="Core command groups" \
  --jq '.number'
```

This was used to create the actual milestones in this repo — a worked example of reaching for `gh api` when a higher-level command doesn't exist.

### 4 — GraphQL query with `-F` typed variables

```bash
# Look up the node ID for a repository (needed for Project v2 mutations)
gh api graphql \
  -F owner='{owner}' \
  -F name='{repo}' \
  -f query='
    query($owner: String!, $name: String!) {
      repository(owner: $owner, name: $name) {
        id
        nameWithOwner
      }
    }
  ' \
  --jq '.data.repository.id'
```

Note: `owner` and `name` use `-F` (typed) so placeholder substitution works — `-f` would send the literal string `{owner}` instead of resolving it.

### 5 — GraphQL mutation: add a Discussion comment (the RFC #1 approach used in this repo)

```bash
# Add a comment to Discussion #1 — this is how the decision-record was posted
DISCUSSION_ID=$(gh api graphql \
  -F owner=borahanmirzaii \
  -F name=gh-mastery \
  -f query='
    query($owner: String!, $name: String!) {
      repository(owner: $owner, name: $name) {
        discussion(number: 1) { id }
      }
    }
  ' --jq '.data.repository.discussion.id')

gh api graphql \
  -F subjectId="$DISCUSSION_ID" \
  -f body="Decision record: adopt scaffold-first build model." \
  -f query='
    mutation($subjectId: ID!, $body: String!) {
      addDiscussionComment(input: {subjectId: $subjectId, body: $body}) {
        comment { url }
      }
    }
  ' --jq '.data.addDiscussionComment.comment.url'
```

### 6 — REST PATCH + `--input`: bulk-update via pre-built JSON

```bash
# Update repository settings from a JSON file
gh api -X PATCH repos/{owner}/{repo} --input settings.json --jq '.full_name'
```

## Gotchas

### 1. `-f` (string) vs `-F` (typed) — the most common mistake

`-f` **always** sends the value as a JSON string. `-F` performs type coercion: booleans, numbers, null, placeholder substitution (`{owner}` etc.), and `@file` reads.

```bash
# WRONG — sends the string "true", not the boolean true
gh api graphql -f query='...' -f isPrivate=true

# CORRECT — sends boolean true
gh api graphql -f query='...' -F isPrivate=true
```

For GraphQL variables, use `-F` for anything that isn't a plain string. Using `-f` where the schema expects a boolean or integer silently sends the wrong type, and the API may accept it or return a cryptic type error.

Also: `-F owner='{owner}'` resolves the placeholder; `-f owner='{owner}'` sends the literal four-character string `{owner}`.

### 2. `--paginate` is opt-in — you silently miss results without it

GitHub paginates most list endpoints (default 30 items; max 100 with `?per_page=100`). `gh api` without `--paginate` returns only the first page. There is no warning. Scripts that count items or iterate all results **must** use `--paginate` or explicitly set `per_page` and check for truncation.

For GraphQL `--paginate`, the query must declare `$endCursor: String` and return `pageInfo { hasNextPage endCursor }` on the paginated collection — otherwise `gh api` cannot walk the cursor and will only fetch the first page.

### 3. `{owner}/{repo}` placeholders require a repo context

Placeholders (`{owner}`, `{repo}`, `{branch}`) are resolved from the current directory's git remote. If you run `gh api repos/{owner}/{repo}/issues` outside a git repo, the command fails with a "no repository found" error. Override with `GH_REPO=owner/repo` or `--repo` (not a flag on `gh api` itself — set `GH_REPO` in the environment).

### 4. `--jq` runs per page; `--slurp` and `--jq` are mutually exclusive

When `--paginate` is active, `--jq` is applied to **each page independently**. If your jq expression expects a single document (e.g. `length` over all items) you have two options:

- Use `--paginate --slurp` (without `--jq`) and pipe to a separate `jq` invocation — `--slurp` and `--jq` cannot be combined in the same command (gh 2.92.0 returns an error).
- Use `--paginate --jq 'length'` and sum the per-page counts externally.

```bash
# Works: slurp then external jq
gh api repos/o/r/issues --paginate --slurp | jq 'map(.[]) | length'

# Fails: --slurp + --jq together
gh api repos/o/r/issues --paginate --slurp --jq 'length'
# → "the `--slurp` option is not supported with `--jq` or `--template`"
```

### 5. `graphql` endpoint — for everything REST can't do

Discussions (create, comment, react), Project v2 mutations (add items, set field values, update Status options), fine-grained cross-resource queries — none of these have REST endpoints. `gh api graphql` is the only path. The RFC #1 decision-record comment in this very repo was posted via `gh api graphql addDiscussionComment` — a lived example, not a hypothetical.

REST vs GraphQL rule of thumb: start with the higher-level `gh` command; fall back to REST `gh api`; fall back to `gh api graphql` when REST can't do it.

### 6. Adding `-f`/`-F` fields silently switches the method to POST

If you add parameters intending a GET (e.g. query-string filters), `gh api` will POST them in the body instead. Use `-X GET` explicitly to force query-string behavior:

```bash
# Sends state=open as a query param, not a POST body
gh api -X GET repos/{owner}/{repo}/issues -f state=open
```

### 7. `--cache` is local and not invalidated automatically

Cached responses persist for the specified duration regardless of upstream changes. Don't use `--cache` for data that must be fresh (e.g. issue status, workflow run state). Safe for stable metadata (repo node IDs, label lists).

## Concepts

- [REST vs GraphQL](../../concepts/rest-vs-graphql.md) _(to be written)_ — when to reach for the REST API vs the GraphQL API, the shape of each response, and how `gh api` abstracts the transport layer.

## Sources

- Manual: https://cli.github.com/manual/gh_api
- Local: `gh api --help` (gh 2.92.0)
- Formatting reference: `gh help formatting` (gh 2.92.0)
- Exit codes: `gh help exit-codes` (gh 2.92.0)
