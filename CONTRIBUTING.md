# Contributing

Thank you for your interest in contributing to Gladia Skills! This guide covers how to add new skills or improve existing ones.

## Quick Start

1. Fork and clone the repository
2. Create a new branch: `git checkout -b feat/my-new-skill`
3. Add or modify skills under `plugins/gladia/skills/`
4. Submit a pull request

## Skill Structure

Each skill lives in its own directory:

```
plugins/gladia/skills/my-skill/
├── SKILL.md           # Required — main skill file
└── references/        # Optional — detailed supporting docs
    └── *.md
```

## Writing a SKILL.md

### Frontmatter (required)

```yaml
---
name: my-skill-name
description: Clear description of what this skill does AND when agents should use it. Include keywords that match how users naturally phrase their questions.
license: MIT
---
```

**Rules:**

- `name` must match the directory name (kebab-case, 1-64 chars)
- `description` should be 1-1024 characters
- Include both capability ("Transcribe pre-recorded audio") and trigger conditions ("Use when the user wants to process audio files")

### Body Content

- Keep under ~300 lines / ~800 words; move excess to `references/`
- Reference files over 100 lines should include a `## Contents` table of contents at the top
- Include code examples in both JavaScript/TypeScript and Python
- Use tables for decision guidance
- Add a "When to Use" section with trigger conditions and "When NOT to use" with alternatives
- Add a "Common Mistakes" section with 3-5 key pitfalls
- Add a "Further Reading" section with links to docs.gladia.io
- Move detailed reference material to `references/` directory

### Content Principles

1. **Be trigger-oriented** — write descriptions that match how users ask questions
2. **Be practical** — include runnable code examples, not just concepts
3. **Be concise** — agents load the full SKILL.md into context; every token counts
4. **Be accurate** — keep code examples in sync with the current SDK versions
5. **Link to references** — use `./references/topic.md` for deep dives
6. **Don't duplicate** — SDK setup lives in `sdk-integration`; use a one-liner cross-reference instead of repeating it

## Skill Quality Guidelines

For deeper guidance on writing effective skills, consult:

- [Anthropic's official skill authoring best practices](https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills/best-practices) — core principles, progressive disclosure, workflows, and evaluation
- The `CLAUDE.md` file in this repo — Gladia-specific conventions, description format, and CI-managed files

Key quality targets:

- **Descriptions**: state WHAT the skill does (capability) + WHEN to use it (trigger conditions). Do NOT summarize the skill's workflow in the description.
- **SDK-first**: every code-generating skill must use a short SDK-first cross-reference: `> **SDK-first**: always use the official SDK — see [sdk-integration](../sdk-integration/SKILL.md) for policy, setup, and fallback criteria.`
- **Cross-references**: every use-case skill must reference `sdk-integration` and `troubleshooting`
- **Token efficiency**: keep SKILL.md lean; agents load the full file into context. Move large code blocks and detailed configs to `references/` files.
- **Naming**: prefer gerund (verb-ing) form for new skill names (e.g., `transcribing-audio`)

## What NOT to Edit

- `plugins/gladia/skills/documentation-auto/SKILL.md` — managed by CI automation
- `plugins/gladia/skills/sdk-integration/references/sdk-versions.md` — managed by CI automation
- `.claude-plugin/marketplace.json` — only modify if adding a new plugin (unlikely)

## Testing Your Changes

Ask an AI agent questions that should trigger your skill and verify:

- The skill activates on relevant queries
- Code examples are correct and runnable
- Information is accurate per current API behavior

## Updating the Skills Table

When adding a new skill, update the table in `plugins/gladia/README.md`:

```markdown
| `my-skill` | Brief description of the skill |
```

## Pull Request Guidelines

- One skill per PR (unless skills are tightly related)
- Include a brief description of what the skill covers
- If updating code examples, verify they work with the current SDK version
- Reference relevant docs.gladia.io pages if applicable

## Questions?

- [Gladia Documentation](https://docs.gladia.io)
- [Support Center](https://support.gladia.io)
