# Gladia Plugin

Official AI agent skills from the Gladia team for integrating speech-to-text transcription into your applications.

## Skills

All skills default to **SDK-based integration** (`@gladiaio/sdk` for JS/TS, `gladiaio-sdk` for Python). Raw REST/WebSocket is documented as a fallback only.

| Skill                        | Description                                                                                                                                  |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `pre-recorded-transcription` | Batch transcription of audio files and URLs with audio intelligence features (SDK-first)                                                     |
| `live-transcription`         | Real-time streaming transcription for voice agents, meetings, and call centers (SDK-first)                                                   |
| `audio-intelligence`         | Configure audio intelligence features (diarization, translation, NER, PII, subtitles, summarization) for pre-recorded and live transcription |
| `sdk-integration`            | Installing/configuring the official SDKs; SDK vs raw API decision guide                                                                      |
| `troubleshooting`            | Common gotchas, error resolution, and verification checklists (SDK-first diagnostics)                                                        |
| `documentation-auto`         | Auto-synced comprehensive reference from docs.gladia.io                                                                                      |

## How Skills Are Discovered

Skills are automatically selected by your AI agent based on the `description` field in each `SKILL.md` frontmatter. You don't need to manually activate them — just ask questions related to Gladia and the agent will pick the right skill.

## Resources

- [Gladia Documentation](https://docs.gladia.io)
- [Gladia Support Center](https://support.gladia.io)
- [Code Samples](https://github.com/gladiaio/gladia-samples)
- [SDK Repository](https://github.com/gladiaio/sdk)
