# gh-mastery Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single repo that teaches every `gh` command group — and every subcommand meaty enough to deserve it — via a reference page, a drills page, and a recall page each, arranged as a **recursive tree that mirrors `gh`'s own command tree**, built with `gh` itself and shipped milestone by milestone.

**Architecture:** Scaffold-then-parallel (locked in RFC #1, spec §6.1). Phase 0 stands up the infra *and* a generator: `scripts/scaffold.sh` walks `gh --help` recursively, records the live command tree in `commands/_inventory.md`, and creates a node (`README`/`DRILLS`/`RECALL` from `_TEMPLATE/`) for every group plus every curated promotion in `commands/_promotions.txt`; `scripts/gen-index.sh` regenerates the README progress dashboard and `cheatsheet.md` from the on-disk tree. The empty-but-complete tree lands in one "scaffold" PR. Phase 1+ is a repeating authoring loop: one issue per command group → linked branch/worktree → fill that group's subtree → verify examples against the sandbox → draft PR → squash-merge. Workers never hand-edit the generated index files; the Lead re-runs `gen-index.sh` at merge. Each closed milestone ships a `gh release`.

**Tech Stack:** `gh` 2.92.0, Bash (the two generator scripts), GitHub (Issues/PRs/Projects/Discussions/Milestones/Releases/Pages), GraphQL via `gh api graphql`, Markdown. No application code; "tests" are existence/link checks, generator idempotency checks, plus running the documented example commands against the sandbox repo.

**Spec:** `docs/superpowers/specs/2026-05-26-gh-mastery-design.md` (rev. 2) · **RFC:** [Discussion #1 decision record](https://github.com/borahanmirzaii/gh-mastery/discussions/1#discussioncomment-17061267) · **Board:** Project #14

---

## Conventions used throughout

- **Repo:** `borahanmirzaii/gh-mastery`. **Sandbox:** `borahanmirzaii/gh-mastery-sandbox`.
- **Branch model:** branch from `dev`, PR into `dev`; promote `dev`→`main` for releases.
- **Every PR body** contains `Closes #<issue>` (auto-added by `gh issue develop`).
- **Identity** is already pinned (`borahanmirzaii`); the PreToolUse hook enforces it — don't pre-check.
- **Node:** a directory under `commands/` holding `README.md` + `DRILLS.md` + `RECALL.md`. A node exists for every group and every *promoted* subcommand. Shallow subcommands are documented inline in their parent node's files.
- **Promotion rule of thumb:** a subcommand earns its own node when it has non-trivial flags, gotchas, or an underlying concept worth a dedicated page; otherwise it stays inline. Suggested promotions are listed per group in the Phase 1/2 tables — confirm against the rule while authoring.
- **Generated files are never hand-edited:** `commands/_inventory.md`, the README `<!-- BEGIN PROGRESS -->…<!-- END PROGRESS -->` block, and `cheatsheet.md` are all produced by the scripts. Workers fill node files; the Lead re-runs `scripts/gen-index.sh`.
- **Drill artifacts are namespaced by group** so parallel workers never collide in the sandbox: the `<group>` worker only creates sandbox objects prefixed `zz-<group>-*` (labels, issue titles, release tags, etc.) and cleans up after itself.
- **"Verify" steps** for content tasks mean: the node files exist, internal links resolve, the node's README no longer contains `<!-- STUB -->`, and the example commands in it actually run against the sandbox.

---

## Phase 0 — Infra (Milestone 0)

### Task 0.1: Create the drill sandbox repo

**Files:** none (remote only)

- [ ] **Step 1: Create the public sandbox repo**

```bash
gh repo create borahanmirzaii/gh-mastery-sandbox \
  --public \
  --description "Throwaway playground for gh-mastery drills — safe to reset/delete." \
  --add-readme
```

- [ ] **Step 2: Verify it exists**

Run: `gh repo view borahanmirzaii/gh-mastery-sandbox --json name,visibility --jq '.name + " " + .visibility'`
Expected: `gh-mastery-sandbox PUBLIC`

- [ ] **Step 3: Enable issues + projects so issue/label/pr drills have a target**

```bash
gh repo edit borahanmirzaii/gh-mastery-sandbox --enable-issues --enable-projects
```

(No commit — remote-only task. **Already done** if `gh repo view` in Step 2 succeeds.)

### Task 0.2: Create the label taxonomy

**Files:** none (remote only)

- [ ] **Step 1: Create the four labels**

```bash
gh label create "kind:command" --color 1D76DB --description "A command-group learning issue (README+DRILLS+RECALL)" --force
gh label create "kind:concept" --color 5319E7 --description "A concepts/ explainer doc" --force
gh label create "kind:infra"   --color 0E8A16 --description "Sandbox, template, scaffold scripts, Pages, cheatsheet" --force
gh label create "meta"         --color FBCA04 --description "RFC / spec / plan / process" --force
```

- [ ] **Step 2: Verify**

Run: `gh label list --json name --jq '[.[].name] | sort | join(", ")'`
Expected: includes `kind:command, kind:concept, kind:infra, meta` (plus GitHub defaults).

### Task 0.3: Create milestones

**Files:** none (remote only). `gh` has no `milestone` command — use the REST API.

- [ ] **Step 1: Create the five milestones**

```bash
for M in \
  "M0 — Infra" \
  "M1 — Core (daily drivers)" \
  "M2 — Long tail" \
  "M3 — Concepts" \
  "M4 — Publish (Pages)"; do
  gh api repos/borahanmirzaii/gh-mastery/milestones -f title="$M" --jq '.title + " #" + (.number|tostring)'
done
```

- [ ] **Step 2: Verify**

Run: `gh api repos/borahanmirzaii/gh-mastery/milestones --jq '[.[].title] | join(" | ")'`
Expected: all five titles listed.

### Task 0.4: Write the canonical `_TEMPLATE/`

**Files:**
- Create: `commands/_TEMPLATE/README.md`
- Create: `commands/_TEMPLATE/DRILLS.md`
- Create: `commands/_TEMPLATE/RECALL.md`

The literal token `<group>` is replaced by `scaffold.sh` with the node's command path (`pr`, or `pr create` for a promoted subcommand). The `<!-- STUB -->` marker is how `gen-index.sh` tells a filled node from an empty one — **delete it when the node is authored.**

- [ ] **Step 1: Write `commands/_TEMPLATE/README.md`**

````markdown
<!-- STUB -->
# `gh <group>`

> **One-liner:** <what this command/subcommand does, in one sentence>.

## When you reach for it

<The real workflow moment. Tie to the solo-builder loop where it fits,
e.g. "Step 1 of the Loop — you draft the brief as an issue body.">

## Subcommands

(Group nodes only — for a promoted subcommand node, delete this section.)

| Subcommand | Purpose | Node? |
|---|---|---|
| `gh <group> <sub>` | <one line> | inline / [→ `<sub>/`](./<sub>/) |

## Key flags

Only the flags that matter, each with *when* to use it (not a raw `--help` dump).

- `--<flag>` — <meaning; when you'd use it>.

## Examples

```bash
# <what this does>
gh <group> --<flag> <value>
```

## Gotchas

- <friction point discovered while drilling>.

## Concepts

- <link into ../../concepts/<x>.md when a GitHub concept underlies this command, else "None.">

## Sources

- Manual: https://cli.github.com/manual/gh_<group>
- Local: `gh <group> --help` (gh 2.92.0)
````

- [ ] **Step 2: Write `commands/_TEMPLATE/DRILLS.md`**

````markdown
# Drills — `gh <group>`

> **Sandbox:** run everything against `borahanmirzaii/gh-mastery-sandbox`
> (add `--repo borahanmirzaii/gh-mastery-sandbox` or `cd` into a clone) unless a drill says otherwise.
> **Namespacing:** create only objects prefixed `zz-<group>-*` and delete them at the end, so parallel drills never collide.

## Drill 1 — <goal>

**Goal:** <what you should accomplish>.
**Try it yourself first**, then reveal:

<details><summary>Answer</summary>

```bash
gh <group> <sub> ...
```
</details>

**Verify:** `gh <group> list ...` → expect <observable outcome>.

## Boss drill — <realistic mini-workflow>

Chain several subcommands into one task that mirrors real use.

<details><summary>Answer</summary>

```bash
gh <group> ...
```
</details>

**Verify:** <end state you can check>. **Cleanup:** delete every `zz-<group>-*` object created.
````

- [ ] **Step 3: Write `commands/_TEMPLATE/RECALL.md`**

````markdown
# Recall — `gh <group>`

Spaced-repetition self-test. Cover the answer, recall it, then check. (≥5 prompts.)

**Q1.** <question about a flag / gotcha / subcommand>?

<details><summary>Answer</summary>

<answer>
</details>
````

- [ ] **Step 4: Verify all three exist**

Run: `ls commands/_TEMPLATE/`
Expected: `DRILLS.md  README.md  RECALL.md`

- [ ] **Step 5: Commit**

```bash
git add commands/_TEMPLATE/
git commit -m "feat: canonical command-node template (reference/drills/recall)"
```

### Task 0.5: Write the tree generator `scripts/scaffold.sh` + promotions seed

**Files:**
- Create: `scripts/scaffold.sh`
- Create: `commands/_promotions.txt`

- [ ] **Step 1: Write `commands/_promotions.txt`** (curated promoted-subcommand nodes; one `group/sub` per line, `#` comments allowed). Seeded from the Phase 1/2 tables — extend as authoring confirms promotions.

```text
# Promoted subcommand nodes — each gets its own commands/<group>/<sub>/ node.
# Lines are paths under commands/. Edit, then re-run scripts/scaffold.sh.
auth/login
auth/refresh
repo/create
repo/edit
issue/create
issue/develop
pr/create
pr/merge
pr/review
release/create
run/watch
workflow/run
search/code
secret/set
project/item-list
project/field-list
```

- [ ] **Step 2: Write `scripts/scaffold.sh`**

```bash
#!/usr/bin/env bash
# scripts/scaffold.sh — generate/refresh the gh-mastery command tree from `gh` itself.
#
#   1. Discovers the live `gh` command tree → commands/_inventory.md
#   2. Creates a node (README/DRILLS/RECALL from _TEMPLATE/) for every command
#      group and every promotion listed in commands/_promotions.txt
#   3. Idempotent: never overwrites a node that already exists
#
# Re-run after a `gh` upgrade, then `git diff commands/_inventory.md` to see what's new.
set -euo pipefail

ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"
CMD="$ROOT/commands"
TPL="$CMD/_TEMPLATE"
PROMO="$CMD/_promotions.txt"
INV="$CMD/_inventory.md"
EXCLUDE="co"   # built-in alias for `pr checkout`, not a real command group

# Parse a `gh ... --help` COMMANDS section into bare subcommand names.
subcommands() {  # args: gh path (none = top level), e.g. pr  |  repo deploy-key
  gh "$@" --help 2>/dev/null | awk '
    /^[A-Z][A-Z ]*COMMANDS/{f=1; next}
    /^[A-Z]/{f=0}
    f && /^[[:space:]]+[a-z]/ {sub(/:$/,"",$1); print $1}
  ' | sort -u
}

groups() { subcommands | grep -vxF "$EXCLUDE"; }

# Create a node from the template if it does not already exist. Idempotent.
make_node() {  # arg: path under commands/, e.g. pr  |  pr/create
  local rel="$1" dir="$CMD/$1"
  [[ -d "$dir" ]] && return 0
  mkdir -p "$dir"
  local label="${rel//\// }"          # pr/create -> "pr create"
  for f in README DRILLS RECALL; do
    sed "s|<group>|$label|g" "$TPL/$f.md" > "$dir/$f.md"
  done
  echo "created node: commands/$rel"
}

# 1. inventory (full live gh tree) ---------------------------------------
{
  echo "# gh command inventory — $(gh --version | head -1)"
  echo "<!-- GENERATED by scripts/scaffold.sh — do not edit. Re-run after a gh upgrade and diff. -->"
  echo
  for g in $(groups); do
    subs="$(subcommands "$g" | tr '\n' ' ')"
    echo "- **$g** — ${subs:-_(no subcommands)_}"
  done
} > "$INV"
echo "wrote $INV"

# 2. group nodes ---------------------------------------------------------
for g in $(groups); do make_node "$g"; done

# 3. promoted subcommand nodes -------------------------------------------
if [[ -f "$PROMO" ]]; then
  grep -vE '^[[:space:]]*(#|$)' "$PROMO" | while read -r rel; do make_node "$rel"; done
fi

echo "scaffold complete. Next: scripts/gen-index.sh"
```

- [ ] **Step 3: Make it executable**

```bash
chmod +x scripts/scaffold.sh
```

- [ ] **Step 4: Dry verify the discovery (before generating anything real)**

Run: `bash -c 'source <(sed -n "/^subcommands()/,/^}/p;/^groups()/,/^}/p" scripts/scaffold.sh); groups | tr "\n" " "'`
Expected: the ~33 group names (`agent-task alias api attestation auth browse cache codespace …`) with **no** `co`.

- [ ] **Step 5: Commit** (script only — running it is Task 0.7)

```bash
git add scripts/scaffold.sh commands/_promotions.txt
git commit -m "feat: scaffold.sh — generate command tree + inventory from gh"
```

### Task 0.6: Write the index generator `scripts/gen-index.sh`

**Files:**
- Create: `scripts/gen-index.sh`
- Modify: `README.md` (add the marker block in Task 0.7 Step 1)

`gen-index.sh` rebuilds the README progress block and `cheatsheet.md` from the on-disk tree. A node counts as **done** when its `README.md` no longer contains `<!-- STUB -->`.

- [ ] **Step 1: Write `scripts/gen-index.sh`**

```bash
#!/usr/bin/env bash
# scripts/gen-index.sh — regenerate the README progress block + cheatsheet.md
# from the on-disk commands/ tree. A node is "done" when its README lacks <!-- STUB -->.
set -euo pipefail

ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"
CMD="$ROOT/commands"
README="$ROOT/README.md"
CHEAT="$ROOT/cheatsheet.md"

# All node README paths except the template, sorted by command path.
nodes() { find "$CMD" -mindepth 2 -name README.md -not -path "$CMD/_TEMPLATE/*" | sort; }

node_path() { sed "s|$CMD/||; s|/README.md||" <<<"$1"; }   # commands/pr/create/README.md -> pr/create
is_done()   { ! grep -q '<!-- STUB -->' "$1"; }
oneliner()  { sed -n 's/^> \*\*One-liner:\*\* //p' "$1" | head -1; }

# --- progress block (top-level groups only; promoted subs roll up) ------
# Portable: iterate immediate subdirs via a glob (no GNU `find -printf`).
progress() {
  echo "<!-- BEGIN PROGRESS (generated by scripts/gen-index.sh) -->"
  local total=0 done=0 d r mark
  for d in "$CMD"/*/; do
    r="$(basename "$d")"
    [[ "$r" == _* ]] && continue          # skip _TEMPLATE and other _meta dirs
    total=$((total+1)); mark=" "
    if [[ -f "$CMD/$r/README.md" ]] && is_done "$CMD/$r/README.md"; then mark="x"; done=$((done+1)); fi
    echo "- [$mark] $r"
  done
  echo
  echo "_Progress: $done / $total groups._"
  echo "<!-- END PROGRESS -->"
}

# Replace the marked block in README.md in place.
tmp="$(mktemp)"
awk -v repl="$(progress)" '
  /<!-- BEGIN PROGRESS/ {print repl; skip=1}
  /<!-- END PROGRESS -->/ {skip=0; next}
  !skip
' "$README" > "$tmp" && mv "$tmp" "$README"
echo "updated $README"

# --- cheatsheet (one block per node, in command-path order) -------------
{
  echo "# gh cheatsheet"
  echo "<!-- GENERATED by scripts/gen-index.sh — do not edit. Edit the node READMEs instead. -->"
  echo
  for n in $(nodes); do
    p="$(node_path "$n")"; one="$(oneliner "$n")"
    if is_done "$n"; then echo "## gh ${p//\// }"; echo "${one:-_TODO_}"; echo
    else echo "## gh ${p//\// } _(stub)_"; echo; fi
  done
} > "$CHEAT"
echo "updated $CHEAT"
```

- [ ] **Step 2: Make it executable**

```bash
chmod +x scripts/gen-index.sh
```

- [ ] **Step 3: Commit**

```bash
git add scripts/gen-index.sh
git commit -m "feat: gen-index.sh — regenerate README progress + cheatsheet from tree"
```

### Task 0.7: Generate the skeleton and land the "scaffold" PR

This is the **scaffold-first** stage: one PR that lands the complete empty tree.

**Files:**
- Modify: `README.md` (replace the bootstrapping stub with the dashboard shell + markers)
- Create (generated): `commands/<all groups>/…`, `commands/_inventory.md`, `cheatsheet.md`

- [ ] **Step 1: Replace `README.md` with the dashboard shell** (static prose + the marker block `gen-index.sh` fills)

````markdown
# gh-mastery

My journey to mastering the **GitHub CLI** (`gh`) — every command group, mapped,
explained, and drilled. Built *with* `gh`: every issue, PR, milestone, and release
here is a worked example of the command it documents.

- **Spec:** [`docs/superpowers/specs/2026-05-26-gh-mastery-design.md`](docs/superpowers/specs/2026-05-26-gh-mastery-design.md)
- **RFC:** [Discussion #1](https://github.com/borahanmirzaii/gh-mastery/discussions/1)
- **Board:** [Project #14](https://github.com/users/borahanmirzaii/projects/14)
- **Sandbox:** [`gh-mastery-sandbox`](https://github.com/borahanmirzaii/gh-mastery-sandbox) — drill playground.

## How to use this repo

`commands/` mirrors `gh`'s own command tree. Each node is a folder with three files:
**`README.md`** (reference), **`DRILLS.md`** (hands-on, against the sandbox),
**`RECALL.md`** (Q&A self-test). Read → drill → recall. Deeper folders = meatier
subcommands. The progress list and `cheatsheet.md` are generated — see
`scripts/gen-index.sh`.

## Progress

<!-- BEGIN PROGRESS (generated by scripts/gen-index.sh) -->
<!-- END PROGRESS -->

## Environment

`gh` 2.92.0 · branch model `dev` → `main`.
````

- [ ] **Step 2: Run the generators**

```bash
scripts/scaffold.sh
scripts/gen-index.sh
```

- [ ] **Step 3: Verify the skeleton** — every group has a 3-file node, promotions exist, index populated

Run:
```bash
echo "groups: $(find commands -mindepth 1 -maxdepth 1 -type d -not -name '_*' | wc -l)"
echo "stub READMEs: $(grep -rl '<!-- STUB -->' commands --include=README.md | wc -l)"
ls commands/pr commands/pr/create commands/api          # promoted + flat-node spot check
grep -c '^- \[ \]' README.md                            # all groups unchecked initially
```
Expected: ~33 groups; every node README still a stub; `commands/pr/create/` exists; `commands/api/` exists with no subdirs; the progress list shows every group as `[ ]`.

- [ ] **Step 4: Verify idempotency** — re-running creates nothing new

Run: `scripts/scaffold.sh && git status --porcelain commands | grep -v '_inventory.md' | wc -l`
Expected: `0` (only `_inventory.md` may differ, and only after a real `gh` upgrade).

- [ ] **Step 5: Commit + open the scaffold PR**

```bash
git add README.md cheatsheet.md commands/
git commit -m "feat: scaffold the full gh command tree (empty nodes + index)"
git push -u origin dev
```

(Working directly on `dev` for the infra phase is fine; the per-group authoring loop branches off `dev` per node.)

### Task 0.8: File all issues (groups + scaffold + concepts + Pages)

**Files:** none (remote only). Capture each issue number from output.

- [ ] **Step 1: File the 13 Core (M1) issues**

```bash
for G in auth repo issue pr label project release run workflow search api secret variable; do
  gh issue create \
    --title "Master \`gh $G\`" \
    --body "Author \`commands/$G/\` (group node + any promoted-subcommand nodes) per the spec. Source: \`gh $G --help\` + https://cli.github.com/manual/gh_$G . Verify examples against the sandbox (namespace \`zz-$G-*\`). Follow the Authoring Procedure in docs/superpowers/plans/2026-05-26-gh-mastery.md." \
    --label "kind:command" \
    --milestone "M1 — Core (daily drivers)" \
    --project "gh-mastery" \
    --assignee @me
done
```

- [ ] **Step 2: File the 19 Long-tail (M2) issues**

```bash
for G in browse codespace gist org status alias config completion extension gpg-key ssh-key attestation ruleset agent-task copilot skill cache preview licenses; do
  gh issue create \
    --title "Master \`gh $G\`" \
    --body "Author \`commands/$G/\` (group node + any promoted-subcommand nodes) per the spec. Source: \`gh $G --help\` + https://cli.github.com/manual/gh_$G . Verify examples against the sandbox (namespace \`zz-$G-*\`). Follow the Authoring Procedure in docs/superpowers/plans/2026-05-26-gh-mastery.md." \
    --label "kind:command" \
    --milestone "M2 — Long tail" \
    --project "gh-mastery" \
    --assignee @me
done
```

- [ ] **Step 3: File the 5 concept (M3) issues**

```bash
for C in rest-vs-graphql actions-model rulesets attestations-slsa projects-v2-data-model; do
  gh issue create \
    --title "Concept: $C" \
    --body "Write \`concepts/$C.md\` — the GitHub concept behind the related commands, per the spec." \
    --label "kind:concept" \
    --milestone "M3 — Concepts" \
    --project "gh-mastery" \
    --assignee @me
done
```

- [ ] **Step 4: File the Pages (M4) issue**

```bash
gh issue create \
  --title "Publish curriculum to GitHub Pages" \
  --body "Add .github/workflows/pages.yml to render commands/ + concepts/ markdown to a Pages site. Teaches the gh workflow/run + Actions deploy model. Per spec §7." \
  --label "kind:infra" \
  --milestone "M4 — Publish (Pages)" \
  --project "gh-mastery" \
  --assignee @me
```

- [ ] **Step 5: Verify counts**

Run: `gh issue list --label kind:command --json number --jq 'length'` → Expected `32`.
Run: `gh project item-list 14 --owner borahanmirzaii --format json --jq '.items | length'` → Expected `38` (32 command + 5 concept + 1 Pages).

### Task 0.9: Release M0

- [ ] **Step 1: Promote `dev` → `main` and tag the infra checkpoint**

```bash
gh pr create --base main --head dev --title "Release v0.1.0 — infra + scaffold" --fill
PR=$(gh pr view --json number --jq .number)
gh pr merge "$PR" --squash --delete-branch=false
gh release create v0.1.0 --target main --generate-notes --title "v0.1.0 — infra + scaffold"
```

- [ ] **Step 2: Verify the release**

Run: `gh release view v0.1.0 --json tagName --jq .tagName`
Expected: `v0.1.0`

---

## The Authoring Procedure (shared by every command-group task in Phase 1+)

Every group task in Phase 1 and Phase 2 follows these exact steps. Parameters per group come from the table in each phase: **`<group>`**, **suggested promotions**, and **must-cover gotchas**. The empty nodes already exist (Task 0.7) — this fills them.

- [ ] **A. Create the linked branch/worktree from the group's issue**

```bash
gh issue develop <issue-num> --base dev --checkout
```

- [ ] **B. Gather sources**

```bash
gh <group> --help
for SUB in $(gh <group> --help | awk '/^[A-Z][A-Z ]*COMMANDS/{f=1;next}/^[A-Z]/{f=0}f && /^[[:space:]]+[a-z]/{sub(/:$/,"",$1);print $1}'); do
  echo "== $SUB =="; gh <group> "$SUB" --help
done
```
Also read the manual page `https://cli.github.com/manual/gh_<group>` (WebFetch). Where the installed `--help` and the manual disagree, the installed version wins (note it as a gotcha).

- [ ] **C. Confirm promotions, then fill the subtree**

Compare the suggested promotions for this group (phase table) against the **promotion rule of thumb**. If a subcommand should be promoted but has no node, add it to `commands/_promotions.txt` and re-run `scripts/scaffold.sh` (creates only the new node). Then edit every file in `commands/<group>/` and each promoted `commands/<group>/<sub>/`:
- Replace every `<...>` placeholder and **delete the `<!-- STUB -->` line** from each authored README.
- README covers all subcommands (promoted ones via the Subcommands table linking to their node; shallow ones inline as sections), every flag that matters, ≥3 examples, and the must-cover gotchas.
- DRILLS run against the sandbox using the `zz-<group>-*` namespace and have a checkable verify + cleanup.
- RECALL has ≥5 Q&A covering the gotchas + key flags.

- [ ] **D. Verify the examples actually work**

Run each example/drill command against `borahanmirzaii/gh-mastery-sandbox` (read-only examples can target any repo). Fix any that don't behave as written. Confirm no `zz-<group>-*` leftovers remain after drill cleanup.
Expected: every documented command produces the stated outcome.

- [ ] **E. Regenerate the index (do NOT hand-edit README/cheatsheet)**

```bash
scripts/gen-index.sh
```
This ticks `<group>` in the README progress block and refreshes its `cheatsheet.md` block from the now-filled node.

- [ ] **F. Commit, push, open the draft PR**

```bash
git add commands/<group>/ README.md cheatsheet.md commands/_promotions.txt
git commit -m "feat(<group>): reference + drills + recall for gh <group>"
git push -u origin "$(git branch --show-current)"
gh pr create --base dev --fill --draft
```
The PR auto-includes `Closes #<issue-num>` because the branch came from `gh issue develop`.

- [ ] **G. Review + merge (Lead)**

```bash
gh pr ready                              # flip out of draft when done
gh pr merge <pr-num> --squash --delete-branch
scripts/gen-index.sh && git commit -am "chore: refresh index after #<issue-num>" || true
```
Issue auto-closes. The post-merge `gen-index.sh` reconciles the dashboard across concurrently-merged PRs (the conflict-killer for the generated files).

---

## Phase 1 — Core command groups (Milestone 1)

Each task = the Authoring Procedure (A–G) with these parameters. Do them in listed order (roughly the solo-builder Loop order). Run in **waves of ~5–6 workers** (spec §6.1). `<issue-num>` is the issue filed in Task 0.8 for that group.

| Task | `<group>` | Suggested promotions | Must-cover gotchas |
|---|---|---|---|
| 1.1 | `auth` | `login`, `refresh` | `gh auth refresh -s` is identity-ambiguous on multi-account; `auth status` shows token scopes; `auth switch` vs per-identity `GH_TOKEN`. |
| 1.2 | `repo` | `create`, `edit` | `gh repo create --source=. --push` vs `--clone`; `gh repo edit` feature flags; no `gh repo transfer` (use `gh api -X POST .../transfer`). |
| 1.3 | `issue` | `create`, `develop` | Body links must be absolute URLs; `gh issue develop` creates the server-side branch link; `--body-file -` reads stdin. |
| 1.4 | `pr` | `create`, `merge`, `review` | `--fill` autofills from commits; `Closes #N` in body auto-closes the issue; `--squash --delete-branch` is the convention. |
| 1.5 | `label` | _(none — shallow)_ | `gh label clone <repo>` copies an entire set; `--force` upserts. |
| 1.6 | `project` | `item-list`, `field-list` | v2 only; built-in Status field options edit via `updateProjectV2Field` (GraphQL), not `field-edit`; `--owner @me`. |
| 1.7 | `release` | `create` | `--generate-notes`; `--target`; asset upload `file#"Label"` syntax; `--verify-tag`. |
| 1.8 | `run` | `watch` | `gh run watch --exit-status` for CI gating; `--log-failed`; `view --json`. |
| 1.9 | `workflow` | `run` | `workflow run -f key=val` dispatch inputs; `--ref`; enable/disable. |
| 1.10 | `search` | `code` | qualifier syntax vs flags; `--json` piping to `jq`; `search code` needs auth scope. |
| 1.11 | `api` | _(none — no subcommands)_ | `-f` (string) vs `-F` (typed/@file); `--paginate`; `{owner}/{repo}` placeholders; `--jq`; `graphql` subcommand. |
| 1.12 | `secret` | `set` | repo vs org vs env scope; `--app actions/codespaces/dependabot`; values never echoed. |
| 1.13 | `variable` | _(none — shallow)_ | like `secret` but non-encrypted; `--env`/`--org` scoping. |

- [ ] **After all 13 merge: Release M1**

```bash
gh pr create --base main --head dev --title "Release v0.2.0 — Core command groups" --fill
gh pr merge "$(gh pr view --json number --jq .number)" --squash --delete-branch=false
gh release create v0.2.0 --target main --generate-notes --title "v0.2.0 — Core command groups"
```

---

## Phase 2 — Long-tail command groups (Milestone 2)

Same Authoring Procedure (A–G), one task per group. Order is not critical; grouped by theme. Most long-tail groups are shallow → no promotions.

| Task | `<group>` | Suggested promotions | Must-cover gotchas |
|---|---|---|---|
| 2.1 | `browse` | _(none)_ | `--no-browser` prints URL; `-s` for settings; deep-links to files/lines. |
| 2.2 | `codespace` | `ssh`, `ports` | `cs` alias; `cp`; billing implications. |
| 2.3 | `gist` | `create` | secret vs public default; `gist create -` from stdin; `--web`. |
| 2.4 | `org` | _(none — `list` only)_ | most org ops live under `gh api`. |
| 2.5 | `status` | _(none)_ | cross-repo digest; `-e` to exclude; auth-scoped. |
| 2.6 | `alias` | _(none)_ | `alias set` with `--shell`; expansion with `$1`; `co` is a built-in alias example. |
| 2.7 | `config` | _(none)_ | `config set` keys (editor, pager, prompt, git_protocol); host-scoped. |
| 2.8 | `completion` | _(none)_ | `-s zsh/bash/fish`; where to source it. |
| 2.9 | `extension` | `install` | `extension install owner/repo`; `gh ext` alias; `--precompiled`. |
| 2.10 | `gpg-key` | _(none)_ | add/list/delete; relation to verified commits. |
| 2.11 | `ssh-key` | _(none)_ | add `--type authentication/signing`; relation to SSH remotes. |
| 2.12 | `attestation` | `verify` | `verify`/`download`; SLSA provenance; needs `concepts/attestations-slsa.md`. |
| 2.13 | `ruleset` | _(none — read-only in CLI)_ | `ruleset list/view/check`; needs `concepts/rulesets.md`. |
| 2.14 | `agent-task` | _(none)_ | preview; create/list/view agent tasks. |
| 2.15 | `copilot` | _(none)_ | preview; launches Copilot CLI; auth/subscription note. |
| 2.16 | `skill` | _(none)_ | preview; install/manage agent skills. |
| 2.17 | `cache` | _(none)_ | `cache list/delete --all`; Actions cache scope. |
| 2.18 | `preview` | _(none)_ | `preview` feature-flag mechanics. |
| 2.19 | `licenses` | _(none)_ | `licenses list/view`; third-party license info. |

Fold the help-topics (`formatting`, `exit-codes`, `environment`, `accessibility`) into the relevant pages (e.g. `--json`/`--jq`/`--template` formatting under `api` and `search`) rather than separate nodes.

- [ ] **After all 19 merge: Release M2** (`v0.3.0`, same pattern.)

---

## Phase 3 — Concepts (Milestone 3)

One task per concept doc (issues from Task 0.8 Step 3). Each: branch from issue, write `concepts/<name>.md` (plain explainer), add cross-links from the command READMEs that need it, verify links, PR, squash-merge.

| Task | `concepts/<name>.md` | Linked from |
|---|---|---|
| 3.1 | `rest-vs-graphql` | `api` |
| 3.2 | `actions-model` | `run`, `workflow`, `cache` |
| 3.3 | `rulesets` | `ruleset` |
| 3.4 | `attestations-slsa` | `attestation` |
| 3.5 | `projects-v2-data-model` | `project` |

- [ ] **After all 5 merge: Release M3** (`v0.4.0`.)

---

## Phase 4 — Publish to Pages (Milestone 4)

### Task 4.1: Pages workflow

**Files:**
- Create: `.github/workflows/pages.yml`

- [ ] **Step 1: Branch from the Pages issue** — `gh issue develop <num> --base dev --checkout`.

- [ ] **Step 2: Write `.github/workflows/pages.yml`** — a workflow that renders `README.md`, `commands/**`, and `concepts/**` to a static site and deploys via `actions/deploy-pages`. (Concrete YAML finalized in the task using the then-current action versions; verify with `gh workflow view` + `gh run watch --exit-status`.)

- [ ] **Step 3: Enable Pages source = GitHub Actions**

```bash
gh api -X POST repos/borahanmirzaii/gh-mastery/pages -f build_type=workflow 2>/dev/null || gh api -X PUT repos/borahanmirzaii/gh-mastery/pages -f build_type=workflow
```

- [ ] **Step 4: Verify deploy**

Run: `gh run list --workflow pages.yml --limit 1`, then `gh run watch <id> --exit-status`, then `gh api repos/borahanmirzaii/gh-mastery/pages --jq .html_url`.
Expected: a live Pages URL.

- [ ] **Step 5: PR + merge + Release `v1.0.0`** (`--generate-notes`).

---

## Self-Review

**Spec coverage:**
- §2 build-with-gh → every remote step uses `gh`; scaffold/index scripts are themselves `gh` lessons. ✓
- §3 mastery unit (node = reference/drills/recall; group + promoted subcommands; promotion rule) → Task 0.4 template + 0.5 promotions + Authoring Procedure C. ✓
- §4 recursive tree + generated index → Tasks 0.5 (`scaffold.sh`/`_inventory.md`), 0.6 (`gen-index.sh`), 0.7 (skeleton). ✓
- §5 sandbox → Task 0.1; drills target it with `zz-<group>-*` namespacing (Procedure C/D). ✓
- §6 milestones + issue-per-group + subtree-per-PR → Tasks 0.3, 0.8; Phases 1–3. ✓
- §6.1 build model (scaffold-first, parallel waves, generated index, namespaced drills) → Task 0.7 + Phase 1 wave note + Procedure E/G + Conventions. ✓
- §7 Pages → Phase 4. ✓
- §8 labels → Task 0.2. ✓
- §9 sources (installed wins) → Procedure B. ✓
- §10 out-of-scope → no tasks touch the terminal stack / Anki / CI harness. ✓

**Placeholder scan:** the only deferred concrete artifact is the Pages YAML (Task 4.1 Step 2) — flagged explicitly because action versions should be pinned at execution time. Both generator scripts are written in full; the `<...>` tokens in `_TEMPLATE/` are intentional fill-in markers, not plan gaps.

**Consistency:** milestone titles match across Task 0.3, the `--milestone` flags (0.8), and the release phases. Group list (13 core + 19 tail) matches spec §6 and the discovery output (`co` excluded as a built-in alias). Counts: 32 command + 5 concept + 1 Pages = 38 (Task 0.8 Step 5). The discovery awk in Procedure B matches `scaffold.sh`'s `subcommands()`; `<!-- STUB -->` is written by the template (0.4), consumed by `gen-index.sh` (0.6), and deleted in Procedure C — consistent across tasks.
