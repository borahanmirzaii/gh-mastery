# `gh run watch`

> **One-liner:** Stream live step-by-step progress of a workflow run until it completes.

## When you reach for it

Right after pushing a branch or triggering a dispatch — you want real-time CI feedback in the terminal rather than refreshing a browser tab. The critical variant is `--exit-status`: it makes the command itself fail when the run fails, so you can gate (= block, then abort on failure) subsequent script steps on CI green.

## Key flags

- `--exit-status` — exit non-zero if the run fails. **This is the single most important flag.** Without it, `watch` always exits 0 (even on a failed run), so downstream `&&` chains see a false green.
- `--compact` — show only failed/relevant steps instead of every step. Useful for long matrix runs where most steps pass.
- `-i, --interval <sec>` — polling interval in seconds (default 3). Lower it to 1 for faster feedback; higher it to 10 to reduce API calls in CI scripts.
- `-R, --repo [HOST/]OWNER/REPO` — operate on a different repo.

## Examples

```bash
# 1. Basic watch — stream progress, exit 0 regardless of outcome
gh run watch 26679159041 --repo borahanmirzaii/gh-mastery-sandbox

# 2. CI-gating pattern — block until green, fail if red
gh run watch 26679159041 \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --exit-status && echo "Tests passed — deploying" && ./deploy.sh

# 3. Compact mode — for large matrix workflows, show only relevant steps
gh run watch 26679159041 \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --compact

# 4. Grab the latest run ID, then watch it (common scripting pattern)
RUN_ID=$(gh run list \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --limit 1 --json databaseId --jq '.[0].databaseId')
gh run watch "$RUN_ID" \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --exit-status

# 5. Notify on completion (cross-platform)
gh run watch 26679159041 --exit-status \
  && echo "Run finished successfully" \
  || echo "Run FAILED — check logs"

# 6. Watch at a slower poll rate (less API noise in long-running CI)
gh run watch 26679159041 \
  --repo borahanmirzaii/gh-mastery-sandbox \
  --interval 10
```

## Gotchas

1. **`--exit-status` is required for CI gating.** Without it, `gh run watch` exits 0 even if the run fails. This is a subtle footgun (= a flag or default that easily causes self-inflicted mistakes) — the command looks like it worked, but the downstream `&&` still fires on a red build.

2. **Fine-grained PATs are not supported.** The `checks:read` permission cannot be granted to fine-grained personal access tokens. If `watch` fails with `403` or an auth error while other `gh` commands work, you are using a fine-grained PAT. Switch to a classic token.

3. **`watch` with no run ID is interactive.** If you omit the run ID, `gh` opens a picker — but only in a terminal (TTY). In scripts, always pass the ID explicitly.

4. **The run must already exist.** If you trigger a workflow and immediately call `gh run watch`, the run may not appear for 2–5 seconds. Add a brief `sleep 3` or poll with `gh run list` before calling `watch`.

5. **`--compact` hides passing steps, not only failed ones.** It shows "relevant" steps — currently in-progress or failed — which speeds up reading long matrix runs but may hide context you want.

## Concepts

- [GitHub Actions model](../../../concepts/actions-model.md) _(to be written)_ — how a push creates a run, how runs map to jobs and steps, and what `checks:read` governs.

## Sources

- Manual: https://cli.github.com/manual/gh_run_watch
- Local: `gh run watch --help` (gh 2.93.0)
