# Drills — `gh label`

> **Sandbox:** run everything against `borahanmirzaii/gh-mastery-sandbox`
> (add `--repo borahanmirzaii/gh-mastery-sandbox` to each command).
> **Namespacing:** create only labels prefixed `zz-label-*` and delete them at the end,
> so parallel drills never collide.

---

## Drill 1 — Create labels with explicit color and description

**Goal:** Create two labels in the sandbox: `zz-label-bug` (red) and `zz-label-docs` (blue).

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh label create "zz-label-bug" \
  --color E11D48 \
  --description "Something is broken" \
  --repo borahanmirzaii/gh-mastery-sandbox

gh label create "zz-label-docs" \
  --color 1D76DB \
  --description "Documentation changes" \
  --repo borahanmirzaii/gh-mastery-sandbox
```

</details>

**Verify:**
```bash
gh label list \
  --search "zz-label-" \
  --json name,color,description \
  --repo borahanmirzaii/gh-mastery-sandbox
```
Expected: two entries — `zz-label-bug` and `zz-label-docs`.

---

## Drill 2 — Edit a label (rename + recolor)

**Goal:** Rename `zz-label-bug` to `zz-label-defect` and change its color to `B60205`.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh label edit "zz-label-bug" \
  --name "zz-label-defect" \
  --color B60205 \
  --repo borahanmirzaii/gh-mastery-sandbox
```

</details>

**Verify:**
```bash
gh label list \
  --search "zz-label-" \
  --json name,color \
  --jq '.[] | "\(.name) #\(.color)"' \
  --repo borahanmirzaii/gh-mastery-sandbox
```
Expected: `zz-label-defect #B60205` and `zz-label-docs #1D76DB`. No `zz-label-bug`.

---

## Drill 3 — Clone a label set from another repo

**Goal:** Clone all labels from `borahanmirzaii/gh-mastery` into the sandbox. Note which labels are copied and which (if any) are skipped because they already exist.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
# Clone without --force: skips labels that already exist in the sandbox
gh label clone borahanmirzaii/gh-mastery \
  --repo borahanmirzaii/gh-mastery-sandbox
```

Then, to also overwrite existing ones:

```bash
gh label clone borahanmirzaii/gh-mastery \
  --force \
  --repo borahanmirzaii/gh-mastery-sandbox
```

</details>

**Verify:**
```bash
gh label list \
  --limit 100 \
  --json name \
  --jq '[.[].name] | sort | join(", ")' \
  --repo borahanmirzaii/gh-mastery-sandbox
```
Expected: includes `kind:command`, `kind:concept`, `kind:infra`, `meta` (the gh-mastery taxonomy) alongside the sandbox's own labels.

**Cleanup after this drill** (remove the cloned labels so they don't interfere with the boss drill):
```bash
for L in "kind:command" "kind:concept" "kind:infra" "meta"; do
  gh label delete "$L" --yes --repo borahanmirzaii/gh-mastery-sandbox 2>/dev/null || true
done
```

---

## Drill 4 — Delete labels non-interactively

**Goal:** Delete `zz-label-docs` without being prompted for confirmation.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh label delete "zz-label-docs" \
  --yes \
  --repo borahanmirzaii/gh-mastery-sandbox
```

</details>

**Verify:**
```bash
gh label list \
  --search "zz-label-docs" \
  --json name \
  --repo borahanmirzaii/gh-mastery-sandbox
```
Expected: empty array `[]`.

---

## Boss drill — Bootstrap a `kind:*` taxonomy, prove upsert, then clean up

**Goal:** Simulate a real label-bootstrap script. You will:
1. Create a small `kind:*` taxonomy in the sandbox with `--force` (first run = create).
2. Re-run the exact same commands (second run = update/upsert — should not error).
3. Verify the labels exist with updated values.
4. Delete all `zz-label-*` labels to leave the sandbox clean.

**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

**Step 1 — First run (creates the labels):**

```bash
REPO="borahanmirzaii/gh-mastery-sandbox"

gh label create "zz-label-kind:feature" \
  --color 1D76DB \
  --description "New feature or request" \
  --force \
  --repo "$REPO"

gh label create "zz-label-kind:bug" \
  --color E11D48 \
  --description "Something is broken" \
  --force \
  --repo "$REPO"

gh label create "zz-label-kind:chore" \
  --color 0E8A16 \
  --description "Maintenance, dependencies, cleanup" \
  --force \
  --repo "$REPO"
```

**Step 2 — Second run (upserts — changes description to prove update):**

```bash
gh label create "zz-label-kind:feature" \
  --color 1D76DB \
  --description "New feature or request (v2)" \
  --force \
  --repo "$REPO"

gh label create "zz-label-kind:bug" \
  --color E11D48 \
  --description "Something is broken (v2)" \
  --force \
  --repo "$REPO"

gh label create "zz-label-kind:chore" \
  --color 0E8A16 \
  --description "Maintenance, dependencies, cleanup (v2)" \
  --force \
  --repo "$REPO"
```

No errors on the second run — that is the whole point of `--force`.

**Step 3 — Verify upsert took effect:**

```bash
gh label list \
  --search "zz-label-kind:" \
  --json name,description \
  --jq '.[] | "\(.name): \(.description)"' \
  --repo "$REPO"
```

Expected: three lines, each ending in `(v2)`.

**Step 4 — Also delete the zz-label-defect leftover from Drill 2:**

```bash
gh label delete "zz-label-defect" --yes --repo "$REPO" 2>/dev/null || true
```

**Step 5 — Clean up all zz-label-* labels:**

```bash
REPO="borahanmirzaii/gh-mastery-sandbox"

gh label list \
  --limit 100 \
  --json name \
  --jq '[.[].name | select(startswith("zz-label-"))] | .[]' \
  --repo "$REPO" \
| while read -r L; do
    gh label delete "$L" --yes --repo "$REPO"
    echo "deleted: $L"
  done
```

</details>

**Verify (sandbox is clean):**
```bash
gh label list \
  --limit 100 \
  --json name \
  --jq '[.[].name | select(startswith("zz-label-"))] | length' \
  --repo borahanmirzaii/gh-mastery-sandbox
```
Expected: `0`.

**Cleanup confirmation:** the verify command above is the cleanup check — if it prints `0`, you are done.
