# Recall — `gh release create`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

---

**Q1.** What flag makes GitHub automatically write the release notes and title from merged PRs and commits since the last tag?

<details><summary>Answer</summary>

`--generate-notes`

It calls the GitHub Release Notes API. If there is no previous release, it covers all commits from the repo's beginning. Pair with `--notes "..."` to prepend a custom "Highlights" section above the auto-generated list.
</details>

---

**Q2.** You run `gh release create v2.0.0 --generate-notes` on a repo whose default branch is `dev`. Where does the tag end up?

<details><summary>Answer</summary>

On `dev` — because `--target` defaults to the repo's default branch. To target `main` explicitly:

```bash
gh release create v2.0.0 --generate-notes --target main
```

In a `dev → main` promotion workflow, always pass `--target main`.
</details>

---

**Q3.** Write the command to attach `./dist/app.tar.gz` to a release with the GitHub UI display label "Linux binary".

<details><summary>Answer</summary>

```bash
gh release create v1.0.0 "./dist/app.tar.gz#Linux binary" --generate-notes --target main
```

The `#"Label"` suffix after the file path sets the visible asset name. Quote the argument in shells where `#` starts a comment.
</details>

---

**Q4.** What does `--verify-tag` do, and what happens if the tag does not already exist remotely?

<details><summary>Answer</summary>

`--verify-tag` checks that the specified tag already exists as a git tag in the remote repository. If the tag is absent, `gh release create` aborts with an error instead of auto-creating the tag. Use this in signed-tag workflows where `git push --tags` must precede the release command.
</details>

---

**Q5.** What is the difference between `--draft` and `--prerelease` on `gh release create`?

<details><summary>Answer</summary>

- `--draft` — Release is not published; invisible to non-owners; does not appear in the "Latest Release" slot. Freely editable before publishing via `gh release edit <tag> --draft=false`.
- `--prerelease` — Release is published and publicly visible, but GitHub excludes it from the "Latest" badge on the repo homepage. Suitable for RCs and betas.

They are independent: you can combine them (`--draft --prerelease`) to stage a prerelease before going live.
</details>

---

**Q6.** You want `--generate-notes` to include only commits since tag `v1.8.0`, not since the very last release (`v1.9.9`). Which flag controls this?

<details><summary>Answer</summary>

`--notes-start-tag v1.8.0`

This overrides the "previous tag" anchor for the generated notes. Useful when you skip a tag, use non-semver tagging schemes, or want to aggregate changelog entries across multiple patch versions into a single release entry.
</details>

---

**Q7.** Your CI should never create a duplicate release if re-triggered with no new commits. Which flag enforces this?

<details><summary>Answer</summary>

`--fail-on-no-commits`

If there are no new commits since the last release, the command exits with an error rather than creating a duplicate release. This flag has no effect on the very first release in the repo (there is nothing to compare against).
</details>
