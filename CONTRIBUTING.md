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
- Critics should always run a deep, full-context review. Never reintroduce a "light" or "quick" mode and never advise saving tokens on inspection depth.
- Findings should include evidence and impact.
- The package should prefer high-signal blockers over long speculative lists, but only after a full-context review. "High signal" must never be implemented by skimming.
- Model-specific prompts should differ in review perspective, not just branding.
- The `inherit-critic` fallback should remain available for users with plan, region, or Max Mode limitations.

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
- [ ] New prompts are evidence-focused and mechanically complete. They must not weaken the always-on full-context default.
- [ ] Known limitations are updated if behavior changed.
