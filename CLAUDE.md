# Polysyllogism -- Development Conventions

Project conventions for contributors and Claude Code sessions working on this plugin.

## Plugin Structure

This is a Claude Code plugin. The directory layout follows the Claude Code plugin specification:

```
polysyllogism/
  .claude-plugin/
    plugin.json              # Required. Plugin manifest with name, description, author.
  commands/
    chain-audit.md           # Slash commands. One file per command. Frontmatter defines description and argument-hint.
  agents/
    chain-analyzer.md        # Agent definitions. One file per agent. Frontmatter defines name, description, tools, model, color.
  skills/
    polysyllogism/
      SKILL.md               # Skill entry point. Frontmatter defines name and description (trigger contract).
      references/
        anti-patterns.md     # 14 named failure modes -- the diagnostic catalog.
        pipeline.md          # Phase-by-phase execution protocol.
        templates.md         # Reusable schemas, manifests, prompt injections for both domains.
        lexicon.md           # Precision terminology. Aristotelian vocabulary used across all files.
```

### Conventions

- Commands go in `commands/` as `.md` files. The filename (minus extension) becomes the slash command name.
- Agents go in `agents/` as `.md` files. They are spawned via the Task tool, not invoked directly by users.
- Skills go in `skills/<skill-name>/SKILL.md`. References live in `skills/<skill-name>/references/`.
- The plugin manifest at `.claude-plugin/plugin.json` must always exist and must contain `name`, `description`, and `author`.

## Testing Changes

Load the plugin locally with the `--plugin-dir` flag:

```
claude --plugin-dir /path/to/polysyllogism
```

To verify changes:

1. Start a fresh Claude Code session with the plugin loaded.
2. Test the `/chain-audit` command with a known agentic codebase or plugin directory.
3. Test skill activation by using trigger phrases ("chain analysis", "agent audit", etc.) in natural language.
4. Confirm the chain-analyzer agent spawns correctly and returns a topology map.
5. Confirm references load on demand -- check that SKILL.md directives are followed and not all references are loaded at once.

There is no automated test suite. Validation is manual: load the plugin, run it against a target, and verify the output.

## Anti-Pattern Naming Conventions

All 14 anti-patterns use Aristotelian vocabulary. This is deliberate. The terms are precise, each mapping to a specific logical failure mode with a defined diagnostic and fix.

When adding or modifying anti-patterns:

- Use terms from the formal logic and rhetorical traditions (syllogism, enthymeme, elenchus, apodeictic, etc.).
- Each anti-pattern must have: a name, a structural description of the failure, a diagnostic (how to detect it), and a fix (how to remediate it).
- The name should describe the logical failure, not the symptom. "Context Starvation" names the structural cause; "agent crashes" names the symptom.
- Patterns 1-8 are framework-agnostic (classic chain failures). Patterns 9-14 are Claude Code / LLM-native. Maintain this division.
- Consult `references/lexicon.md` before introducing new terminology to avoid conflicts or redundancies.

## Reference File Organization

References live in `skills/polysyllogism/references/` and are loaded on demand during execution phases:

| File | Loaded During | Contains |
|------|---------------|----------|
| `pipeline.md` | SCAN, ANALYZE, DELIBERATE, full walkthroughs | Step-by-step execution protocol for all six phases |
| `anti-patterns.md` | ANALYZE, DELIBERATE | The 14 failure modes with diagnostics and fixes |
| `templates.md` | OPTIMIZE / APPLY | Reusable code schemas, capability manifests, prompt injections, plugin templates |
| `lexicon.md` | On demand (terminology questions) | Precision vocabulary definitions |

Rules for reference files:

- Each reference file must have a clear, distinct scope. Do not duplicate content across references.
- SKILL.md must contain explicit loading directives telling the model which reference to load for which phase. This prevents Reference Overflow (anti-pattern 12).
- Keep individual reference files focused. If a file grows beyond its scope, split it along natural joints.
- Templates in `templates.md` cover both code pipeline and Claude Code plugin domains. Maintain this dual coverage.

## Lexicon Stability

`references/lexicon.md` defines the precision vocabulary used across all other files in the plugin. Terms defined there appear in SKILL.md, anti-patterns.md, pipeline.md, templates.md, the agent prompt, and the command prompt.

Do not modify lexicon.md without careful consideration. Renaming or redefining a term can silently break references throughout the plugin. If a term must change:

1. Search all files for current usage of the term.
2. Update every reference to the term across all files atomically.
3. Verify that anti-pattern descriptions, diagnostics, and fixes still read correctly with the new term.
4. Confirm that SKILL.md trigger phrases and key terms sections remain consistent.

When adding new terms, ensure they do not overlap with existing definitions. Each term should name exactly one concept.
