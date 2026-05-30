# Drills — `gh issue create`

> **Sandbox:** run everything against `borahanmirzaii/gh-mastery-sandbox`
> **Namespacing:** all created issues must be prefixed `zz-issue-*`; delete them in cleanup.

---

## Drill 1 — Non-interactive creation with flags only

**Goal:** Create an issue where title, body, and label are all supplied via flags — no prompts, no browser, no editor.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh issue create \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --title "zz-issue-create-d1: flag-only creation" \
  --body "All fields supplied via CLI flags. No prompts." \
  --label "bug" \
  --assignee @me
```
</details>

**Verify:** `gh issue list --repo borahanmirzaii/gh-mastery-sandbox --search "zz-issue-create-d1 in:title" --json number,title,labels --jq '.[0] | {title, labels: [.labels[].name]}'` → title matches; labels include `bug`.

---

## Drill 2 — Rich body from stdin

**Goal:** Create an issue whose body has multiple markdown sections, supplied via `--body-file -` and a heredoc. Include an absolute URL link in the body (not a relative path).

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh issue create \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --title "zz-issue-create-d2: rich stdin body" \
  --body-file - <<'EOF'
## Problem

The widget feature is missing.

## Acceptance criteria

- [ ] Widget loads.
- [ ] Widget has unit tests.

## Resources

**Plan:** https://github.com/borahanmirzaii/gh-mastery/blob/main/docs/superpowers/plans/2026-05-26-gh-mastery.md
EOF
```
</details>

**Verify:** `gh issue list --repo borahanmirzaii/gh-mastery-sandbox --search "zz-issue-create-d2 in:title" --json body --jq '.[].body'` → body contains `## Problem` and an `https://` link.

---

## Drill 3 — Batch creation via a loop

**Goal:** Create three issues in a single shell loop, each with a different title but shared label. This mirrors the Task 0.8 pattern used to bootstrap this curriculum.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
for ITEM in alpha beta gamma; do
  gh issue create \
    --repo borahanmirzaii/gh-mastery-sandbox \
    --title "zz-issue-create-d3: batch-$ITEM" \
    --body "Batch-created item: $ITEM" \
    --label "documentation"
done
```
</details>

**Verify:** `gh issue list --repo borahanmirzaii/gh-mastery-sandbox --search "zz-issue-create-d3 in:title" --json number,title --jq 'length'` → `3`.

---

## Boss drill — Scripted issue creation with piped body

**Goal:** Use `gh issue view` on an existing issue to fetch its body, then pipe that body directly into a new `gh issue create` call via `--body-file -`. This tests the "body from a generator" pattern.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Fetch an existing issue's body and pipe it into a new issue
gh issue view 4 \
  --repo borahanmirzaii/gh-mastery \
  --json body --jq .body \
  | gh issue create \
      --repo borahanmirzaii/gh-mastery-sandbox \
      --title "zz-issue-create-boss: piped body from issue #4" \
      --body-file -
```
</details>

**Verify:** `gh issue list --repo borahanmirzaii/gh-mastery-sandbox --search "zz-issue-create-boss in:title" --json body --jq '.[].body | length > 0'` → `true`.

**Cleanup — delete all `zz-issue-*` issues:**

```bash
gh issue list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --search "zz-issue in:title" \
  --state all \
  --json number \
  --jq '.[].number' \
| xargs -I{} gh issue delete {} \
    --repo borahanmirzaii/gh-mastery-sandbox \
    --yes 2>/dev/null || true

echo "Cleanup done."
gh issue list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --search "zz-issue in:title" \
  --state all \
  --json number --jq 'length'
# Expected: 0
```
