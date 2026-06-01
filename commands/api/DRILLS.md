# Drills — `gh api`

> **Sandbox:** run everything against `borahanmirzaii/gh-mastery-sandbox`
> (add `--repo borahanmirzaii/gh-mastery-sandbox` or set `GH_REPO=borahanmirzaii/gh-mastery-sandbox`) unless a drill says otherwise.
> **Namespacing:** create only objects prefixed `zz-api-*` and delete them at the end, so parallel drills never collide.

---

## Drill 1 — REST GET with placeholder substitution and `--jq`

**Goal:** Fetch the sandbox repo's metadata and extract just its default branch name using `--jq` inline (no separate `jq` pipe).

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh api repos/borahanmirzaii/gh-mastery-sandbox --jq '.default_branch'
```

Or using placeholders (run from inside a clone of the sandbox):

```bash
gh api repos/{owner}/{repo} --jq '.default_branch'
```

</details>

**Verify:** Output is a single branch name (e.g. `main`). No extra JSON noise — only the value extracted by `--jq`.

---

## Drill 2 — REST GET with `--paginate` vs without

**Goal:** Understand the pagination gap. First fetch issues from `cli/cli` without `--paginate`, then with it, and compare the counts.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Without pagination — first page only (30 items by default)
gh api repos/cli/cli/issues -X GET -f state=open --jq 'length'

# With pagination — slurp all pages into one array, then pipe to jq for total count
# Note: --slurp and --jq cannot be combined (gh 2.92.0); use a separate jq pipe
gh api repos/cli/cli/issues -X GET -f state=open --paginate --slurp | jq 'map(.[]) | length'
```

Alternative without `--slurp` — sum per-page lengths:

```bash
gh api repos/cli/cli/issues -X GET -f state=open \
  --paginate --jq 'length' | awk '{s+=$1} END{print s}'
```

</details>

**Verify:** The paginated count is larger than (or equal to) the single-page count. If the repo has >30 open issues, the difference is stark.

---

## Drill 3 — REST POST with `-f`: create a label via the API, then clean up

**Goal:** Use `gh api` with `-f` string fields to create a label named `zz-api-test` in the sandbox, verify it exists, then delete it.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Create the label
gh api repos/borahanmirzaii/gh-mastery-sandbox/labels \
  -f name="zz-api-test" \
  -f color="ff0000" \
  -f description="Temporary drill label — safe to delete" \
  --jq '.name + " created (color: #" + .color + ")"'

# Verify it exists
gh api repos/borahanmirzaii/gh-mastery-sandbox/labels/zz-api-test \
  --jq '.name'

# Delete it
gh api -X DELETE repos/borahanmirzaii/gh-mastery-sandbox/labels/zz-api-test
```

</details>

**Verify:**
- Create step prints `zz-api-test created (color: #ff0000)`.
- Verify step prints `zz-api-test`.
- Delete step returns no output (HTTP 204).
- Confirm cleanup: `gh api repos/borahanmirzaii/gh-mastery-sandbox/labels/zz-api-test` returns a 404 after deletion.

---

## Drill 4 — GraphQL query with `-F` typed variables

**Goal:** Use `gh api graphql` with `-F` to look up the repository node ID of the sandbox, then verify that using `-f` (instead of `-F`) breaks placeholder substitution.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# CORRECT: -F resolves {owner} and {repo} placeholders
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
  --jq '.data.repository.nameWithOwner' \
  -- <<< "" 2>&1 || true

# Run this from inside a clone of the sandbox, or set GH_REPO:
GH_REPO=borahanmirzaii/gh-mastery-sandbox \
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
    --jq '.data.repository.nameWithOwner'
```

Now try with `-f` for owner/name (should fail or return null because `{owner}` is sent as a literal string):

```bash
GH_REPO=borahanmirzaii/gh-mastery-sandbox \
  gh api graphql \
    -f owner='{owner}' \
    -f name='{repo}' \
    -f query='
      query($owner: String!, $name: String!) {
        repository(owner: $owner, name: $name) {
          id
          nameWithOwner
        }
      }
    ' \
    --jq '.data.repository.nameWithOwner'
# Expect: null (the API looks for a repo named literally "{repo}" which doesn't exist)
```

</details>

**Verify:**
- `-F` version prints `borahanmirzaii/gh-mastery-sandbox`.
- `-f` version prints `null` — confirming that `-f` does not resolve placeholders.

---

## Drill 5 — `--jq` vs `--template` for custom output

**Goal:** List the 3 most recently updated open issues in the sandbox using first `--jq` and then `--template` to compare the two formatting approaches.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Using --jq
gh api repos/borahanmirzaii/gh-mastery-sandbox/issues \
  -X GET -f state=open -f per_page=3 \
  --jq '.[] | "#\(.number) \(.title)"'

# Using --template with tablerow for aligned output
gh api repos/borahanmirzaii/gh-mastery-sandbox/issues \
  -X GET -f state=open -f per_page=3 \
  --template '{{range .}}{{tablerow (printf "#%v" .number) .title (timeago .updated_at)}}{{end}}'
```

</details>

**Verify:** Both commands print the same issues in different formats. `--template` output is tabularly aligned; `--jq` output is one line per issue.

---

## Boss drill — Chain REST pagination → GraphQL variable injection → cleanup

**Goal:** A realistic multi-step workflow that exercises the full `gh api` toolkit:

1. Use REST + `--paginate` + `--slurp` to count all labels in the sandbox.
2. Create a temporary label `zz-api-boss` via REST POST.
3. Use `gh api graphql` with `-F` typed variable to fetch the sandbox's `id` (node ID) — simulating the lookup you'd do before a Project v2 mutation.
4. Delete the `zz-api-boss` label to clean up.
5. Confirm the label is gone.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Step 1: count labels (paginate to be safe)
# Note: --slurp and --jq cannot be combined; pipe to jq separately
echo "=== Label count before ==="
gh api repos/borahanmirzaii/gh-mastery-sandbox/labels \
  --paginate --slurp | jq 'map(.[]) | length'

# Step 2: create the drill label
echo "=== Creating zz-api-boss ==="
gh api repos/borahanmirzaii/gh-mastery-sandbox/labels \
  -f name="zz-api-boss" \
  -f color="0075ca" \
  -f description="Boss drill label — delete me" \
  --jq '.name + " created"'

# Step 3: GraphQL — fetch sandbox node ID
echo "=== Sandbox node ID (via GraphQL) ==="
gh api graphql \
  -F owner=borahanmirzaii \
  -F name=gh-mastery-sandbox \
  -f query='
    query($owner: String!, $name: String!) {
      repository(owner: $owner, name: $name) {
        id
        nameWithOwner
      }
    }
  ' \
  --jq '"nodeId: " + .data.repository.id'

# Step 4: delete the drill label
echo "=== Cleaning up zz-api-boss ==="
gh api -X DELETE repos/borahanmirzaii/gh-mastery-sandbox/labels/zz-api-boss
echo "deleted"

# Step 5: verify gone (expect 404 error output)
echo "=== Confirming deletion ==="
gh api repos/borahanmirzaii/gh-mastery-sandbox/labels/zz-api-boss \
  --jq '.message' 2>&1 || echo "label not found (expected)"
```

</details>

**Verify:**
- Step 1 prints a number.
- Step 2 prints `zz-api-boss created`.
- Step 3 prints a `R_kg...` node ID.
- Step 4 prints `deleted`.
- Step 5 prints `Not Found` or `label not found (expected)`.

**Cleanup:** No `zz-api-*` labels should remain after Step 4.
Confirm with: `gh api repos/borahanmirzaii/gh-mastery-sandbox/labels --paginate --slurp | jq 'map(.[]) | map(select(.name | startswith("zz-api-"))) | length'` → expect `0`.
