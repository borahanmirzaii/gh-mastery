# Drills — `gh release`

> **Sandbox:** run everything against `borahanmirzaii/gh-mastery-sandbox`
> (add `--repo borahanmirzaii/gh-mastery-sandbox` to each command) unless a drill says otherwise.
> **Namespacing:** create only objects prefixed `zz-release-*` and delete them at the end, so parallel drills never collide.

---

## Drill 1 — Create a basic release with auto-generated notes

**Goal:** Create a published release tagged `zz-release-v0.1.0` with notes generated automatically from commits.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh release create zz-release-v0.1.0 \
  --generate-notes \
  --title "zz-release-v0.1.0 — auto notes drill" \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh release view zz-release-v0.1.0 --repo borahanmirzaii/gh-mastery-sandbox --json tagName,name,isDraft --jq '.'`
Expected: `isDraft: false`, `tagName: "zz-release-v0.1.0"`.

---

## Drill 2 — Create a draft release, then publish it

**Goal:** Create a draft release for `zz-release-v0.2.0`, confirm it is in draft state, then publish it using `gh release edit`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Create the draft
gh release create zz-release-v0.2.0 --draft \
  --title "zz-release-v0.2.0 draft" \
  --notes "Initial draft" \
  --repo borahanmirzaii/gh-mastery-sandbox

# Confirm draft state
gh release view zz-release-v0.2.0 --json isDraft --jq .isDraft \
  --repo borahanmirzaii/gh-mastery-sandbox

# Publish
gh release edit zz-release-v0.2.0 --draft=false \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** After `edit`, run `gh release view zz-release-v0.2.0 --json isDraft --jq .isDraft --repo borahanmirzaii/gh-mastery-sandbox` → Expected: `false`.

---

## Drill 3 — Upload an asset with a display label

**Goal:** Create release `zz-release-v0.3.0` and attach a small file with a custom display label, then verify the asset name in the UI.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Create a tiny asset file
echo "drill asset content" > /tmp/zz-asset.txt

# Create the release with the asset and a display label
gh release create zz-release-v0.3.0 \
  /tmp/zz-asset.txt#"Test asset" \
  --title "zz-release-v0.3.0 asset drill" \
  --notes "Testing asset label syntax" \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh release view zz-release-v0.3.0 --json assets --jq '.[].assets[].name' --repo borahanmirzaii/gh-mastery-sandbox`
Expected: `"Test asset"` (the label, not `zz-asset.txt`).

---

## Drill 4 — Upload an additional asset to an existing release

**Goal:** Upload a second file to `zz-release-v0.3.0` using `gh release upload`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
echo "sha256 checksum placeholder" > /tmp/zz-checksums.txt

gh release upload zz-release-v0.3.0 \
  /tmp/zz-checksums.txt#"SHA256 checksums" \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh release view zz-release-v0.3.0 --json assets --jq '[.[].assets[].name]' --repo borahanmirzaii/gh-mastery-sandbox`
Expected: both `"Test asset"` and `"SHA256 checksums"` appear.

---

## Drill 5 — List and filter releases

**Goal:** List all `zz-release-*` releases in the sandbox, excluding drafts, using JSON output.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh release list \
  --exclude-drafts \
  --json tagName,isLatest,isDraft \
  --jq '.[] | select(.tagName | startswith("zz-release-"))' \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** Output should include entries for `zz-release-v0.1.0`, `zz-release-v0.3.0`, and any others you published (not `zz-release-v0.2.0` if it was still a draft when you created it, but it was published — so all three published ones appear).

---

## Boss drill — Full release lifecycle with asset and teardown

Chain: create draft → upload asset → publish → download asset → delete release + tag.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# 1. Create a draft release
gh release create zz-release-v1.0.0 --draft \
  --title "zz-release-v1.0.0 boss drill" \
  --notes "Boss drill release — safe to delete" \
  --repo borahanmirzaii/gh-mastery-sandbox

# 2. Create an asset and upload it with a display label
echo "boss drill build artifact" > /tmp/zz-boss-asset.tar.gz
gh release upload zz-release-v1.0.0 \
  /tmp/zz-boss-asset.tar.gz#"Boss drill artifact" \
  --repo borahanmirzaii/gh-mastery-sandbox

# 3. Publish the draft
gh release edit zz-release-v1.0.0 --draft=false \
  --repo borahanmirzaii/gh-mastery-sandbox

# 4. Verify it is live
gh release view zz-release-v1.0.0 \
  --json tagName,isDraft,assets \
  --jq '{tag: .tagName, draft: .isDraft, assets: [.[].assets[].name]}' \
  --repo borahanmirzaii/gh-mastery-sandbox

# 5. Download the asset
gh release download zz-release-v1.0.0 \
  --pattern '*.gz' \
  --dir /tmp/zz-dl \
  --repo borahanmirzaii/gh-mastery-sandbox
ls /tmp/zz-dl/

# 6. Teardown — delete ALL drill releases + their tags
for tag in zz-release-v0.1.0 zz-release-v0.2.0 zz-release-v0.3.0 zz-release-v1.0.0; do
  gh release delete "$tag" --cleanup-tag --yes \
    --repo borahanmirzaii/gh-mastery-sandbox 2>/dev/null || true
done
```
</details>

**Verify (end state):** `gh release list --repo borahanmirzaii/gh-mastery-sandbox --json tagName --jq '[.[].tagName] | map(select(startswith("zz-release-")))' ` → Expected: `[]`.

**Verify (no lingering tags):** `gh api repos/borahanmirzaii/gh-mastery-sandbox/git/refs/tags --jq '[.[].ref | select(contains("zz-release"))]'` → Expected: `[]`.

**Cleanup:** The teardown loop above covers all drill releases. If any linger, delete manually: `gh release delete <tag> --cleanup-tag --yes --repo borahanmirzaii/gh-mastery-sandbox`.
