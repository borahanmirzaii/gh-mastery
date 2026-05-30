# Recall — `gh search`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** `gh search code "foo language:go"` works — so why should you prefer `gh search code foo --language=go` instead?

<details><summary>Answer</summary>

The dedicated `--language` flag (and its siblings: `--repo`, `--owner`, `--extension`, `--path`, `--filename`) has tab-completion, is more readable in scripts and shell history, and avoids quoting pitfalls when mixing with other arguments. Qualifier syntax inside the positional query string (`language:go`) works but is a fallback for qualifiers that have no dedicated flag (e.g. `is:public`, `extension:yml`). Mixing is allowed.

</details>

---

**Q2.** You run `gh search code "api_key" --repo=myorg/backend` and get a 403 error. What is the likely cause and how do you fix it?

<details><summary>Answer</summary>

`gh search code` requires the `gist` OAuth scope, which is NOT included in the default scopes granted by `gh auth login`. The other `search` subcommands (`commits`, `issues`, `prs`, `repos`) work with default scopes. Fix:

```bash
gh auth refresh -s gist
```

Then re-run your search. The extra scope is persistent; you only need to do this once per account.

</details>

---

**Q3.** You want to print just the full names of the top 10 most-starred Python repos on GitHub. Write the full command.

<details><summary>Answer</summary>

```bash
gh search repos --language=python --sort=stars --order=desc --limit=10 \
  --json fullName --jq '.[].fullName'
```

Key points: `--json fullName` limits the payload to one field; `--jq '.[].fullName'` iterates the array and extracts each name. Without `--json` + `--jq` you'd get a human-formatted table that's awkward to pipe.

</details>

---

**Q4.** How do you search for issues that do NOT have the label `"needs-triage"` using `gh search issues`?

<details><summary>Answer</summary>

Negated qualifiers start with `-` and look like a flag to the shell, so you must use `--` to end flag parsing first:

```bash
gh search issues -- "-label:needs-triage" --state=open
```

On PowerShell, also prepend `--%`:

```powershell
gh --% search issues -- "-label:needs-triage" --state=open
```

</details>

---

**Q5.** `gh search commits --repo=cli/cli` fails with "Search text is required". Why, and how do you fix it?

<details><summary>Answer</summary>

Unlike `gh search issues` or `gh search repos`, `gh search commits` requires at least one keyword in the positional query — a qualifiers-only query is rejected by the GitHub Search API. Fix: add a keyword, even a broad one:

```bash
gh search commits "." --repo=cli/cli --limit=10
# or a real keyword
gh search commits "fix" --repo=cli/cli
```

</details>

---

**Q6.** What is the difference between `extension:yml` and `path:` in a `gh search code` query, and which one has a dedicated flag?

<details><summary>Answer</summary>

- `extension:yml` (or `--extension yml`) — matches files whose **file extension** is `.yml`, regardless of path. Use when you know the file type but not its location. This has a dedicated flag: `--extension`.
- `path:` (e.g. `path:.github/workflows`) — matches files whose **full path** contains the string. Use when you know where the file lives in the directory tree. This also has a dedicated flag: `--match path` (restricts keyword matching to the file path field) but the `path:` qualifier inline is more specific.

For example, to find all YAML GitHub Actions workflow files:

```bash
gh search code "on: push" --extension=yml --match=path --repo=cli/cli
```

</details>

---

**Q7.** How would you search for all public issues assigned to you across ALL of GitHub (not just one repo), and print a count?

<details><summary>Answer</summary>

```bash
gh search issues --assignee=@me --state=open --visibility=public \
  --json number --jq 'length'
```

`gh search issues` defaults to searching all of GitHub when `--repo` and `--owner` are omitted. `--assignee=@me` uses the token's identity. `--jq 'length'` counts the JSON array elements returned.

</details>

---

**Q8.** What scope change is required before `gh search code` works, and what exact command applies it?

<details><summary>Answer</summary>

The `gist` scope must be added to your token. The command is:

```bash
gh auth refresh -s gist
```

This is a one-time step per GitHub account. The other `gh search` subcommands (`commits`, `issues`, `prs`, `repos`) do not need it.

</details>
