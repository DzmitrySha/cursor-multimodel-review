# Cursor Adversarial Multimodel Review

A Cursor plugin that runs deep, full-context adversarial review of previous agent work before you commit, merge, or deploy AI-generated changes. It coordinates independent critic subagents that request different model families and always optimizes for correctness over speed, cost, and brevity.

The workflow is simple: after one agent implements something, ask separate critics to verify the work from scratch. They reconstruct the full task, read complete relevant files, inspect the full diff, check tests, logs, docs, rules, and runtime state, and the parent agent synthesizes the disagreements into a final readiness decision.

This is intentionally an expensive workflow. It is for high-stakes commits, merges, and deploys where missing a bug costs more than the extra tokens. There is no light mode.

## What This Includes

- A Cursor skill: `adversarial-multimodel-review`
- Four custom read-only critic subagents that always run full-context reviews:
  - `gemini-critic` requesting `gemini-3-1-pro`
  - `gpt-critic` requesting `gpt-5-5`
  - `opus-critic` requesting `claude-opus-4-7`
  - `inherit-critic` requesting `inherit` as a fallback
- A demanding review checklist for correctness, regressions, tests, scope creep, docs drift, runtime state, and deploy risk
- Copy-paste prompts for running the review loop

## Why This Works

AI agents are often confident about their own work. Independent reviewers with fresh context are better at spotting:

- Requirement mismatches
- Hidden regressions
- Scope creep
- Missing tests
- Stale logs or runtime assumptions
- Risky deploy steps
- Overconfident conclusions from the implementation agent

This is commonly called adversarial review. The point is not to create drama between models. The point is to force claims back through evidence.

## Package Layout

```text
cursor-adversarial-review/
├── .cursor-plugin/
│   └── plugin.json
├── agents/
│   ├── gemini-critic.md
│   ├── gpt-critic.md
│   ├── opus-critic.md
│   └── inherit-critic.md
├── skills/
│   └── adversarial-multimodel-review/
│       └── SKILL.md
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

## Local Installation

Clone this repository, then copy it into Cursor's local plugin directory:

```bash
mkdir -p ~/.cursor/plugins/local
cp -R /path/to/cursor-adversarial-review ~/.cursor/plugins/local/adversarial-multimodel-review
```

Restart Cursor or run `Developer: Reload Window`.

In Cursor Settings, check that the plugin's skill and agents are visible. The skill can be invoked manually with:

```text
/adversarial-multimodel-review
```

For plugin development, a symlink can be more convenient:

```bash
mkdir -p ~/.cursor/plugins/local
ln -s /path/to/cursor-adversarial-review ~/.cursor/plugins/local/adversarial-multimodel-review
```

If Cursor does not load the symlinked plugin, replace it with the copied directory.

## Quick Start

After an implementation, ask Cursor:

```text
/adversarial-multimodel-review

Review the previous agent's work. Do not optimize for token savings, speed, or a short answer. Use the full available context budget.

Build the complete relevant picture from our conversation, my follow-up comments, the agent's plan, the current diff, complete relevant files, tests, logs, rules, prompts, docs, config, and runtime state. Read whole relevant files instead of tiny snippets. Trace all related readers, writers, callers, and downstream consumers for shared data formats, public APIs, tools, prompts, or config changes.

If logs are stale, runtime was not restarted, tests were skipped, credentials are missing, or a file is too large to fully inspect, say exactly how that limits the verdict. Do not trust the previous agent's summary. Verify from source.

Run gemini-critic, gpt-critic, and opus-critic in parallel if available. Synthesize disagreements and decide whether this is safe to commit, safe to deploy after runtime checks, needs fixes first, blocked, or has insufficient evidence.

Task:
<what the user originally asked for>

Implemented change:
<short summary from the implementing agent>

Evidence pointers:
- Diff: inspect the current workspace diff, or use the pasted diff below
- Key files: <paths>
- Tests run: <commands and output>
- Logs/runtime: <timestamps and caveats, or say not checked>
- Known caveats: <stale logs, skipped checks, runtime not restarted>
```

The critics will still run a deep, full-context review even if you skip the prompt body, because the skill and agents enforce that by default.

To run the fallback critic that inherits the parent chat model:

```text
Use inherit-critic to review the previous agent's work. Use the full available context. Verify the actual code, complete relevant files, and diff, not just the implementation summary. Tell me whether this is safe to commit.
```

To run one critic directly, ask for the custom agent by name:

```text
Use the opus-critic custom agent to review the previous implementation. Do not edit files. Use the full available context. Return blockers first.
```

If your Cursor build does not expose custom agents directly, invoke `/adversarial-multimodel-review` and ask it to use only `opus-critic`.

## Recommended Workflow

1. Let an implementation agent make the change.
2. Run `/adversarial-multimodel-review`.
3. Let the parent agent synthesize the critic outputs.
4. Hand confirmed findings back to the implementation agent.
5. Ask the implementation agent to verify each finding against real code before fixing it.
6. Re-run the review if the fix touched risky behavior.

## Follow-Up Prompt For The Implementer

```text
Here is an independent adversarial review of your previous work. Do not assume it is correct. Re-open the code, diff, tests, logs, and user requirements. Verify each finding against the actual project state. Fix only the findings you can confirm. If a finding is wrong, explain why with evidence. If there are multiple reasonable fixes, ask before changing behavior.
```

## Review Verdicts

The skill asks the parent agent to return one of these verdicts:

- `BLOCK`: serious correctness, data, security, or deploy risk.
- `FIX FIRST`: likely safe after targeted fixes.
- `SAFE TO COMMIT`: code is ready to commit, with minor caveats if any.
- `SAFE TO DEPLOY AFTER RUNTIME CHECK`: code can be committed, but deployment still needs restart, smoke test, migration check, or live verification.
- `INSUFFICIENT EVIDENCE`: the reviewer lacks enough task, diff, test, or runtime evidence to make a reliable call.

## Model Configuration

The explicit critic agents request these model IDs:

```yaml
model: gemini-3-1-pro
model: gpt-5-5
model: claude-opus-4-7
```

Cursor may fall back to another model if your plan, region, Max Mode settings, or team admin policy does not allow a requested model.

Some Cursor surfaces display marketing names with dots, such as `Gemini 3.1 Pro` or `GPT-5.5`, while docs URLs and examples often use slug-style IDs. If your Cursor build rejects an explicit ID, either edit the agent frontmatter to the model ID shown in your model picker or use `inherit-critic`.

If exact model IDs are unreliable in your environment, use:

```yaml
model: inherit
```

Then select the desired model in the parent Cursor chat and invoke `inherit-critic`.

To customize a critic model, edit only the `model:` line at the top of the relevant file in `agents/`. Keep the rest of the frontmatter and prompt body in sync with the package, otherwise the deep-review default will drift. Example:

```yaml
---
name: gpt-critic
description: Independent GPT critic for deep, full-context implementation review. Always uses the full available context.
model: inherit
readonly: true
---
```

## Known Limitations

- This is intentionally an expensive workflow. Always-on full-context review plus three critics will use significantly more tokens, time, and rate-limit pressure than a single agent review. That is the point. Use it for risky commits, merges, deploys, or work where missing a bug costs more than the extra tokens.
- Subagents start with clean context. The parent agent must pass them the task, diff, key files, logs, and constraints. The critic prompts are tuned to read more from source on top of that.
- Built-in Cursor subagents may choose faster models automatically. This package uses custom subagents so the model can be requested in frontmatter.
- Exact model IDs can change or be unavailable. Keep `inherit-critic` as the stable fallback.
- A static review cannot prove deployment safety. If runtime has not been restarted or logs are stale, require a runtime check.
- Critics can be wrong. Treat their output as hypotheses and verify important findings against source.

## Worked Mini Example

```text
Task:
Add validation so archived users cannot create new projects.

Implemented change:
The previous agent added a UI check that hides the "New Project" button for archived users.

Evidence to inspect:
- Diff: current workspace diff
- Key files: src/projects/CreateProjectButton.tsx, src/api/projects.ts
- Tests run: npm test -- CreateProjectButton
- Known caveats: API tests were not run
```

Possible synthesis:

- Accept the critic finding that the API still allows archived users to create projects. This is a blocker because UI-only validation is bypassable.
- Reject the critic finding about button styling because it is unrelated and has no behavioral impact.
- Verdict: `FIX FIRST`.

## Publishing Notes

Before publishing your fork:

1. Update `.cursor-plugin/plugin.json` with your author name and repository URL.
2. Pick a license. This template uses MIT.
3. Test locally from `~/.cursor/plugins/local`.
4. Make sure this is a public Git repository.
5. Push to GitHub.
6. Submit at https://cursor.com/marketplace/publish or list it on community plugin directories.

## Credits

Inspired by adversarial review workflows where one agent implements and independent agents try to disprove the result before commit or deploy.
