# Recall — `gh search code`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** You run `gh search code "api_key" --owner=myorg` and get a 403. What is the cause and the fix?

<details><summary>Answer</summary>

`gh search code` requires the `gist` OAuth scope, which is NOT granted by a default `gh auth login`. The other `gh search` subcommands work with default scopes, but code search calls a different API endpoint.

Fix:

```bash
gh auth refresh -s gist
```

You only need to do this once per account. The scope addition is persistent in your stored token.

</details>

---

**Q2.** What is the difference between `gh search code "foo language:go"` and `gh search code foo --language=go`? Which is preferred and why?

<details><summary>Answer</summary>

Both forms produce the same search query on GitHub's backend. However, the dedicated flag form (`--language=go`) is preferred because:

1. It supports tab-completion in shells.
2. It is unambiguous — no quoting or spacing errors when combined with other flags.
3. It is easier to read and audit in scripts.

The inline qualifier form (`language:go` inside the query string) is a fallback for qualifiers that have no dedicated flag, such as `is:fork`, `is:public`, or complex `size:` expressions. Mixing both in the same command is allowed.

</details>

---

**Q3.** How do you use `--json` and `--jq` together to extract only the file paths from a `gh search code` result?

<details><summary>Answer</summary>

```bash
gh search code --repo=cli/cli "func main" --language=go \
  --json path \
  --jq '.[].path'
```

`--json path` limits the JSON payload to just the `path` field per result. `--jq '.[].path'` iterates the array and prints each path on its own line. Without `--json`, output is a human-formatted table; without `--jq`, you get the full JSON array.

Available JSON fields: `path`, `repository`, `sha`, `textMatches`, `url`.

</details>

---

**Q4.** What does `--match=path` do, and how does it differ from `--match=file`?

<details><summary>Answer</summary>

The `--match` flag tells GitHub's search engine which field(s) to match your keyword against:

- `--match=file` — match keyword against the **file contents**. A file is returned if the keyword appears in its source text.
- `--match=path` — match keyword against the **file path** (directory + filename). A file is returned if the keyword appears in its path string.

Default (no `--match`) searches both. Example:

```bash
# Only return files whose path contains "auth"
gh search code --repo=cli/cli --match=path "auth"

# Only return files whose contents mention "auth"
gh search code --repo=cli/cli --match=file "auth"
```

</details>

---

**Q5.** How do you search for files named exactly `Dockerfile` across all of GitHub and print the repository and path for each?

<details><summary>Answer</summary>

```bash
gh search code --filename=Dockerfile "" \
  --json path,repository \
  --jq '.[] | .repository.fullName + ": " + .path'
```

`--filename=Dockerfile` is an exact filename match (the `filename:` qualifier). It's independent of `--extension` (which matches the suffix). Note: an empty string `""` is needed as the positional query if you use only flags — alternatively, use a broad keyword like `FROM` which almost every Dockerfile contains.

A more practical version:

```bash
gh search code --filename=Dockerfile "FROM" --limit=10 \
  --json path,repository \
  --jq '.[] | .repository.fullName + ": " + .path'
```

</details>

---

**Q6.** `gh search code` is described as using a "legacy" engine. What does that mean in practice?

<details><summary>Answer</summary>

The `gh search code` command calls the GitHub Code Search REST API (`/search/code`), which is powered by GitHub's original (legacy) search engine, not the modern "Blackbird" engine introduced in 2023 and used on `github.com`. In practice this means:

- **No regex search** — pattern matching is not available via the API.
- **No symbol search** — you cannot search for function definitions, class names, etc. as semantic symbols.
- **Results may differ** from what you see when searching on `github.com` in the browser.
- **`--web` opens the modern engine** — use `-w` to flip over to the browser view with the Blackbird results for comparison.

This is a documented limitation in `gh search code --help` itself.

</details>

---

**Q7.** How do you write a CI guard that fails if a string `"HARDCODED_SECRET"` appears anywhere in an organisation's repos?

<details><summary>Answer</summary>

```bash
result=$(gh search code --owner=myorg "HARDCODED_SECRET" --limit=1 \
  --json url --jq 'length')

if [ "$result" -gt 0 ]; then
  echo "FAIL: forbidden string found" >&2
  exit 1
fi
echo "OK"
```

Key points: `--limit=1` keeps the query fast — you only care whether at least one match exists, not the full list. `--jq 'length'` returns `0` or `1` without any text formatting. Exit code 1 causes the CI step to fail.

</details>
