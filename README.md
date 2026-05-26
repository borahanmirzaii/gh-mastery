# gh-mastery

My journey to mastering the **GitHub CLI** (`gh`) — every command and subcommand,
mapped, explained, and drilled until it's muscle memory.

> **Meta:** this repo is built *with* `gh`. Every issue, label, milestone, project
> card, discussion reply, and release here is itself a worked example of the command
> it documents. Learn `gh` by watching `gh` build its own curriculum.

## Status

🟡 **Bootstrapping.** The repo exists; the *shape* of the curriculum (file format,
coverage order, how we drill and verify) is being decided in the open — see the
pinned **Discussion** before assuming any structure.

## How this is organized

- **Discussion** — the RFC thread where we converge on what "mastering" means here.
- **Project board** — the journey tracker; one card per command group.
- **Milestones + Issues** — systematic coverage, one issue per command group.
- **Releases** — checkpoints marking chunks of the surface as "learned."

_Structure of the `commands/` tree is intentionally left undecided until the
Discussion resolves — see the spec under `docs/` once it lands._

## Environment

- `gh` version: 2.92.0
- Surface to cover: ~30 top-level command groups + the `api` / GraphQL escape hatch.
