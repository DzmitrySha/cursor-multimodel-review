# Contributing

Thanks for improving Cursor Adversarial Multimodel Review.

## Good Contributions

- Better critic prompts with fewer false positives.
- Updated model IDs when Cursor changes model availability.
- Clearer installation instructions.
- Examples from real review workflows.
- Additional critic agents with distinct review value.

## Review Principles

Keep the package focused:

- Critics should be read-only by default.
- The skill defines three modes — **standard** (default, bounded evidence), **light** (single `inherit-critic`, minimal packet), and **deep** (full context). Do not add unofficial modes or bypass mode selection in docs.
- In **standard** and **light**, critics must stay inside the evidence packet the parent sends; depth comes from evidence quality, not repo-wide guessing.
- In **deep**, critics use full-context rules; do not water down deep-mode instructions.
- Findings should include evidence and impact.
- Prefer high-signal blockers over long speculative lists; avoid vague quality advice in any mode.
- Model-specific prompts should differ in review perspective, not just branding.
- The `inherit-critic` agent remains required for **light** mode and as a fallback when named models are unavailable.

## Testing Changes Locally

Symlink the repository into Cursor's local plugin directory:

```bash
mkdir -p ~/.cursor/plugins/local
ln -s /path/to/cursor-adversarial-review ~/.cursor/plugins/local/adversarial-multimodel-review
```

Restart Cursor or run `Developer: Reload Window`, then verify that the skill and agents appear in Cursor.

If symlink loading is unreliable in your Cursor build, test with a copied directory instead:

```bash
rm -rf ~/.cursor/plugins/local/adversarial-multimodel-review
cp -R /path/to/cursor-adversarial-review ~/.cursor/plugins/local/adversarial-multimodel-review
```

## Pull Request Checklist

- [ ] The plugin manifest remains valid JSON.
- [ ] Skill `name` matches its folder name.
- [ ] Agent names are lowercase and hyphenated.
- [ ] README examples (including any agent frontmatter snippets) match the actual agent files.
- [ ] New prompts are evidence-focused and mechanically complete. They must respect **Review mode** behavior (light/standard/deep) and not contradict the skill.
- [ ] Known limitations are updated if behavior changed.
