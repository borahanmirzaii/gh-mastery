# Recall — `gh api`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** What is the difference between `-f key=value` and `-F key=value`? When does it matter most?

<details><summary>Answer</summary>

`-f` (`--raw-field`) **always** sends the value as a JSON string — no type coercion.

`-F` (`--field`) performs magic type conversion:
- `true`/`false`/`null` → JSON booleans/null
- Integer strings → JSON numbers
- `@path` → reads the file and sends its contents as the value
- `{owner}`, `{repo}`, `{branch}` → resolved from the current repo context

**Matters most for GraphQL variables.** If a variable has type `Boolean!` or `Int!` and you pass it with `-f`, the API receives a string — which may cause a cryptic type error or silently wrong behaviour. Use `-F` for any non-string GraphQL variable.

Also critical for placeholder substitution: `-F owner='{owner}'` resolves to the actual owner; `-f owner='{owner}'` sends the four-character literal `{owner}`.

</details>

---

**Q2.** You run `gh api repos/myorg/myrepo/issues --jq 'length'` and get `30` even though the repo has 200 open issues. What's wrong and how do you fix it?

<details><summary>Answer</summary>

`gh api` returns only the **first page** (30 items by default) unless you pass `--paginate`. There is no warning that results were truncated.

Fix — add `--paginate` and `--slurp` to merge all pages into one document, then apply `--jq`:

```bash
gh api repos/myorg/myrepo/issues \
  -X GET -f state=open \
  --paginate --slurp \
  --jq 'map(.[]) | length'
```

Or without `--slurp`, sum the per-page lengths:

```bash
gh api repos/myorg/myrepo/issues -X GET -f state=open \
  --paginate --jq 'length' | paste -sd+ - | bc
```

</details>

---

**Q3.** What does `gh api graphql` do, and what kinds of operations require it that the REST API can't handle?

<details><summary>Answer</summary>

`graphql` is a **special endpoint value** (not a subcommand) that routes the request to the GitHub GraphQL API (v4) instead of the REST API (v3):

```bash
gh api graphql -f query='{ viewer { login } }'
```

Operations REST can't do, requiring GraphQL:
- **GitHub Discussions** — create, comment, react (no REST endpoints for creation)
- **Project v2 mutations** — adding items, editing field values, updating Status options (`updateProjectV2Field`)
- **Cross-resource queries** — fetching a PR's reviews + assignees + labels in one round trip
- **Node ID lookups** — REST gives you numeric IDs; GraphQL uses global node IDs (e.g. `R_kg...`) needed for subsequent mutations

The `-f`/`-F` distinction is especially important here: all fields except `query` and `operationName` are interpreted as GraphQL variables, so types must match the schema.

</details>

---

**Q4.** How do `{owner}`, `{repo}`, and `{branch}` placeholders work in `gh api`, and what happens if you run the command outside a git repo?

<details><summary>Answer</summary>

These placeholders are **auto-substituted** from the current directory's git remote context (or the `GH_REPO` environment variable). They work in both the endpoint path and in `-F` field values:

```bash
# Resolved at runtime from the local repo's remote
gh api repos/{owner}/{repo}/releases --jq '.[0].tag_name'

# -F resolves placeholders; -f sends them literally
gh api graphql -F owner='{owner}' -F name='{repo}' -f query='...'
```

**Outside a git repo:** the command fails with a "no repository found" or "could not determine GitHub repository" error. Fix: set `GH_REPO=owner/repo` in the environment before running, or hardcode the owner/repo in the endpoint path.

</details>

---

**Q5.** How does `--jq` behave when combined with `--paginate`? What flag do you add to change that behaviour?

<details><summary>Answer</summary>

With `--paginate`, `gh api` emits each page as a separate JSON document and runs `--jq` against **each page independently**. This is useful when you want per-page filtering, but it breaks aggregate expressions like `length` (which would return the count per page, not the total).

To treat all pages as a single document, use `--slurp` — but note that `--slurp` and `--jq` are **mutually exclusive** in gh 2.92.0 (combining them returns an error). Use `--slurp` and pipe to a separate `jq` invocation:

```bash
# length per page — runs jq against each page independently
gh api repos/o/r/issues --paginate --jq 'length'

# total count: slurp all pages, then pipe to jq (cannot combine --slurp + --jq)
gh api repos/o/r/issues --paginate --slurp | jq 'map(.[]) | length'
```

</details>

---

**Q6.** You want to add fields to a GET request as query-string parameters, not as a POST body. How do you do it?

<details><summary>Answer</summary>

By default, adding any `-f`/`-F` field **switches the HTTP method to POST** and puts the fields in the request body. To keep them as query-string parameters, explicitly force the method back to GET:

```bash
gh api -X GET repos/{owner}/{repo}/issues \
  -f state=open \
  -f per_page=100
```

Without `-X GET`, this would POST `{state: "open", per_page: "100"}` as a body — which is almost certainly not what the issues endpoint expects.

</details>

---

**Q7.** What flag gives you full HTTP request/response debugging output, and what's the lighter option that only shows status + headers?

<details><summary>Answer</summary>

- `--verbose` — prints the complete HTTP request and response, including headers and body. The heaviest debugging mode.
- `-i / --include` — prints only the HTTP status line and response headers before the body. Lighter; useful when you just want to confirm the status code or inspect a specific header without the full request dump.

```bash
# Just the response headers + status
gh api repos/{owner}/{repo} -i 2>&1 | head -10

# Full request+response dump
gh api repos/{owner}/{repo} --verbose
```

</details>

---

**Q8.** For a GraphQL `--paginate` query, what must the query declare for `gh api` to walk the cursor automatically?

<details><summary>Answer</summary>

The query must:
1. Accept `$endCursor: String` as a variable.
2. Pass it as the `after:` argument on the paginated collection.
3. Return `pageInfo { hasNextPage endCursor }` on that collection.

`gh api` reads `pageInfo.hasNextPage` to decide whether to continue and injects the new `endCursor` value as `$endCursor` on subsequent requests.

Minimal example:

```graphql
query($endCursor: String) {
  viewer {
    repositories(first: 100, after: $endCursor) {
      nodes { nameWithOwner }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
}
```

If `pageInfo` is missing, `gh api --paginate` only fetches the first page — silently, with no error.

</details>
