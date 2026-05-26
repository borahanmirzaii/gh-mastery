# gh-mastery — Design Spec

**Date:** 2026-05-26
**Status:** Approved (defaults bundle + Q6=gh-concepts-only, confirmed in chat)
**RFC:** https://github.com/borahanmirzaii/gh-mastery/discussions/1
**Repo:** https://github.com/borahanmirzaii/gh-mastery (public)

## 1. Goal

Become fluent in the GitHub CLI (`gh`) — every command group, its subcommands,
the flags that matter, the `api`/GraphQL escape hatch, and the GitHub concepts
behind them. The deliverable is a single self-contained repo that doubles as a
reference handbook and a hands-on training program.

**Success criteria:** every command group in `gh` 2.92.0 has a reference page, a
drills page, and a recall page; the daily-driver groups are done first; the
curriculum is browsable as a published site; and the repo's own history (issues,
PRs, releases) is a worked example of the commands it documents.

## 2. Guiding principle — build it *with* `gh`

The repo is constructed using the very commands it teaches. The RFC is
`gh api graphql`; the tracker is `gh project`; each command group is a
`gh issue` inside a `gh milestone`; finished work is a `gh pr`; each completed
milestone is a `gh release`. Reading this repo's GitHub history *is* a `gh`
tutorial. This is a hard constraint, not decoration: we never reach for the web
UI when `gh` can do the job (web UI only where `gh` genuinely can't — e.g.
pinning a Discussion, creating Discussion categories).

## 3. The mastery unit (Q1 = reference + drills, Q5 = recall)

Every command group is a directory under `commands/<group>/` holding three files.
A canonical `commands/_TEMPLATE/` defines the exact shape; every group conforms.

### 3.1 `README.md` — annotated reference
- **What it does** — one or two sentences.
- **When you reach for it** — the real workflow context (ties to the solo-builder loop where relevant).
- **Subcommands** — a table: subcommand → one-line purpose.
- **Key flags** — the flags that matter, each explained (not a raw `--help` dump, but thorough — no important flag silently dropped).
- **Examples** — 3-5 real, copy-paste-able commands drawn from actual workflows, each with a one-line "what this does."
- **Gotchas** — the friction points (e.g. issue body links must be absolute URLs; `gh auth refresh -s` is identity-ambiguous on multi-account).
- **Concepts** — links into `concepts/` where a GitHub concept underlies the command.
- **Sources** — link to the official manual page + the exact `gh <group> --help` invocation.

### 3.2 `DRILLS.md` — hands-on practice
- A numbered ladder of tasks, easy → hard, each with: **goal**, the **command to discover** (the answer is collapsible/below, not handed over up front), and a **verify** step with the expected observable outcome.
- All drills mutate the **sandbox repo** (see §5), never a real project.
- A closing **boss drill** that chains several subcommands into one realistic mini-workflow.

### 3.3 `RECALL.md` — spaced-repetition self-test
- Plain Q&A prompts (question, then answer below a divider) covering the flags and gotchas from the reference. Cheap to write, doubles as a quiz. (Anki export is explicitly out of scope for v1.)

## 4. Repository layout (Q3 = in-repo markdown is source of truth)

```
gh-mastery/
├── README.md                       # mission, how-to-use, progress dashboard, links
├── cheatsheet.md                   # aggregated terse quick-ref across all groups
├── commands/
│   ├── _TEMPLATE/                  # canonical README/DRILLS/RECALL shape
│   ├── auth/  repo/  issue/  pr/  release/  project/  label/
│   ├── run/  workflow/  api/  search/  secret/  variable/        # ← Milestone 1
│   └── browse/ codespace/ gist/ org/ status/ alias/ config/ ...  # ← Milestone 2
├── concepts/                       # Q6: GitHub concepts behind the commands
│   ├── rest-vs-graphql.md
│   ├── actions-model.md
│   ├── rulesets.md
│   ├── attestations-slsa.md
│   └── projects-v2-data-model.md
├── docs/superpowers/
│   ├── specs/2026-05-26-gh-mastery-design.md
│   └── plans/2026-05-26-gh-mastery-plan.md
└── .github/workflows/pages.yml     # added in the Pages milestone
```

Content lives in version-controlled markdown so every command group flows through
the full **issue → branch → PR → merge** loop — which is itself a lesson. The Wiki
is reserved for freeform scratch only; Pages publishes the markdown (§7).

## 5. Drill sandbox (Q4)

A dedicated throwaway repo **`borahanmirzaii/gh-mastery-sandbox`** (public) is the
playground where drills create/edit/delete issues, labels, releases, etc. It can be
reset or recreated at will without polluting `gh-mastery`'s history. Its creation is
the first infra issue. DRILLS pages assume the learner is targeting the sandbox
(`--repo borahanmirzaii/gh-mastery-sandbox`).

## 6. Coverage, milestones, and the issue model (Q2 = core-first)

One **issue per command group**; one **PR per issue**; merging a PR delivers that
group's three files. Issues are grouped into milestones:

- **Milestone 0 — Infra:** sandbox repo, `_TEMPLATE/`, label taxonomy, cheatsheet skeleton, README dashboard.
- **Milestone 1 — Core (daily drivers):** `auth`, `repo`, `issue`, `pr`, `label`, `project`, `release`, `run`, `workflow`, `search`, `api`, `secret`, `variable`. Ordered roughly by where they appear in the solo-builder loop.
- **Milestone 2 — Long tail:** `browse`, `codespace`, `gist`, `org`, `status`, `alias`, `config`, `completion`, `extension`, `gpg-key`, `ssh-key`, `attestation`, `ruleset`, `agent-task`, `copilot`, `skill`, `cache`, `preview`, `licenses`, plus the help topics (`formatting`, `exit-codes`, `environment`, `accessibility`).
- **Milestone 3 — Concepts:** the `concepts/` docs.
- **Milestone 4 — Publish to Pages.**

Each closed milestone ships a `gh release` (`--generate-notes`) marking that chunk
as "learned."

## 7. Publishing (Pages) — later milestone

Milestone 4 adds `.github/workflows/pages.yml` to render the markdown into a
browsable site via GitHub Pages — itself a worked lesson in `gh workflow` /
`gh run` and the Actions deploy model. Not built until the content milestones land.

## 8. Labels

Lean taxonomy (kind + the RFC marker); milestones carry the phase, the Project
board carries status — so no redundant `phase:`/`status:` labels.

- `kind:command` — a command-group learning issue (produces README + DRILLS + RECALL).
- `kind:concept` — a `concepts/` doc.
- `kind:infra` — sandbox, template, tooling, Pages, cheatsheet.
- `meta` — RFC/spec/plan/process issues.

## 9. Sources

For each group: the **official manual** (`https://cli.github.com/manual/gh_<group>`)
plus the locally installed `gh <group> --help` (canonical for 2.92.0). The reference
synthesizes both; where they disagree, the installed version wins and the gotcha is noted.

## 10. Out of scope (Q6 = gh-concepts-only)

- The surrounding terminal stack (WezTerm / Zellij / Claude Code) — already covered by the `solo-builder-flow` skill.
- Anki/external spaced-repetition tooling (v1 uses in-repo `RECALL.md`).
- Automated test harness for drills — drills self-verify via expected output, not CI.

## 11. Open risks

- `gh` evolves; pages are pinned to 2.92.0 and dated. A future "refresh pass" is a possible Milestone 5.
- Manual content may lag the installed binary — mitigated by §9 (installed version wins).
