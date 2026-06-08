# Gladia Skills — Maintainer Guide

Internal documentation for adding, editing, and maintaining skills in this repository.

## Repository Structure

```
├── .claude-plugin/marketplace.json    # Claude Code marketplace catalog
├── plugins/gladia/
│   ├── .claude-plugin/plugin.json     # Plugin manifest
│   ├── README.md                      # Plugin overview
│   └── skills/
│       ├── <skill-name>/
│       │   ├── SKILL.md               # Main skill file (required)
│       │   └── references/            # Optional detailed docs
│       │       └── *.md
│       ├── gladia-sdk-integration/
│       │   ├── SKILL.md               # SDK setup, decision guide
│       │   └── references/
│       │       ├── sdk-versions.md    # CI-managed (do not edit)
│       │       ├── javascript.md
│       │       └── python.md
│       └── gladia-documentation-auto/        # CI-managed (do not edit)
│           └── SKILL.md
├── .github/workflows/
│   ├── sync-documentation.yml             # Auto-sync documentation skill from docs.gladia.io
│   └── sync-sdk-versions.yml         # Auto-sync SDK versions from npm/PyPI
├── CLAUDE.md                          # This file
├── CONTRIBUTING.md                    # Public contribution guide
├── README.md                          # User-facing installation docs
└── LICENSE                            # MIT
```

## Adding a New Skill

1. Create a new directory under `plugins/gladia/skills/` with a kebab-case name
2. Add a `SKILL.md` file with the required frontmatter
3. Optionally add a `references/` directory for detailed content
4. Update `plugins/gladia/README.md` skill table

### SKILL.md Requirements

Every `SKILL.md` must have YAML frontmatter:

```yaml
---
name: my-skill-name # Must match directory name, kebab-case
description: ... # 1-1024 chars. Describes WHAT it does AND WHEN to use it.
license: MIT # Optional
---
```

### Naming Conventions

Use **kebab-case** for all skill directory names and use the `gladia-` prefix for all Gladia skills (for example, `gladia-transcribing-audio`, `gladia-integrating-sdk`). Prefer **gerund (verb-ing) form** for new skill names after the prefix.

### Writing Good Descriptions

The `description` field is critical — it's how agents decide which skill to activate. Write it as a trigger phrase that includes both WHAT the skill does (capability) and WHEN to use it (triggers). Every description for a skill that involves JS/TS or Python code must include a short **SDK-first directive**.

**Important:** describe the skill's _capability_ and trigger conditions, but do NOT summarize the skill's _workflow or process_ in the description. If the description summarizes the workflow, agents may follow the description as a shortcut instead of reading the full SKILL.md body.

**Good**: "Real-time speech-to-text streaming with Gladia via WebSocket. Use when the user needs live transcription, builds a voice agent, meeting recorder, or call center integration. Always prefer the official SDK; fall back to raw WebSocket/REST only when SDK cannot satisfy the requirement."

**Bad**: "Live transcription skill" (too vague, won't reliably trigger, no SDK-first directive)

**Bad**: "Real-time streaming skill that opens a WebSocket, streams audio chunks, polls for partials, and closes the session" (summarizes workflow — agents will skip the full skill body)

### Content Guidelines

- Keep `SKILL.md` under ~300 lines / ~800 words; move excess to `references/`
- Reference files over 100 lines should include a `## Contents` table of contents at the top
- Include code examples in both JavaScript and Python using the SDK
- Use decision tables to help agents choose the right approach
- Add a "When to Use" section with trigger conditions and "When NOT to use" with alternatives
- Add a "Further Reading" section linking to docs.gladia.io pages
- **Do not duplicate SDK setup/init content** — reference the `gladia-sdk-integration` skill instead
- Code examples should assume the `GladiaClient` is already initialized; point to `gladia-sdk-integration` for setup
- **Do not duplicate** the SDK-first policy blockquote in every skill — use a short one-liner referencing `gladia-sdk-integration`

### Reference Files

Link to reference files explicitly in SKILL.md:

```markdown
## References

Consult these resources as needed:

- ./references/topic-a.md -- Description of what's in this file
- ./references/topic-b.md -- Description of what's in this file
```

### Cross-Referencing Skills

Skills should cross-reference related skills so the agent discovers them at load time. Use relative paths in the References section:

```markdown
- ../gladia-sdk-integration/SKILL.md -- SDK setup, client initialization, error handling
- ../gladia-sdk-integration/references/sdk-versions.md -- Current SDK versions (auto-synced by CI)
- ../gladia-troubleshooting/SKILL.md -- Common errors and verification checklist
```

Every skill that generates code should reference `gladia-sdk-integration` for setup and `gladia-troubleshooting` for diagnostics. The `gladia-sdk-integration` skill should reference use-case skills back.

## Modifying Existing Skills

- Edit the SKILL.md or reference files directly
- Test by asking an AI agent questions that should trigger the skill
- Verify the skill doesn't overlap too much with other skills' descriptions

## CI-Managed Files

### gladia-documentation-auto Skill

**Do not edit** `plugins/gladia/skills/gladia-documentation-auto/SKILL.md` manually. It is managed by the `sync-documentation.yml` GitHub Actions workflow that:

1. Runs daily at 06:00 UTC
2. Fetches the digest from `docs.gladia.io/.well-known/agent-skills/index.json`
3. If changed, downloads the new skill and opens a PR

To force a sync: run the workflow manually from GitHub Actions.

### SDK Versions

**Do not edit** `plugins/gladia/skills/gladia-sdk-integration/references/sdk-versions.md` manually. It is managed by the `sync-sdk-versions.yml` GitHub Actions workflow that:

1. Runs daily at 06:30 UTC (after documentation sync)
2. Checks npm (`@gladiaio/sdk`) and PyPI (`gladiaio-sdk`) for the latest versions
3. If changed, updates `sdk-versions.md` and the JS/Python reference files, then opens a PR

The workflow also updates the `Version` field in `references/javascript.md` and `references/python.md`.

To force a sync: run the workflow manually from GitHub Actions.

## Local Testing

### Claude Code

```bash
claude --plugin-dir ./plugins/gladia
```

### Cursor

Symlink or add this repo as a Remote Rule in settings.

### Validation

If you have the skills CLI:

```bash
npx skills-ref validate ./plugins/gladia/skills/my-skill
```

## Conventions

- Directory names: `kebab-case`
- File names: `SKILL.md` (uppercase), `*.md` for references (lowercase kebab-case)
- Frontmatter `name`: must match the directory name
- All skills in one plugin (`plugins/gladia`)
- No runtime dependencies at repo root
- Pure markdown + optional scripts
- **SDK-first**: all code examples must use the official SDK; raw REST/WebSocket only as a documented fallback, or if the user want a low level control
- **No duplication**: SDK setup, client init, and error handling live in `gladia-sdk-integration` — other skills reference it
- **No hardcoded SDK versions**: version info is CI-managed in `sdk-versions.md`
