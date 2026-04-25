# Cursor Multimodel Review

Deep adversarial review for Cursor. One agent implements; several read-only critic agents review the work from scratch before you commit, merge, or deploy.

This plugin is intentionally expensive by default. It tells critics to use the full available context, read complete relevant files, inspect the full diff, check tests/logs/docs/rules/runtime state, and avoid saving tokens when review quality is at stake.

## Install: Paste This Into Cursor

Open a Cursor chat in any project and paste:

```text
Install this Cursor plugin locally:

https://github.com/joi-lab/cursor-multimodel-review

Clone it into ~/.cursor/plugins/local/adversarial-multimodel-review.
If that folder already exists, update it with git pull.
Do not modify my current project files.
After installing, tell me to run "Developer: Reload Window" in Cursor.
```

Then run **Developer: Reload Window** in Cursor.

Cursor's docs say local plugins are loaded from `~/.cursor/plugins/local` and require either **Developer: Reload Window** or a Cursor restart.

After reload, run:

```text
/adversarial-multimodel-review

Review the previous agent's work. Use the full available context. Do not optimize for token savings, speed, or a short answer. Verify from source, not from the previous agent's summary.
```

If the slash command is not visible after reload, ask Cursor:

```text
Use the adversarial-multimodel-review skill to review the previous agent's work.
```

## Manual Install

If you prefer terminal commands:

```bash
mkdir -p ~/.cursor/plugins/local
git clone https://github.com/joi-lab/cursor-multimodel-review \
  ~/.cursor/plugins/local/adversarial-multimodel-review
```

Update later with:

```bash
cd ~/.cursor/plugins/local/adversarial-multimodel-review
git pull
```

Then run **Developer: Reload Window**.

## What It Adds

- Skill: `adversarial-multimodel-review`
- Read-only critics:
  - `gemini-critic` requests `gemini-3-1-pro`
  - `gpt-critic` requests `gpt-5-5`
  - `opus-critic` requests `claude-opus-4-7`
  - `inherit-critic` inherits the parent model

## Review Verdicts

The skill asks the parent agent to return one of these verdicts:

- `BLOCK`: serious correctness, data, security, or deploy risk.
- `FIX FIRST`: likely safe after targeted fixes.
- `SAFE TO COMMIT`: code is ready to commit, with minor caveats if any.
- `SAFE TO DEPLOY AFTER RUNTIME CHECK`: code can be committed, but deployment still needs restart, smoke test, migration check, or live verification.
- `INSUFFICIENT EVIDENCE`: the reviewer did not get enough task, diff, test, or runtime evidence to make a reliable call.

## Models And Max Mode

The plugin requests these models:

```yaml
model: gemini-3-1-pro
model: gpt-5-5
model: claude-opus-4-7
```

Cursor may fall back if a model is unavailable on your plan, region, team policy, or Max Mode settings.

Cursor's documented subagent frontmatter does **not** include separate `reasoning_effort`, `max_tokens`, or `context_length` fields. This plugin asks critics to use the maximum available effort/context in their prompts. For the biggest context window, turn on **Max Mode** in Cursor before running the review.

Use `inherit-critic` when you want the critic to inherit the exact parent model and Max Mode state.

## Known Limitations

- This uses more tokens and takes longer than normal review.
- Subagents start with clean context. Give them the task, diff, files, logs, and constraints.
- Exact model IDs can change or be unavailable. Keep `inherit-critic` as the stable fallback.
- Static review cannot prove deployment safety. If runtime was not restarted or logs are stale, require a runtime check.

## Marketplace

Yes, this can be submitted to the Cursor Marketplace.

Cursor says marketplace plugins are reviewed manually and must be open source. Submit here:

https://cursor.com/marketplace/publish

This repository is already public, so the next step is a manual submission.
