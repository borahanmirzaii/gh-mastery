# Recall — `gh variable`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** What is the fundamental difference between `gh variable` and `gh secret` — and how does that determine which one you choose?

<details><summary>Answer</summary>

Variables are **not encrypted** — their values are visible in plaintext via `gh variable list`, `gh variable get`, the GitHub UI, and the REST API. Secrets are encrypted at rest, and their values are **never returned** by any CLI or API call after being written.

Choose **variable** for non-sensitive config: AWS region, feature flags, deploy targets, image tags, timeout values.
Choose **secret** for anything sensitive: tokens, passwords, API keys, private keys, certificates.

A useful test: "Would I be comfortable with this value appearing in plain text in a `gh variable list` printout read by everyone with repo access?" If no → use `gh secret set`.

</details>

---

**Q2.** Why doesn't `gh secret get <name>` exist, but `gh variable get <name>` does?

<details><summary>Answer</summary>

Secrets are write-only by design — once written, the value is encrypted and GitHub deliberately provides no API to read it back. The `get` subcommand would be a security hole (anyone with the right token could exfiltrate all secrets).

Variables have no such restriction because their values are **expected to be visible**. `gh variable get` exists to make that visibility convenient and scriptable.

</details>

---

**Q3.** You run `gh variable set DEPLOY_TARGET --body "prod" --env production --repo owner/repo` and get a 404 error. What is the most likely cause?

<details><summary>Answer</summary>

The `production` **deployment environment** does not exist in the repository yet. Environment-scoped variables require the environment to be created first.

Fix:
```bash
gh api -X PUT repos/owner/repo/environments/production
gh variable set DEPLOY_TARGET --body "prod" --env production --repo owner/repo
```

</details>

---

**Q4.** What flag would you use to set a repository-level variable for a **different** repo (not the one your working directory is cloned from)?

<details><summary>Answer</summary>

`-R` / `--repo [HOST/]OWNER/REPO` — the inherited repo-selection flag available on all four subcommands.

```bash
gh variable set MY_VAR --body "value" --repo other-owner/other-repo
```

</details>

---

**Q5.** You want to set an org-level variable that is readable only by two specific repositories. Which flags do you need, and what value does `--visibility` take?

<details><summary>Answer</summary>

```bash
gh variable set MY_ORG_VAR \
  --body "value" \
  --org myOrg \
  --visibility selected \
  --repos repo1,repo2
```

`--visibility selected` combined with `--repos` restricts access to the listed repositories. The other visibility options are `all` (every repo in the org) and `private` (only private repos — the default).

Org-level variable operations require **org admin** privileges.

</details>

---

**Q6.** How do you bulk-import a set of variables from a file? What format must the file be in, and what is one common pitfall?

<details><summary>Answer</summary>

Use `gh variable set -f <file>` (or `--env-file`). The file must be in **strict dotenv format**: one `KEY=VALUE` pair per line, no spaces around `=`.

Common pitfall: lines starting with `#` (comments) and lines with `export KEY=VALUE` may be silently skipped or cause errors depending on the CLI version. Stick to bare `KEY=VALUE` lines and test with a small file first.

</details>

---

**Q7.** After running `gh variable list --repo owner/repo`, you notice `DEPLOY_TARGET` does not appear. But you know you set it. What scope did you probably forget to specify?

<details><summary>Answer</summary>

You probably set it with `--env <environment>`, making it an **environment-scoped** variable rather than a repo-level one. `gh variable list` without `--env` only shows repo-level variables.

To see it: `gh variable list --env <environment-name> --repo owner/repo`

This mirrors the same scope separation in `gh secret` — repo, environment, and org scopes are independent buckets.

</details>

---

**Q8.** What are the two aliases provided by `gh variable` for convenience?

<details><summary>Answer</summary>

- `gh variable ls` — alias for `gh variable list`
- `gh variable remove` — alias for `gh variable delete`

Both are built in; no configuration required.

</details>

---

**Q9.** You try to create a variable named `my-feature-flag` and get an HTTP 422 error. What is wrong, and how do you fix it?

<details><summary>Answer</summary>

GitHub variable names are restricted to alphanumeric characters (`[a-z]`, `[A-Z]`, `[0-9]`) and underscores (`_`). Hyphens are not allowed, and the name must start with a letter or underscore.

Fix: rename to `MY_FEATURE_FLAG` or `my_feature_flag`.

```bash
# This fails (HTTP 422):
gh variable set my-feature-flag --body "true" --repo owner/repo

# This works:
gh variable set MY_FEATURE_FLAG --body "true" --repo owner/repo
```

</details>
