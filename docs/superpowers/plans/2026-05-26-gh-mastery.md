# gh-mastery Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single repo that teaches every `gh` command group via a reference page, a drills page, and a recall page each — built using `gh` itself, shipped milestone by milestone.

**Architecture:** Infra-first (Phase 0: sandbox repo, labels, milestones, canonical `_TEMPLATE/`, README dashboard, and all command-group issues filed via `gh`). Then a repeating authoring loop (Phase 1+): one issue per command group → linked branch → fill the three files from the template using `gh <group> --help` + the manual → verify examples against the sandbox → PR → squash-merge. Each closed milestone ships a `gh release`.

**Tech Stack:** `gh` 2.92.0, GitHub (Issues/PRs/Projects/Discussions/Milestones/Releases/Pages), GraphQL via `gh api graphql`, Markdown. No application code; "tests" are existence/link checks plus running the documented example commands against the sandbox repo.

**Spec:** `docs/superpowers/specs/2026-05-26-gh-mastery-design.md` · **RFC:** Discussion #1 · **Board:** Project #14

---

## Conventions used throughout

- **Repo:** `borahanmirzaii/gh-mastery`. **Sandbox:** `borahanmirzaii/gh-mastery-sandbox`.
- **Branch model:** branch from `dev`, PR into `dev`; promote `dev`→`main` for releases.
- **Every PR body** contains `Closes #<issue>` (auto-added by `gh issue develop`).
- **Identity** is already pinned (`borahanmirzaii`); the PreToolUse hook enforces it — don't pre-check.
- **"Verify" steps** for content tasks mean: the file exists, internal links resolve, and the example commands in it actually run against the sandbox.

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

- [ ] **Step 3: Enable issues so issue/label/pr drills have a target**

```bash
gh repo edit borahanmirzaii/gh-mastery-sandbox --enable-issues --enable-projects
```

(No commit — remote-only task.)

### Task 0.2: Create the label taxonomy

**Files:** none (remote only)

- [ ] **Step 1: Create the four labels**

```bash
gh label create "kind:command" --color 1D76DB --description "A command-group learning issue (README+DRILLS+RECALL)" --force
gh label create "kind:concept" --color 5319E7 --description "A concepts/ explainer doc" --force
gh label create "kind:infra"   --color 0E8A16 --description "Sandbox, template, tooling, Pages, cheatsheet" --force
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

- [ ] **Step 1: Write `commands/_TEMPLATE/README.md`**

````markdown
# `gh <group>`

> **One-liner:** <what this command group does, in one sentence>.

## When you reach for it

<The real workflow moment. Tie to the solo-builder loop where it fits,
e.g. "Step 1 of the Loop — you draft the brief as an issue body.">

## Subcommands

| Subcommand | Purpose |
|---|---|
| `gh <group> <sub>` | <one line> |

## Key flags

Only the flags that matter, each with *when* to use it (not a raw `--help` dump).

- `--<flag>` — <meaning; when you'd use it>.

## Examples

```bash
# <what this does>
gh <group> <sub> --<flag> <value>
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

**Verify:** <end state you can check>.
````

- [ ] **Step 3: Write `commands/_TEMPLATE/RECALL.md`**

````markdown
# Recall — `gh <group>`

Spaced-repetition self-test. Cover the answer, recall it, then check.

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
git commit -m "feat: canonical command-group template (reference/drills/recall)"
```

### Task 0.5: README dashboard + cheatsheet skeleton

**Files:**
- Modify: `README.md`
- Create: `cheatsheet.md`

- [ ] **Step 1: Replace `README.md` with the dashboard**

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

Each command group lives in `commands/<group>/` with three files:
**`README.md`** (reference), **`DRILLS.md`** (hands-on, against the sandbox),
**`RECALL.md`** (Q&A self-test). Read → drill → recall.

## Progress

### M1 — Core (daily drivers)
- [ ] auth · [ ] repo · [ ] issue · [ ] pr · [ ] label · [ ] project · [ ] release
- [ ] run · [ ] workflow · [ ] search · [ ] api · [ ] secret · [ ] variable

### M2 — Long tail
- [ ] browse · [ ] codespace · [ ] gist · [ ] org · [ ] status · [ ] alias · [ ] config
- [ ] completion · [ ] extension · [ ] gpg-key · [ ] ssh-key · [ ] attestation
- [ ] ruleset · [ ] agent-task · [ ] copilot · [ ] skill · [ ] cache · [ ] preview · [ ] licenses

### M3 — Concepts
- [ ] rest-vs-graphql · [ ] actions-model · [ ] rulesets · [ ] attestations-slsa · [ ] projects-v2-data-model

### M4 — Publish (Pages)
- [ ] Pages workflow

## Environment

`gh` 2.92.0 · branch model `dev` → `main`.
````

- [ ] **Step 2: Create `cheatsheet.md` skeleton**

````markdown
# gh cheatsheet

Terse quick-reference, one block per command group. Filled in as each group lands.

<!-- Each command-group PR appends its block here. -->
````

- [ ] **Step 3: Verify links resolve**

Run: `grep -o '](docs/[^)]*' README.md` and confirm `docs/superpowers/specs/2026-05-26-gh-mastery-design.md` exists with `ls docs/superpowers/specs/`.
Expected: file present.

- [ ] **Step 4: Commit**

```bash
git add README.md cheatsheet.md
git commit -m "feat: README progress dashboard + cheatsheet skeleton"
```

### Task 0.6: File all command-group issues (M1 + M2)

**Files:** none (remote only). Run from a clean `dev` checkout. Capture each issue number from output.

- [ ] **Step 1: File the 13 Core (M1) issues**

```bash
for G in auth repo issue pr label project release run workflow search api secret variable; do
  gh issue create \
    --title "Master \`gh $G\`" \
    --body "Produce \`commands/$G/\` (README + DRILLS + RECALL) per the spec. Source: \`gh $G --help\` + https://cli.github.com/manual/gh_$G . Verify examples against the sandbox. Follow the Authoring Procedure in docs/superpowers/plans/2026-05-26-gh-mastery.md." \
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
    --body "Produce \`commands/$G/\` (README + DRILLS + RECALL) per the spec. Source: \`gh $G --help\` + https://cli.github.com/manual/gh_$G . Verify examples against the sandbox. Follow the Authoring Procedure in docs/superpowers/plans/2026-05-26-gh-mastery.md." \
    --label "kind:command" \
    --milestone "M2 — Long tail" \
    --project "gh-mastery" \
    --assignee @me
done
```

- [ ] **Step 3: Verify counts**

Run: `gh issue list --label kind:command --json number --jq 'length'`
Expected: `32`.

### Task 0.7: File the Concepts (M3) and Pages (M4) issues

**Files:** none (remote only)

- [ ] **Step 1: File the 5 concept issues**

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

- [ ] **Step 2: File the Pages issue**

```bash
gh issue create \
  --title "Publish curriculum to GitHub Pages" \
  --body "Add .github/workflows/pages.yml to render commands/ + concepts/ markdown to a Pages site. Teaches the gh workflow/run + Actions deploy model. Per spec §7." \
  --label "kind:infra" \
  --milestone "M4 — Publish (Pages)" \
  --project "gh-mastery" \
  --assignee @me
```

- [ ] **Step 3: Verify the board is populated**

Run: `gh project item-list 14 --owner borahanmirzaii --format json --jq '.items | length'`
Expected: `38` (32 command + 5 concept + 1 Pages).

### Task 0.8: Release M0

- [ ] **Step 1: Promote `dev` → `main` and tag the infra checkpoint**

```bash
git push origin dev
gh pr create --base main --head dev --title "Release v0.1.0 — infra scaffold" --fill
PR=$(gh pr view --json number --jq .number)
gh pr merge "$PR" --squash --delete-branch=false
gh release create v0.1.0 --target main --generate-notes --title "v0.1.0 — infra scaffold"
```

- [ ] **Step 2: Verify the release**

Run: `gh release view v0.1.0 --json tagName --jq .tagName`
Expected: `v0.1.0`

---

## The Authoring Procedure (shared by every command-group task in Phase 1+)

Every group task in Phase 1 and Phase 2 follows these exact steps. Parameters per
group come from the table in each task: **`<group>`** and any **must-cover gotchas**.

- [ ] **A. Create the linked branch from the group's issue**

```bash
gh issue develop <issue-num> --base dev --checkout
```

- [ ] **B. Gather sources**

```bash
gh <group> --help
for SUB in $(gh <group> --help | sed -n '/COMMANDS/,/FLAGS/p' | awk '{print $1}' | grep -v -E 'COMMANDS|FLAGS|^$'); do echo "== $SUB =="; gh <group> "$SUB" --help; done
```
Also read the manual page `https://cli.github.com/manual/gh_<group>` (WebFetch). Where the installed `--help` and the manual disagree, the installed version wins (note it as a gotcha).

- [ ] **C. Instantiate the template**

```bash
mkdir -p commands/<group>
cp commands/_TEMPLATE/README.md commands/<group>/README.md
cp commands/_TEMPLATE/DRILLS.md commands/<group>/DRILLS.md
cp commands/_TEMPLATE/RECALL.md commands/<group>/RECALL.md
```
Fill in all three: replace every `<...>` placeholder. Reference page covers all subcommands and every flag that matters; ≥3 examples; the must-cover gotchas from the task table. Drills run against the sandbox and have a checkable verify. Recall has ≥5 Q&A covering the gotchas + key flags.

- [ ] **D. Verify the examples actually work**

Run each example/drill command against `borahanmirzaii/gh-mastery-sandbox`. Fix any that don't behave as the file claims. (Read-only examples can target any repo.)
Expected: every documented command produces the stated outcome.

- [ ] **E. Update the dashboard + cheatsheet**

Tick `[x] <group>` in `README.md`'s progress list and append a `## gh <group>` block to `cheatsheet.md`.

- [ ] **F. Commit, push, open the PR**

```bash
git add commands/<group>/ README.md cheatsheet.md
git commit -m "feat(<group>): reference + drills + recall for gh <group>"
git push -u origin "$(git branch --show-current)"
gh pr create --base dev --fill --draft
```
The PR auto-includes `Closes #<issue-num>` because the branch came from `gh issue develop`.

- [ ] **G. Review + merge (Lead)**

```bash
gh pr ready    # flip out of draft when done
gh pr merge <pr-num> --squash --delete-branch
```
Issue auto-closes; tick the board.

---

## Phase 1 — Core command groups (Milestone 1)

Each task = the Authoring Procedure (A–G) applied with these parameters. Do them
in listed order (roughly the solo-builder Loop order). `<issue-num>` is the issue
filed in Task 0.6 for that group.

| Task | `<group>` | Must-cover gotchas |
|---|---|---|
| 1.1 | `auth` | `gh auth refresh -s` is identity-ambiguous on multi-account; `auth status` shows token scopes; `auth switch` vs per-identity `GH_TOKEN`. |
| 1.2 | `repo` | `gh repo create --source=. --push` vs `--clone`; `gh repo edit` feature flags; no `gh repo transfer` (use `gh api -X POST .../transfer`). |
| 1.3 | `issue` | Body links must be absolute URLs; `gh issue develop` creates the server-side branch link; `--body-file -` reads stdin. |
| 1.4 | `pr` | `--fill` autofills from commits; `Closes #N` in body auto-closes the issue; `--squash --delete-branch` is the convention. |
| 1.5 | `label` | `gh label clone <repo>` copies an entire set; `--force` upserts. |
| 1.6 | `project` | v2 only; built-in Status field options edit via `updateProjectV2Field` (GraphQL), not `field-edit`; `--owner @me`. |
| 1.7 | `release` | `--generate-notes`; `--target`; asset upload `file#"Label"` syntax; `--verify-tag`. |
| 1.8 | `run` | `gh run watch --exit-status` for CI gating; `--log-failed`; `view --json`. |
| 1.9 | `workflow` | `workflow run -f key=val` dispatch inputs; `--ref`; enable/disable. |
| 1.10 | `search` | qualifier syntax vs flags; `--json` piping to `jq`; `search code` needs auth scope. |
| 1.11 | `api` | `-f` (string) vs `-F` (typed/@file); `--paginate`; `{owner}/{repo}` placeholders; `--jq`; `graphql` subcommand. |
| 1.12 | `secret` | repo vs org vs env scope; `--app actions/codespaces/dependabot`; values never echoed. |
| 1.13 | `variable` | like `secret` but non-encrypted; `--env`/`--org` scoping. |

(Tasks 1.1–1.13 each: run Authoring Procedure A–G with the row's `<group>` and gotchas.)

- [ ] **After all 13 merge: Release M1**

```bash
gh pr create --base main --head dev --title "Release v0.2.0 — Core command groups" --fill
gh pr merge "$(gh pr view --json number --jq .number)" --squash --delete-branch=false
gh release create v0.2.0 --target main --generate-notes --title "v0.2.0 — Core command groups"
```

---

## Phase 2 — Long-tail command groups (Milestone 2)

Same Authoring Procedure (A–G), one task per group, parameters below. Order is not
critical; suggested grouping by theme.

| Task | `<group>` | Must-cover gotchas |
|---|---|---|
| 2.1 | `browse` | `--no-browser` prints URL; `-s` for settings; deep-links to files/lines. |
| 2.2 | `codespace` | `cs` alias; `ssh`/`cp`/`ports`; billing implications. |
| 2.3 | `gist` | secret vs public default; `gist create -` from stdin; `--web`. |
| 2.4 | `org` | `org list` only; most org ops live under `gh api`. |
| 2.5 | `status` | cross-repo digest; `-e` to exclude; auth-scoped. |
| 2.6 | `alias` | `alias set` with `--shell`; expansion with `$1`; `co` is a built-in alias example. |
| 2.7 | `config` | `config set` keys (editor, pager, prompt, git_protocol); host-scoped. |
| 2.8 | `completion` | `-s zsh/bash/fish`; where to source it. |
| 2.9 | `extension` | `extension install owner/repo`; `gh ext` alias; `--precompiled`. |
| 2.10 | `gpg-key` | add/list/delete; relation to verified commits. |
| 2.11 | `ssh-key` | add `--type authentication/signing`; relation to SSH remotes. |
| 2.12 | `attestation` | `verify`/`download`; SLSA provenance; needs `concepts/attestations-slsa.md`. |
| 2.13 | `ruleset` | `ruleset list/view/check`; read-only in CLI; needs `concepts/rulesets.md`. |
| 2.14 | `agent-task` | preview; create/list/view agent tasks. |
| 2.15 | `copilot` | preview; launches Copilot CLI; auth/subscription note. |
| 2.16 | `skill` | preview; install/manage agent skills. |
| 2.17 | `cache` | `cache list/delete --all`; Actions cache scope. |
| 2.18 | `preview` | `preview` feature-flag mechanics. |
| 2.19 | `licenses` | `licenses list/view`; third-party license info. |

Also fold the help-topics (`formatting`, `exit-codes`, `environment`, `accessibility`)
into the relevant pages (e.g. `--json`/`--jq`/`--template` formatting under `api` and
`search`) rather than separate dirs.

- [ ] **After all 19 merge: Release M2** (`v0.3.0`, same pattern as above.)

---

## Phase 3 — Concepts (Milestone 3)

One task per concept doc (issues from Task 0.7). Each: branch from issue, write
`concepts/<name>.md` (plain explainer, link back from the command READMEs that need
it), verify cross-links, PR, squash-merge.

| Task | `concepts/<name>.md` | Anchored by commands |
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

- [ ] **Step 2: Write `.github/workflows/pages.yml`** — a workflow that renders `README.md`, `commands/**`, and `concepts/**` to a static site and deploys via `actions/deploy-pages`. (Concrete YAML to be finalized in the task using the then-current action versions; verify with `gh workflow view` + `gh run watch --exit-status`.)

- [ ] **Step 3: Enable Pages source = GitHub Actions**

```bash
gh api -X POST repos/borahanmirzaii/gh-mastery/pages -f build_type=workflow 2>/dev/null || gh api -X PUT repos/borahanmirzaii/gh-mastery/pages -f build_type=workflow
```

- [ ] **Step 4: Verify deploy**

Run: `gh run list --workflow pages.yml --limit 1` then `gh run watch <id> --exit-status`; then `gh api repos/borahanmirzaii/gh-mastery/pages --jq .html_url`.
Expected: a live Pages URL.

- [ ] **Step 5: PR + merge + Release `v1.0.0`** (`--generate-notes`).

---

## Self-Review

**Spec coverage:**
- §3 mastery unit (reference/drills/recall) → Task 0.4 template + Authoring Procedure C. ✓
- §4 layout → Tasks 0.4, 0.5, Phases 1–4. ✓
- §5 sandbox → Task 0.1; drills target it (Procedure D). ✓
- §6 milestones + issue-per-group → Tasks 0.3, 0.6, 0.7; Phases 1–3. ✓
- §7 Pages → Phase 4. ✓
- §8 labels → Task 0.2. ✓
- §9 sources (installed wins) → Procedure B. ✓
- §10 out-of-scope → no tasks touch the terminal stack / Anki / CI harness. ✓
- §2 build-with-gh → every remote step uses `gh`. ✓

**Placeholder scan:** The only deliberately deferred concrete artifact is the Pages YAML (Task 4.1 Step 2), because action versions should be pinned at execution time, not now — flagged explicitly, not a silent TODO. All infra/issue commands are concrete and runnable.

**Consistency:** milestone titles match between Task 0.3, the issue-creation `--milestone` flags (0.6/0.7), and the release phases. Group list (13 core + 19 tail) matches the spec §6 and the README dashboard. Issue count 32 (command) + 5 (concept) + 1 (Pages) = 38 matches Task 0.7 Step 3.
