# Recall — `gh release`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** What flag auto-generates the release title and notes from merged PRs and commits since the previous tag?

<details><summary>Answer</summary>

`--generate-notes`

It calls the GitHub Release Notes API. If there is no previous release tag, it includes all commits since the repo was created.
</details>

---

**Q2.** Your release flow promotes `dev → main`. You run `gh release create v1.0.0 --generate-notes`. The tag ends up pointing at `dev`. What flag should you have added?

<details><summary>Answer</summary>

`--target main`

`--target <branch-or-sha>` controls which branch (or commit SHA) the new tag is created on. It defaults to the repo's default branch — which may be `dev` if that's what was set — so passing `--target main` explicitly is essential in a `dev → main` promotion flow.
</details>

---

**Q3.** What is the syntax to attach a file `./dist/app.tar.gz` to a release with the display label "App tarball" visible in the GitHub UI?

<details><summary>Answer</summary>

```bash
gh release create v1.0.0 ./dist/app.tar.gz#"App tarball"
```

The `#"Display Label"` suffix appended to the filename sets the visible asset name in the release UI. Without it, GitHub uses the raw filename. In shells that treat `#` as a comment character, quote the whole argument.
</details>

---

**Q4.** What does `--verify-tag` do, and when would you use it?

<details><summary>Answer</summary>

`--verify-tag` aborts `gh release create` if the specified tag does not already exist as a real git tag in the remote repository. You'd use it in signed-tag workflows where the developer or CI system must push a signed tag first (`git tag -s v1.0.0 && git push --tags`), and the release command should fail rather than silently auto-create an unsigned tag.
</details>

---

**Q5.** What is the difference between `--draft` and `--prerelease`?

<details><summary>Answer</summary>

- `--draft` — The release is not published. It is invisible to non-repo-owners and does not occupy the "Latest Release" slot. Can be modified freely; publish later with `gh release edit <tag> --draft=false`.
- `--prerelease` — The release is published and publicly visible, but GitHub excludes it from the "Latest Release" badge on the repo homepage. Use for alpha/beta/RC versions you want users to find but not mistake for stable.

They can be combined (`--draft --prerelease`) to stage a prerelease before publishing it.
</details>

---

**Q6.** You run `gh release delete zz-release-v0.1.0 --yes --repo borahanmirzaii/gh-mastery-sandbox` but then `gh api repos/borahanmirzaii/gh-mastery-sandbox/git/refs/tags` still shows the tag. Why?

<details><summary>Answer</summary>

`gh release delete` only deletes the GitHub Release object, not the underlying git tag. The tag persists in the repo. To delete both, pass `--cleanup-tag`:

```bash
gh release delete zz-release-v0.1.0 --cleanup-tag --yes \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

---

**Q7.** You want to upload a second asset to a release that already exists. Which subcommand do you use, and how is its asset-label syntax different from `gh release create`?

<details><summary>Answer</summary>

Use `gh release upload <tag> <file>#"Label" --repo <repo>`. The `file#"Label"` syntax is identical to `gh release create` — append `#"Display Label"` after the file path. If a file with the same name already exists in the release and you want to replace it, add `--clobber` (but note: the original is deleted before the new upload starts, so a failed upload loses the original permanently).
</details>

---

**Q8.** How do you list only non-draft, non-prerelease releases and get just their tag names?

<details><summary>Answer</summary>

```bash
gh release list \
  --exclude-drafts \
  --exclude-pre-releases \
  --json tagName \
  --jq '.[].tagName' \
  --repo <owner>/<repo>
```
</details>
