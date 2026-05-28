# Gladia Skills

Official AI agent skills from the Gladia team for integrating speech-to-text transcription into your applications.

## Installation

### Claude Code

Add the marketplace and install the plugin:

```
/plugin marketplace add gladiaio/skills
/plugin install gladia
```

### Cursor

1. Open Cursor Settings (Cmd+Shift+J / Ctrl+Shift+J)
2. Navigate to **Rules & Commands** → **Project Rules** → **Add Rule** → **Remote Rule (GitHub)**
3. Enter: `https://github.com/gladiaio/skills.git`

Skills are automatically discovered and used by the agent based on context. When you ask questions about Gladia transcription, the agent will select the relevant skill.

### Any Agent (via skills CLI)

```bash
npx skills add gladiaio/skills
```

## Available Skills

All skills default to **SDK-based integration** (`@gladiaio/sdk` for JS/TS, `gladiaio-sdk` for Python). Raw REST/WebSocket is only used as a fallback when the SDK cannot satisfy the requirement.

| Skill                          | When it activates                                                                                      |
| ------------------------------ | ------------------------------------------------------------------------------------------------------ |
| **pre-recorded-transcription** | Batch processing audio files, file uploads, URL transcription, diarization, PII redaction, subtitles   |
| **live-transcription**         | Real-time streaming, WebSocket sessions, voice agents, meeting recorders, call centers                 |
| **sdk-integration**            | Installing/configuring the JS/TS or Python SDK, client setup, error handling, SDK vs raw API decisions |
| **troubleshooting**            | Errors, common gotchas, performance issues, format problems, billing questions                         |
| **documentation-auto**         | General-purpose fallback — comprehensive reference auto-synced from docs.gladia.io                     |

## How It Works

Skills are **not** slash commands — they are automatically discovered from the `description` field in each skill's frontmatter. When you ask your AI agent a question related to Gladia, it matches your query against skill descriptions and loads the relevant skill(s).

## Keeping Skills Up to Date

The `documentation-auto` skill is automatically synced from [docs.gladia.io](https://docs.gladia.io) via a daily GitHub Actions workflow. When the documentation changes, a PR is opened to update the skill content.

To update skills in your project:

```bash
npx skills update
```

## Resources

- [Gladia Documentation](https://docs.gladia.io)
- [Support Center](https://support.gladia.io)
- [JavaScript SDK](https://www.npmjs.com/package/@gladiaio/sdk)
- [Python SDK](https://pypi.org/project/gladiaio-sdk/)
- [SDK Source Code](https://github.com/gladiaio/sdk)
- [Code Samples](https://github.com/gladiaio/gladia-samples)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on adding or modifying skills.

## License

MIT
