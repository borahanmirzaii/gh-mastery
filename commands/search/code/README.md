# `gh search code`

> **One-liner:** Search within file contents across all of GitHub (or a specific repo/owner) and return matching code snippets with their file paths and URLs.

## When you reach for it

- **Cross-repo archaeology** — finding every repo that uses a particular function, constant, or API call, without cloning anything.
- **CI-driven "search-as-test"** — failing a build if a forbidden string (e.g. a hard-coded credential pattern, a deprecated API name) appears anywhere in the codebase.
- **Dependency / migration audits** — discovering which repos in an org still import an old package path.
- **Onboarding** — locating example usages of an internal SDK function scattered across multiple repos.

## Key flags

`gh search code` has no subcommands — it is a leaf command. All filtering is done with flags or inline qualifiers.

- `--repo strings` / `-R` — restrict to one or more repositories (`owner/name`). Repeat for multiple: `-R cli/cli -R cli/go-gh`.
- `--owner strings` — restrict to an owner or organisation (e.g. `--owner=github`). Expands to all repos of that owner.
- `--language string` — filter by programming language (e.g. `--language=go`, `--language=python`). Corresponds to the `language:` qualifier.
- `--extension string` — filter by file extension (e.g. `--extension=yml`). Corresponds to the `extension:` qualifier.
- `--filename string` — filter by exact filename (e.g. `--filename=Makefile`). Corresponds to the `filename:` qualifier.
- `--path string` — filter by path fragment (e.g. `--path=cmd/`). Corresponds to the `path:` qualifier; matches anywhere in the path.
- `--match strings` — restrict keyword matching to `file` (file contents) or `path` (file path). Default matches both. Useful to find files *named* something vs. files that *contain* something.
- `--size string` — filter on file size in kilobytes, e.g. `--size="<10"` (files under 10 KB). Useful to skip auto-generated mega-files.
- `--limit int` / `-L` — cap results (default 30). The API returns at most 100 per call.
- `--json fields` — return JSON. Available fields: `path`, `repository`, `sha`, `textMatches`, `url`.
- `--jq expression` / `-q` — filter/transform the JSON inline. Always pair with `--json` when scripting.
- `--web` / `-w` — open the query in the browser instead of printing results.

## Examples

```bash
# Find every file in cli/cli that contains "func main" and is written in Go
gh search code --repo=cli/cli --language=go "func main"

# Search for "api_key" across all repos of the acme organisation
gh search code --owner=acme "api_key"

# Locate all YAML files named exactly "release.yml" containing "on: push"
gh search code --filename=release.yml --extension=yml "on: push"

# Extract just the file paths from a JSON response (scripting pattern)
gh search code --repo=cli/cli "RunE" --language=go --limit=10 \
  --json path,url \
  --jq '.[] | .path'

# Find files in the cmd/ sub-tree of a repo that import a specific package
gh search code --repo=myorg/myservice --language=go --path=cmd/ \
  "\"github.com/myorg/myservice/internal/config\""

# Check for hard-coded AWS keys anywhere in an org (CI guard pattern)
gh search code --owner=myorg "AKIA" --limit=1 \
  --json url --jq '
    if length > 0
    then "FAIL: potential AWS key found:\n" + .[0].url
    else "OK: no matches"
    end'
```

## Gotchas

- **`gh search code` requires the `gist` OAuth scope.** A fresh `gh auth login` grants `repo` + `read:org` by default — enough for `gh search issues/prs/repos/commits` but *not* for code search. Without the `gist` scope you get a 403 immediately. Fix: `gh auth refresh -s gist`. You only need to do this once per account.

- **Qualifier syntax vs dedicated flags.** The inline qualifier syntax works — e.g. `gh search code "foo language:go"` — but the dedicated flags (`--language go`, `--extension yml`, `--repo owner/name`, `--owner org`, `--path dir/`, `--filename file`) are preferred for clarity and tab-completion. Mixing is allowed when a qualifier has no dedicated flag counterpart (e.g. `is:fork`, `is:public`, `size:` with complex ranges). Always prefer a flag when one exists.

- **Results are powered by a legacy search engine.** The GitHub CLI docs warn that `gh search code` calls what is now GitHub's *legacy* code search API (not the modern Blackbird engine used on `github.com`). Regex search, symbol search, and other modern features are unavailable. Results may differ from what you see in the browser. Open with `--web` to compare.

- **`--limit` caps at 100, not unlimited.** GitHub's code search API returns at most 100 results per request, and `gh search code` does not paginate (unlike `gh api --paginate`). If you need more, narrow your query with additional flags.

- **`textMatches` field shows surrounding context.** When you add `--json textMatches`, each result includes an array of match objects with `fragment` (the surrounding lines), `matches` (exact offsets), and `objectType`/`property`. This is the programmatic equivalent of the highlighted snippet shown in the browser.

- **`extension:` vs `path:` are different axes.** `--extension=yml` filters on the file suffix; `--path=.github/workflows` filters on the directory path. They are independent: a file can match on extension only, path only, or both. Combine them for precision.

## Concepts

None.

## Sources

- Manual: https://cli.github.com/manual/gh_search_code
- Local: `gh search code --help` (gh 2.92.0)
