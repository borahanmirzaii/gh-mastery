# Drills — `gh release create`

> **Sandbox:** run everything against `borahanmirzaii/gh-mastery-sandbox`
> (add `--repo borahanmirzaii/gh-mastery-sandbox` to each command) unless a drill says otherwise.
> **Namespacing:** create only objects prefixed `zz-release-*` and delete them at the end, so parallel drills never collide.

---

## Drill 1 — Create a release with auto-generated notes

**Goal:** Create a published release tagged `zz-release-v0.1.0` using `--generate-notes` so GitHub writes the changelog for you.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh release create zz-release-v0.1.0 \
  --generate-notes \
  --title "zz-release-v0.1.0 — auto notes drill" \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh release view zz-release-v0.1.0 --json tagName,isDraft,body --jq '{tag: .tagName, draft: .isDraft}' --repo borahanmirzaii/gh-mastery-sandbox`
Expected: `{"tag":"zz-release-v0.1.0","draft":false}`.

---

## Drill 2 — Create a release with a custom `--target`

**Goal:** Create `zz-release-v0.2.0` explicitly targeting the `main` branch (simulating a `dev → main` promotion flow). Confirm the tag's commit is on `main`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh release create zz-release-v0.2.0 \
  --target main \
  --generate-notes \
  --title "zz-release-v0.2.0 — explicit target" \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh api repos/borahanmirzaii/gh-mastery-sandbox/git/refs/tags/zz-release-v0.2.0 --jq '.object.sha'` — then confirm that SHA is reachable from main.

---

## Drill 3 — Attach an asset with a display label

**Goal:** Create `zz-release-v0.3.0` with a tiny asset file and the display label `"Test asset"`. Confirm the label appears (not the raw filename).

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Create a tiny asset
echo "drill build artifact" > /tmp/zz-artifact.txt

# Create release with the asset + label
gh release create zz-release-v0.3.0 \
  "/tmp/zz-artifact.txt#Test asset" \
  --title "zz-release-v0.3.0 — asset label drill" \
  --notes "Testing the file#Label syntax." \
  --repo borahanmirzaii/gh-mastery-sandbox
```
</details>

**Verify:** `gh release view zz-release-v0.3.0 --json assets --jq '[.[].assets[].name]' --repo borahanmirzaii/gh-mastery-sandbox`
Expected: `["Test asset"]` — the label, not `zz-artifact.txt`.

---

## Drill 4 — Draft + publish workflow

**Goal:** Create `zz-release-v0.4.0` as a draft, verify its draft state, then publish it with `gh release edit`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Create as draft
gh release create zz-release-v0.4.0 \
  --draft \
  --title "zz-release-v0.4.0 draft" \
  --notes "Will publish after review." \
  --repo borahanmirzaii/gh-mastery-sandbox

# Confirm it is a draft
gh release view zz-release-v0.4.0 --json isDraft --jq .isDraft \
  --repo borahanmirzaii/gh-mastery-sandbox
# → true

# Publish
gh release edit zz-release-v0.4.0 --draft=false \
  --repo borahanmirzaii/gh-mastery-sandbox

# Confirm published
gh release view zz-release-v0.4.0 --json isDraft --jq .isDraft \
  --repo borahanmirzaii/gh-mastery-sandbox
# → false
```
</details>

**Verify:** Final `gh release view ... --json isDraft --jq .isDraft` returns `false`.

---

## Boss drill — Full release lifecycle: create → asset → verify → teardown

Chain: create a draft release with `--target main` and `--generate-notes`, upload an additional asset, publish, verify the release JSON, then fully tear down (release + tag).

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# 1. Create the draft
gh release create zz-release-v1.0.0 \
  --draft \
  --target main \
  --generate-notes \
  --title "zz-release-v1.0.0 boss drill" \
  --repo borahanmirzaii/gh-mastery-sandbox

# 2. Upload an additional asset with a label
echo "build checksum" > /tmp/zz-checksums.txt
gh release upload zz-release-v1.0.0 \
  "/tmp/zz-checksums.txt#SHA256 checksums" \
  --repo borahanmirzaii/gh-mastery-sandbox

# 3. Publish the draft
gh release edit zz-release-v1.0.0 --draft=false \
  --repo borahanmirzaii/gh-mastery-sandbox

# 4. Verify the full release shape
gh release view zz-release-v1.0.0 \
  --json tagName,isDraft,isPrerelease,assets \
  --jq '{tag: .tagName, draft: .isDraft, prerelease: .isPrerelease, assets: [.[].assets[].name]}' \
  --repo borahanmirzaii/gh-mastery-sandbox

# 5. Teardown — delete all drill releases + tags
for tag in zz-release-v0.1.0 zz-release-v0.2.0 zz-release-v0.3.0 zz-release-v0.4.0 zz-release-v1.0.0; do
  gh release delete "$tag" --cleanup-tag --yes \
    --repo borahanmirzaii/gh-mastery-sandbox 2>/dev/null || true
done
```
</details>

**Verify (end state):** `gh release list --repo borahanmirzaii/gh-mastery-sandbox --json tagName --jq '[.[].tagName] | map(select(startswith("zz-release-")))'` → Expected: `[]`.

**Verify (no lingering tags):** `gh api repos/borahanmirzaii/gh-mastery-sandbox/git/refs/tags --jq '[.[].ref | select(contains("zz-release"))]'` → Expected: `[]`.

**Cleanup:** The teardown loop above covers everything. If any releases linger, delete them manually: `gh release delete <tag> --cleanup-tag --yes --repo borahanmirzaii/gh-mastery-sandbox`.
