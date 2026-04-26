---
name: adversarial-multimodel-review
description: Adversarial multi-model review with modes — standard (default, bounded evidence), light (fast, single inherit critic), or deep (full context). Use before commit, merge, or deployment when the user wants independent verification, bug checks, or Gemini/GPT/Opus critic opinions.
license: MIT
---

# Adversarial Multimodel Review

Use this skill after an AI agent has planned or implemented work and the user wants an independent readiness check before commit, merge, or deployment.

The goal is not to generate more opinions. The goal is to reconstruct the real task, inspect the actual code and evidence, find disagreements, and decide what must happen next.

**Precedence:** If anything below conflicts with older community phrasing about "always full context" or "no light mode", this document wins for the **selected review mode**.

## Review modes

Choose **exactly one** mode per invocation.

### How to select the mode

1. If the user explicitly asks for **light** / **лёгкий** / **быстрый** / `light review` / `quick review` → **light**.
2. If they ask for **deep** / **глубокий** / **полный контекст** / `deep review` / `max review` / `full context` → **deep**.
3. If they ask for **standard** / **стандартный** / `standard review` → **standard**.
4. Otherwise → **standard** (default).

State the chosen mode once in the user-visible summary (for example: "Using **standard** mode.").

### standard (default)

- **Goal:** High-signal adversarial review with controlled cost.
- **Critics:** Launch `gemini-critic`, `gpt-critic`, and `opus-critic` in parallel when available. Use `inherit-critic` instead of unavailable named critics (same evidence packet for all).
- **Evidence packet (hard limits):**
  - Start from the **current diff** and **only files touched** by that diff.
  - Add **at most one hop** of callers or readers for symbols changed in public APIs, configs, schemas, shared types, or persistence boundaries (stop at the first convincing layer; no repo-wide search).
  - **Full-file reads** in the packet: at most **5** files, prioritized by risk (auth, data, concurrency, migrations, prompts, security-sensitive config). Everything else: **excerpts with path and line range** (cap **~120 lines** per excerpt unless the user chose deep).
  - Conversation / task recap for the packet: **≤ ~800 tokens** unless deep; include user goal, constraints, and implementation summary—omit unrelated chat.
- **Quality bar:** Same verdict taxonomy and skepticism toward summaries. Use `INSUFFICIENT EVIDENCE` when the packet cannot support a reliable claim. Do not broaden into unrelated code or expose secrets.

### light

- **Goal:** Fast gate; cheapest useful check before deeper work.
- **Critics:** Run **only** `inherit-critic` unless the user names one other critic.
- **Evidence packet:** The diff (or patch), optional **≤ 2** supporting excerpts with line ranges, one test or lint command plus **≤ 120 lines** of output, and a short bullet list of **assumptions**. No multi-hop tracing unless the user expands scope.
- **Limitation:** Say explicitly that cross-module and architectural risks may be under-covered compared to standard or deep.

### deep

- **Goal:** Maximum assurance for high-stakes changes (releases, security, large refactors).
- **Critics:** Launch `gemini-critic`, `gpt-critic`, and `opus-critic` in parallel when available; use `inherit-critic` as fallback when model IDs are unavailable.
- **Evidence packet:** Spend the review budget. Use the full available context budget, maximum reasoning effort, output budget, and context window the Cursor runtime allows. If Max Mode is enabled, assume the review should use the model's maximum supported context window; if you cannot verify Max Mode, state that as a confidence limitation.
- Inspect the **complete relevant diff**, not only the implementation summary. Read **complete relevant files** where justified (prompts, rules, configs, schemas, migrations, shared interfaces). Search for readers, writers, and callers when shared surfaces change. Include test output, runtime logs, deployment state, and stale-evidence caveats with timestamps. Reconstruct full task context including follow-ups and project rules. Do not stop at the first issue. If evidence is too large or inaccessible, say exactly what could not be reviewed.

## Trigger phrases

Apply this skill when the user asks things like:

- "Review the previous agent's work." (optionally with light / standard / deep.)
- "Can we commit this?" / "Can we deploy this?"
- "Run a multimodel review." / "multimodel review"
- "Check for bugs, regressions, or scope creep."
- "Get Gemini/GPT/Opus to review this independently."
- Russian: «проверь работу агента», «мультимодельное ревью», «лёгкий/стандартный/глубокий обзор».

## Workflow

1. **Select the review mode** using the rules above. Announce it briefly.

2. Reconstruct the assignment (scope to the mode):
   - User request and later corrections.
   - Agent plan and implementation summary.
   - Current diff, staged changes, relevant files, tests, logs, docs, rules, and runtime state — **only as permitted by the mode's evidence limits**.
   - Any explicit caveats (stale logs, runtime not restarted).

3. **Build an evidence packet** before launching critics. Always include where applicable:
   - Original request and later corrections.
   - Acceptance criteria or expected behavior.
   - Implementation summary from the previous agent.
   - Current `git status`, diff summary, and diff content (full for light/standard if small; for deep follow deep rules).
   - Key files and why they matter (respect standard/light caps).
   - Test commands and results (respect line caps in light).
   - Runtime/log evidence with timestamps when provided.
   - Known caveats and target decision (commit, merge, deploy, continue).

4. **Launch critic subagents** (see mode):
   - **light:** only `inherit-critic`.
   - **standard** or **deep:** `gemini-critic`, `gpt-critic`, `opus-critic` in parallel when available; substitute `inherit-critic` for any unavailable named critic.

5. **First line of every critic task** (required):  
   `Review mode: light` | `Review mode: standard` | `Review mode: deep`  
   Then paste the same evidence packet for each critic in that invocation.

6. Give each critic:
   - Original task and constraints.
   - What the previous agent claims changed.
   - Files, diffs, logs, tests, and docs **included in the packet** (do not ask critics to pull paths not supplied in standard/light).
   - Known limitations.
   - A request to verify from source, not from summaries.
   - A requirement to return `INSUFFICIENT EVIDENCE` instead of guessing when task, diff, or test evidence is missing.

7. Synthesize the reviews:
   - Treat every critic finding as untrusted until checked against code or logs.
   - Merge duplicate findings.
   - Highlight disagreements and decide which side has better evidence.
   - Separate blockers from nice-to-have improvements.

8. Return a final readiness decision:
   - `BLOCK`: serious correctness, data, security, or deploy risk.
   - `FIX FIRST`: likely safe after targeted fixes.
   - `SAFE TO COMMIT`: code is ready to commit, with any minor caveats.
   - `SAFE TO DEPLOY AFTER RUNTIME CHECK`: code can be committed, but deployment needs restart, smoke test, or live verification.
   - `INSUFFICIENT EVIDENCE`: the reviewer lacked enough task, diff, test, or runtime evidence to make a reliable call.

## Critic checklist

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

## Output format

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

## Model disagreements
- What the critics disagreed about and which interpretation is best supported.

## Checks performed
- Code/diff inspected.
- Tests or commands considered.
- Logs/runtime evidence considered.

## Follow-up prompt
Copy-paste prompt for the implementation agent to fix or verify the accepted findings.
```

## Follow-up prompt template

Use this when handing review output back to the implementation agent:

```text
Here is an independent adversarial review of your previous work. Do not assume it is correct. Re-open the code, diff, tests, logs, and user requirements. Verify each finding against the actual project state. Fix only the findings you can confirm. If a finding is wrong, explain why with evidence. If there are multiple reasonable fixes, ask before changing behavior.
```

## Rules of engagement

- Prefer evidence over confidence.
- `SAFE TO COMMIT` requires at minimum the task, relevant diff, and either relevant test results or a clear reason tests are unnecessary.
- Use `INSUFFICIENT EVIDENCE` when the task, diff, or verification evidence is missing.
- Do not punish the previous agent for harmless style differences.
- Do not expand scope unless the current change created a real risk.
- Do not approve deployment based only on static code review when runtime restart, migrations, credentials, or live checks are required.
- If critics disagree, investigate the disputed point directly before making the final call.
