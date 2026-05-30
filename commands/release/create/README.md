# `gh release create`

> **One-liner:** Create a new GitHub Release for a tag — auto-generating notes, setting draft/prerelease state, and uploading assets in one command.

## When you reach for it

At the close of a milestone or sprint: all PRs have merged into `main`, you're ready to tag the work, and you want a polished release entry on the Releases page without manually curating a changelog. The canonical pattern in this project is:

```bash
gh release create v0.2.0 \
  --target main \
  --generate-notes \
  --title "v0.2.0 — Core command groups"
```

You also reach for it when:
- Releasing a binary with downloadable assets (`./dist/app.tar.gz#"App tarball"`).
- Cutting a prerelease (RC, beta) that should be visible but not "Latest".
- Creating a draft release so the team can review notes before going live.

## Key flags

- `--generate-notes` — Calls the GitHub Release Notes API to auto-generate a title and notes from merged PRs and commits since the previous tag. The cleanest workflow: no manual changelog. If you also pass `--notes`, those notes are prepended to the auto-generated block.

- `--target <branch-or-sha>` — The branch or commit the new tag is created on if the tag doesn't already exist. Defaults to the repo's default branch. **Always specify `--target main`** when your flow promotes `dev → main`, or the tag lands on the wrong branch.

- `-d, --draft` — Saves the release without publishing it; invisible to non-owners. Publish later with `gh release edit <tag> --draft=false`. Drafts can be freely modified — assets added/removed — before going live.

- `-p, --prerelease` — Publishes the release but marks it as a prerelease. GitHub excludes prereleases from the "Latest Release" slot. Use for alpha/beta/RC cuts.

- `--verify-tag` — Aborts if the tag doesn't already exist as a real git tag in the remote. Use in signed-tag workflows where the tag must be pushed and verified before a release is attached.

- `-t, --title <string>` — Sets the release title. If omitted with `--generate-notes`, GitHub generates the title automatically.

- `-n, --notes <string>` — Inline release notes. Prepended to `--generate-notes` output if both are given.

- `-F, --notes-file <file>` — Read notes from a file. Use `-F -` for stdin.

- `--notes-from-tag` — Use the annotation from the git tag (or the commit message if the tag is lightweight) as the release notes.

- `--notes-start-tag <tag>` — Override the "previous tag" anchor for `--generate-notes`. Useful when you skip a tag or use non-standard tag ordering.

- `--fail-on-no-commits` — Fails if there are no new commits since the last release. Guards against duplicate releases when automation might re-trigger.

- `--latest` / `--latest=false` — Explicitly control the "Latest" badge. Default is automatic based on date and semver. Pass `--latest=false` to release a patch for an older major version without displacing the current latest.

- `--discussion-category <string>` — Creates a linked Discussion in the given category when the release is published. Useful for announcing and collecting feedback.

## Examples

```bash
# Cleanest milestone close: auto-notes, targeting main
gh release create v0.2.0 \
  --target main \
  --generate-notes \
  --repo borahanmirzaii/gh-mastery

# Attach an asset with a display label (the #"..." syntax)
gh release create v1.0.0 \
  ./dist/myapp-linux-amd64.tar.gz#"Linux x86_64 tarball" \
  ./dist/myapp-darwin-arm64.tar.gz#"macOS ARM64 tarball" \
  --generate-notes \
  --target main \
  --repo borahanmirzaii/gh-mastery

# Draft a release first, review, then publish
gh release create v1.1.0 --draft \
  --title "v1.1.0 — review me" \
  --notes "Draft notes — to be edited before publishing." \
  --repo borahanmirzaii/gh-mastery
# ... review on GitHub, then:
gh release edit v1.1.0 --draft=false --repo borahanmirzaii/gh-mastery

# Prerelease (RC) — visible but not "Latest"
gh release create v2.0.0-rc1 \
  --prerelease \
  --generate-notes \
  --notes-start-tag v1.9.0 \
  --target main \
  --repo borahanmirzaii/gh-mastery

# Gate on the tag existing in the remote (signed-tag workflow)
git tag -s v1.2.0 -m "Release v1.2.0"
git push --tags origin
gh release create v1.2.0 \
  --verify-tag \
  --generate-notes \
  --repo borahanmirzaii/gh-mastery

# Fail fast if nothing new since the last release
gh release create v1.3.0 \
  --fail-on-no-commits \
  --generate-notes \
  --repo borahanmirzaii/gh-mastery
```

## Gotchas

1. **`--generate-notes` scopes from the previous release tag** — if no prior release exists, it pulls in all commits since the repo was created. On a first release this can be noisy; use `--notes-start-tag` to narrow the window.

2. **`--target` defaults to the repo's default branch** — in this project's `dev → main` flow, omitting `--target main` means the release tag points at `dev`. Always be explicit.

3. **Asset label syntax: `file#"Label"`** — the `#"Display Label"` suffix after the file path sets the name visible in the GitHub UI. Without it, GitHub uses the raw filename. In `zsh`/`bash`, `#` starts an inline comment after whitespace — quote the whole argument (`'./dist/app.tar.gz#My label'`) to be safe.

4. **`--verify-tag` is a safety gate, not the default** — without it, if the tag doesn't exist, `gh release create` auto-creates one pointing at `--target`. This is convenient but bypasses signing workflows. Use `--verify-tag` in any CI pipeline where tags must be explicitly pushed.

5. **`--draft` and `--prerelease` are independent** — you can have a draft prerelease (`--draft --prerelease`) or a published prerelease (`--prerelease` only). A draft is invisible; a prerelease is public but excluded from "Latest".

6. **Asset upload is a multi-step API call** — `gh release create <tag> <assets>` creates the release as a draft internally, uploads each asset, then publishes. If you have immutability enabled, the release locks after publication — so any failed mid-upload asset is gone. Check asset count after creation to confirm.

7. **`--generate-notes` and `--notes` compose** — `--notes` content is prepended above the auto-generated block. Use it for a short "Highlights" blurb: `--notes "## Highlights\n- Feature X landed" --generate-notes`.

8. **Alias:** `gh release new` is an alias for `gh release create` (as of 2.92.0).

## Concepts

- None — the tag-and-release model is self-contained; no separate `concepts/` doc is needed.

## Sources

- Manual: https://cli.github.com/manual/gh_release_create
- Local: `gh release create --help` (gh 2.92.0)
