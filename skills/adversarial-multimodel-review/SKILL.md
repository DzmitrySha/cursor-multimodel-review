---
name: adversarial-multimodel-review
description: Runs a deep, full-context adversarial multi-model review of previous agent work. Always optimizes for correctness over speed, cost, and brevity. Use when the user wants independent verification before commit, merge, or deployment, when checking for bugs, regressions, or scope creep, or when getting Gemini, GPT, or Opus critic opinions.
license: MIT
---

# Adversarial Multimodel Review

Use this skill after an AI agent has planned or implemented work and the user wants an independent readiness check before commit, merge, or deployment.

The goal is not to generate more opinions. The goal is to reconstruct the real task, inspect the actual code and evidence, find disagreements, and decide what must happen next.

This skill is intentionally expensive by default. Independent critics matter most for high-stakes work, so this workflow always optimizes for correctness over speed, cost, and brevity.

## Trigger Phrases

Apply this skill when the user asks things like:

- "Review the previous agent's work."
- "Can we commit this?"
- "Can we deploy this?"
- "Run a multimodel review."
- "Check for bugs, regressions, or scope creep."
- "Get Gemini/GPT/Opus to review this independently."

## Default Review Behavior

This skill always runs a deep, full-context adversarial review. There is no light mode. Always:

- Use the full available context budget. Do not optimize for token savings, speed, or brevity.
- Use the maximum available reasoning effort, thinking depth, output budget, and context window that the current Cursor model/runtime makes available.
- If Max Mode is enabled, assume the review should use the model's maximum supported context window. If you cannot verify Max Mode, model effort, or context limits from the environment, state that as a confidence limitation.
- Inspect the complete relevant diff, not only the implementation summary.
- Read complete relevant files instead of small snippets, especially prompts, rules, configs, schemas, migrations, and shared interfaces.
- Search for all readers, writers, and callers when a shared file, public API, data format, tool, prompt, or config changes.
- Review test output, runtime logs, deployment state, and known stale evidence with timestamps.
- Reconstruct the full task context: original request, follow-up corrections, plan, implementation summary, conversation history, and project rules.
- Do not stop at the first issue. Build the most complete picture possible before giving a verdict.
- Do not broaden into unrelated code or expose secrets. Deep review means complete relevant evidence, not random repository traversal.
- If evidence is too large or inaccessible, say exactly what could not be reviewed and how that limits confidence.
- In short: spend the review budget. The user opted into this workflow because missing a bug costs more than extra tokens.

## Workflow

1. Reconstruct the assignment:
   - User request and later corrections.
   - Agent plan and implementation summary.
   - Current diff, staged changes, relevant files, tests, logs, docs, rules, and runtime state.
   - Any explicit caveats such as stale logs or runtime not restarted.

2. Build an evidence packet before launching critics. Include:
   - Original request and later corrections.
   - Acceptance criteria or the expected behavior.
   - Implementation summary from the previous agent.
   - Current `git status`, diff summary, and relevant diff excerpts.
   - Key files and why they matter.
   - Test commands and exact results, including skipped or failed checks.
   - Runtime/log evidence with timestamps.
   - Known caveats such as stale logs, runtime not restarted, missing credentials, or unavailable commands.
   - Target decision: commit, merge, deploy, or continue implementation.

3. Launch critic subagents in parallel when available:
   - `gemini-critic` for broad-context alternative reasoning.
   - `gpt-critic` for rigorous implementation and regression review.
   - `opus-critic` for deep architectural and intent review.
   - `inherit-critic` when exact model IDs are unavailable or the user has selected a specific parent model.

4. Give each critic the same evidence packet:
   - Original task and constraints.
   - What the previous agent claims changed.
   - Files, diffs, logs, tests, and docs to inspect.
   - Known limitations, for example "runtime has not been restarted" or "logs may be stale."
   - A request to verify from source, not from summaries.
   - A requirement to return `INSUFFICIENT EVIDENCE` instead of guessing when task, diff, or test evidence is missing.

5. Synthesize the reviews:
   - Treat every critic finding as untrusted until checked against code or logs.
   - Merge duplicate findings.
   - Highlight disagreements and decide which side has better evidence.
   - Separate blockers from nice-to-have improvements.

6. Return a final readiness decision:
   - `BLOCK`: serious correctness, data, security, or deploy risk.
   - `FIX FIRST`: likely safe after targeted fixes.
   - `SAFE TO COMMIT`: code is ready to commit, with any minor caveats.
   - `SAFE TO DEPLOY AFTER RUNTIME CHECK`: code can be committed, but deployment needs restart, smoke test, or live verification.
   - `INSUFFICIENT EVIDENCE`: the reviewer lacked enough task, diff, test, or runtime evidence to make a reliable call.

## Critic Checklist

Each critic must answer:

- Did the implementation satisfy the user's actual request, including follow-up comments?
- Does the plan still make sense after reading the final code?
- Did the implementation preserve behavior outside the intended scope?
- Are there new bugs, regressions, race conditions, data-loss risks, security issues, or broken invariants?
- Are tests meaningful, passing, and aimed at the risky paths?
- Are docs, prompts, config, migrations, and shared data consumers still consistent?
- Did the previous agent ignore runtime evidence, stale logs, permissions, deploy state, or environment constraints?
- Did the implementation add unnecessary abstraction, scope creep, compatibility shims, or unrelated churn?
- What must be fixed before commit, merge, or deploy?

## Output Format

Use this structure:

```markdown
## Verdict
`SAFE TO COMMIT` | `SAFE TO DEPLOY AFTER RUNTIME CHECK` | `FIX FIRST` | `BLOCK` | `INSUFFICIENT EVIDENCE`

A short verdict summary explaining the decision. The detail belongs in Findings below; this is only the headline. Brevity here does not imply a shallow review.

## Findings
### Finding 1: short title
- Severity: `blocker` | `high` | `medium` | `low`
- Status: `verified` | `plausible` | `needs-evidence`
- Location: file, symbol, command, or runtime surface
- Evidence: what was observed
- Failure mode: how this could break
- Required fix: what must change
- Verification: how to prove the fix

If there are no findings, write: "No confirmed findings." Then list residual verification gaps.

## Model Disagreements
- What the critics disagreed about and which interpretation is best supported.

## Checks Performed
- Code/diff inspected.
- Tests or commands considered.
- Logs/runtime evidence considered.

## Follow-Up Prompt
Copy-paste prompt for the implementation agent to fix or verify the accepted findings.
```

## Follow-Up Prompt Template

Use this when handing review output back to the implementation agent:

```text
Here is an independent adversarial review of your previous work. Do not assume it is correct. Re-open the code, diff, tests, logs, and user requirements. Verify each finding against the actual project state. Fix only the findings you can confirm. If a finding is wrong, explain why with evidence. If there are multiple reasonable fixes, ask before changing behavior.
```

## Rules of Engagement

- Prefer evidence over confidence.
- `SAFE TO COMMIT` requires at minimum the task, relevant diff, and either relevant test results or a clear reason tests are unnecessary.
- Use `INSUFFICIENT EVIDENCE` when the task, diff, or verification evidence is missing.
- Do not punish the previous agent for harmless style differences.
- Do not expand scope unless the current change created a real risk.
- Do not approve deployment based only on static code review when runtime restart, migrations, credentials, or live checks are required.
- If critics disagree, investigate the disputed point directly before making the final call.
