---
name: mm-review
description: Slash shortcut for adversarial multimodel review of the previous agent's work.
---

Use the adversarial-multimodel-review skill to review the previous agent's work. Use the full available context. Do not optimize for token savings, speed, or a short answer. Verify from source, not from the previous agent's summary. Run gemini-critic, gpt-critic, and opus-critic in parallel if available. Synthesize disagreements and decide whether this is safe to commit, safe to deploy after runtime checks, needs fixes first, blocked, or has insufficient evidence.
