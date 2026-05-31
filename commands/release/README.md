# `gh release`

> **One-liner:** Create, inspect, edit, and delete GitHub Releases — including tag management, release-note generation, and asset uploads.

## When you reach for it

The natural home for `gh release` is **the end of a milestone** — once all PRs for a phase have merged into `main`, you tag the work and ship it as a release. In this project that looks like:

```bash
gh release create v0.2.0 --target main --generate-notes --title "v0.2.0 — Core command groups"
```

You also reach for it when:
- You want to attach a binary, tarball, or changelog file as a downloadable asset.
- You need to inspect what the latest release is (`gh release view`) before deciding the next version tag.
- You're cleaning up a mis-fired tag+release pair during drill work.

## Subcommands

| Subcommand | Purpose | Node? |
|---|---|---|
| `gh release create` | Create a new release, optionally auto-generating notes and uploading assets | [→ `create/`](./create/) |
| `gh release list` | List releases in a repository | inline |
| `gh release view` | Show details for a specific release (or the latest) | inline |
| `gh release edit` | Modify a release's title, notes, draft/prerelease status, or tag | inline |
| `gh release delete` | Delete a release (optionally also deleting the underlying tag) | inline |
| `gh release delete-asset` | Delete a single asset from an existing release | inline |
| `gh release download` | Download release assets to disk | inline |
| `gh release upload` | Upload additional asset files to an existing release | inline |
| `gh release verify` | Verify the attestation for a release (SLSA provenance) | inline |
| `gh release verify-asset` | Verify a local file matches a release's signed attestation | inline |

## Key flags

Flags that apply across subcommands or that are most commonly useful:

**`gh release create`**

- `--generate-notes` — Auto-generates the release title and notes from merged PRs and commits since the previous tag using the GitHub Release Notes API. The cleanest way to ship — no manual changelog editing.
- `--target <branch-or-sha>` — Controls which branch or commit the new tag points to. Defaults to the repo's default branch. Use `--target main` when your release flow promotes `dev → main` and you want the tag on `main` (not `dev`).
- `-d, --draft` — Saves the release without publishing it. Useful for staging; you can upload assets, preview, then flip live with `gh release edit <tag> --draft=false`.
- `-p, --prerelease` — Marks the release as a prerelease. It appears in the releases list but GitHub won't show it as the "Latest Release" on the repo home page.
- `--verify-tag` — Aborts the command if the tag doesn't already exist as a real git tag in the remote repo. Useful as a gate in signed-tag workflows where `git push --tags` must happen first.
- `--notes-start-tag <tag>` — When using `--generate-notes`, start the generated changelog from this tag rather than the previous release tag.
- `--fail-on-no-commits` — Fails if no new commits exist since the last release. Guards against accidental duplicate releases.
- `--latest` / `--latest=false` — Explicitly controls the "Latest" badge. Defaults to automatic (date + semver heuristic).
- `-t, --title <string>` — Set the release title. If omitted alongside `--generate-notes`, GitHub generates a title automatically.
- `-n, --notes <string>` / `-F, --notes-file <file>` — Provide or read release notes; `-F -` reads from stdin.
- `--discussion-category <string>` — Starts a linked Discussion in the given category when the release is published.

**`gh release list`**

- `--exclude-drafts` — Omits draft releases.
- `--exclude-pre-releases` — Omits prereleases.
- `-L, --limit <n>` — Cap the number of results (default 30).
- `--json fields` — Machine-readable output; available fields: `createdAt, isDraft, isImmutable, isLatest, isPrerelease, name, publishedAt, tagName`.

**`gh release download`**

- `-p, --pattern <glob>` — Download only assets matching the glob; stackable (`-p '*.deb' -p '*.rpm'`).
- `-A, --archive <zip|tar.gz>` — Download the source code archive instead of release assets.
- `-D, --dir <directory>` — Destination directory (default: `.`).
- `--clobber` — Overwrite existing files of the same name.

**`gh release delete`**

- `--cleanup-tag` — Deletes the git tag in addition to the release object. Required for a full teardown during drill cleanup.
- `-y, --yes` — Skips the interactive confirmation prompt; essential in scripts.

**`gh release upload`**

- `--clobber` — Deletes and re-uploads any asset with the same name. Warning: if the upload fails mid-way, the original asset is lost permanently.

**`gh release edit`**

- `--draft=false` — Publishes a draft release without changing other fields — the standard "go live" pattern.
- `--tag <string>` — Rename the tag itself. Use with caution in immutable-release repos.

## Examples

```bash
# Create a release with auto-generated notes and title, targeting main
gh release create v0.2.0 --target main --generate-notes \
  --repo borahanmirzaii/gh-mastery

# Create a draft release, then publish it later
gh release create v1.0.0-rc1 --draft --title "v1.0.0 RC1" --notes "Release candidate" \
  --repo borahanmirzaii/gh-mastery
gh release edit v1.0.0-rc1 --draft=false --repo borahanmirzaii/gh-mastery

# Upload a tarball with a display label visible in the GitHub UI
gh release create v1.2.0 ./dist/app.tar.gz#"App tarball" \
  --generate-notes --target main --repo borahanmirzaii/gh-mastery

# Upload an additional asset to an existing release
echo "checksum-here" > /tmp/checksums.txt
gh release upload v1.2.0 /tmp/checksums.txt#"SHA256 checksums" \
  --repo borahanmirzaii/gh-mastery

# List the 5 most recent releases, excluding drafts
gh release list --exclude-drafts --limit 5 --repo borahanmirzaii/gh-mastery

# View the latest release in JSON and extract the tag name
gh release view --json tagName --jq .tagName --repo borahanmirzaii/gh-mastery

# Download all assets from a specific release into /tmp
gh release download v1.2.0 --dir /tmp --repo borahanmirzaii/gh-mastery

# Delete a release AND its underlying git tag
gh release delete zz-release-v0.0.1 --cleanup-tag --yes \
  --repo borahanmirzaii/gh-mastery-sandbox
```

## Gotchas

1. **`--generate-notes` scopes to merged PRs and commits since the previous tag** — if there is no previous release, it includes everything since the repo was created. Plan tag ordering carefully.

2. **`--target <branch-or-sha>` defaults to the repo's default branch** — in a `dev → main` promotion flow, always pass `--target main` explicitly, or your release tag will point to `dev` (or whatever the default branch is), not `main`.

3. **Asset upload uses `file#"Display Label"` syntax** — e.g., `gh release create v1.0.0 ./dist/app.tar.gz#"App tarball"`. The text after `#` becomes the file's visible name in the GitHub release UI. Omit it and GitHub uses the bare filename. Quote the whole argument (or the `#...` part) in shells that treat `#` as a comment character.

4. **`--verify-tag` gates on the tag already existing remotely** — if you run `git tag v1.0.0 && git push --tags origin` first and then pass `--verify-tag`, the command aborts if that push somehow didn't land. Useful for signed-tag workflows where the tag must be created and verified before any release object is attached.

5. **`--draft` vs `--prerelease`** — these are independent flags. A draft is invisible to non-owners and doesn't show up in the latest-release slot; a prerelease is publicly visible but excluded from "Latest". You can combine them: `--draft --prerelease` saves a prerelease that isn't published yet.

6. **`gh release delete` does NOT delete the git tag by default** — the tag lingers in the repo even after the GitHub Release object is gone. Always add `--cleanup-tag` during drill teardown (or you'll have dangling tags polluting `gh api repos/.../git/refs/tags`).

7. **Asset upload is a multi-step API sequence** — `gh release create <tag> <asset>` actually creates the release as a draft, uploads the asset, then publishes. If you have release immutability enabled, modifications are locked after publish. This means the window to add or replace assets is between creation and publication.

8. **`--clobber` on `gh release upload` is destructive** — the existing asset is deleted before the new one is uploaded. If the new upload fails partway through, the original is gone.

9. **`--generate-notes` and `--notes` compose** — extra notes passed via `--notes` are prepended to the auto-generated block. Handy for a short "Highlights" section above the auto-list.

10. **`verify` and `verify-asset` require SLSA attestations to exist** — these subcommands check cryptographic provenance; they'll error on repos that don't publish attestations. Not a routine release workflow step unless your repo uses GitHub's artifact attestation feature.

## Concepts

- None — releases are a GitHub UI/API concept with no separate `concepts/` doc needed; the tag + release model is self-evident from the examples above.

## Sources

- Manual: https://cli.github.com/manual/gh_release
- Local: `gh release --help` (gh 2.92.0)
