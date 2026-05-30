# `gh issue create`

> **One-liner:** Open a new GitHub Issue with title, body, labels, assignees, milestone, and project — fully non-interactively from the command line.

## When you reach for it

This is **Step 1 of the solo-builder loop** — every piece of planned work starts as an issue body. You use `gh issue create` to file the brief without opening the browser. The `gh-mastery` project itself was bootstrapped with a batch loop of `gh issue create` calls (see `docs/superpowers/plans/2026-05-26-gh-mastery.md` Task 0.8) — every command-group task in this curriculum is an issue created this way.

Typical moments:
- Scripted batch issue creation (loop over a list of items).
- CI pipeline that files a bug report automatically on test failure.
- Personal workflow where you want to capture a brief in the repo immediately, before context is lost.

## Key flags

- `-t, --title string` — Issue title. Required non-interactively; if omitted, `gh` prompts.
- `-b, --body string` — Body text inline. For more than a sentence or two, prefer `--body-file`.
- `-F, --body-file file` — Read body from a file. Pass `-` to read from **stdin** (heredoc or pipe from a generator). This is the key to scripted rich-body issues.
- `-a, --assignee login` — Assign by GitHub login. `@me` self-assigns without hardcoding your username. Repeat the flag for multiple assignees, or comma-separate: `monalisa,hubot`.
- `-l, --label name` — Attach a label by name. Repeat the flag or comma-separate for multiple.
- `-m, --milestone name` — Attach to a milestone by **name** (not number).
- `-p, --project title` — Add to a Projects v2 board by title. Requires `project` scope (see Gotchas).
- `-T, --template name` — Pre-fill the body with an issue template by name.
- `-e, --editor` — Skip prompts and open your `$EDITOR`; first line = title, rest = body.
- `-w, --web` — Open the browser creation form instead; useful when you want to pick a template interactively.
- `--recover string` — Resume a failed creation run from saved draft input.

## Examples

```bash
# 1. Minimal non-interactive — title + body from flags
gh issue create \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --title "zz-issue-create-example: minimal issue" \
  --body "This issue was created entirely from CLI flags."

# 2. Body from stdin — multiline heredoc, with an absolute link
gh issue create \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --title "zz-issue-create-example: stdin body" \
  --body-file - <<'EOF'
## Problem

Something is broken in the widget.

## Acceptance criteria

- [ ] Widget loads without errors.

**Spec:** https://github.com/borahanmirzaii/gh-mastery/blob/main/docs/superpowers/specs/2026-05-26-gh-mastery-design.md
EOF

# 3. Fully-featured: label, assignee, milestone
gh issue create \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --title "zz-issue-create-example: full-featured" \
  --body "All fields set from flags." \
  --label "bug" \
  --label "help wanted" \
  --assignee @me

# 4. Batch creation — loop over a list (the pattern used in Task 0.8)
for GROUP in auth repo issue; do
  gh issue create \
    --repo borahanmirzaii/gh-mastery \
    --title "Master \`gh $GROUP\`" \
    --body "Author commands/$GROUP/ per the plan." \
    --label "kind:command"
done

# 5. Body from a generator — pipe gh issue view body into a new issue
gh issue view 1 \
  --repo borahanmirzaii/gh-mastery \
  --json body --jq .body \
  | gh issue create \
      --repo borahanmirzaii/gh-mastery-sandbox \
      --title "zz-issue-create-example: cloned body" \
      --body-file -
```

## Gotchas

- **Issue body links must be absolute URLs.** A relative path like `[spec](./docs/spec.md)` resolves against `https://github.com/OWNER/REPO/issues/N` — not the repo root — and 404s. Always write the full URL: `https://github.com/OWNER/REPO/blob/BRANCH/path/to/file.md`.

- **`--project` requires the `project` scope.** The flag will silently succeed at creating the issue but skip the project assignment if the token lacks the scope. Fix with `gh auth refresh -s project`. This is separate from the standard issue read/write scope.

- **`--milestone` takes the name, not the number.** `--milestone "M1 — Core"` works; `--milestone 1` does not.

- **`--body-file -` consumes stdin.** You cannot combine `--body-file -` with an interactive TTY prompt — `gh` reads stdin fully and then exits. This is by design: the flag is for scripted, non-interactive use.

- **`gh issue new` is an alias** for `gh issue create` — both do exactly the same thing.

- **`--template` by name, not filename.** Pass the display name from the template chooser, not the `.github/ISSUE_TEMPLATE/bug_report.md` path.

- **`@copilot` assignee is not supported on GitHub Enterprise Server** — the `--assignee @copilot` special value only works on github.com.

## Concepts

- None specific to this subcommand. For Projects v2 (the `--project` flag), see [`../../../concepts/projects-v2-data-model.md`](../../../concepts/projects-v2-data-model.md) once authored.

## Sources

- Manual: https://cli.github.com/manual/gh_issue_create
- Local: `gh issue create --help` (gh 2.92.0)
