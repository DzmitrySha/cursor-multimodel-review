---
name: mm-review
description: Slash shortcut for adversarial multimodel review — routes light / standard / deep and runs the adversarial-multimodel-review skill.
---

Determine the review mode from the user's message (same language rules as the skill):

- **light** if they said light / лёгкий / быстрый / quick review / light review (or clearly want the cheapest pass).
- **deep** if they said deep / глубокий / полный контекст / max review / full context / deep review (or clearly want maximum assurance).
- **standard** if they said standard / стандартный / standard review (explicit default).
- Otherwise **standard** (default).

Then run the **adversarial-multimodel-review** skill for the previous agent's work using that mode only: follow its evidence limits, critic launch rules, and require the first line of each critic task to be `Review mode: <light|standard|deep>`. Verify from source within the packet; synthesize critic output and return the skill's verdict format (including model disagreements when multiple critics ran).
