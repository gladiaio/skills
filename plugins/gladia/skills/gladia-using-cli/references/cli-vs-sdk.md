# CLI vs SDK Feature Matrix

Use this table to decide whether to run `gladia transcribe` or escalate to SDK skills.

## Contents

- [CLI vs SDK Feature Matrix](#cli-vs-sdk-feature-matrix)
  - [Contents](#contents)
  - [Transcription modes](#transcription-modes)
  - [Audio intelligence features](#audio-intelligence-features)
  - [Output and post-processing](#output-and-post-processing)
  - [When to escalate](#when-to-escalate)

## Transcription modes

| Need                               | CLI                          | SDK / other skill                                                                  |
| ---------------------------------- | ---------------------------- | ---------------------------------------------------------------------------------- |
| Pre-recorded file or URL           | `gladia transcribe <source>` | [gladia-pre-recorded-transcription](../gladia-pre-recorded-transcription/SKILL.md) |
| Live / real-time stream            | Not supported                | [gladia-live-transcription](../gladia-live-transcription/SKILL.md)                 |
| Batch jobs with webhooks           | Not supported                | SDK `transcribe()` + `callback_url`                                                |
| Job management (list, delete, get) | Not supported                | SDK `get()`, `list()`, `delete()`                                                  |

## Audio intelligence features

| Feature                   | CLI flag / behavior            | If not in CLI                                                                                                     |
| ------------------------- | ------------------------------ | ----------------------------------------------------------------------------------------------------------------- |
| Basic transcription       | `gladia transcribe`            | —                                                                                                                 |
| Speaker diarization       | `--diarize`                    | [gladia-audio-intelligence](../gladia-audio-intelligence/SKILL.md) for `diarization_config` (speaker count, etc.) |
| Language hints            | `--language en,fr`             | SDK `language_config.languages`                                                                                   |
| Code switching            | `--code-switching`             | SDK `language_config.code_switching`                                                                              |
| Translation               | No                             | [gladia-audio-intelligence](../gladia-audio-intelligence/SKILL.md) — `translation`                                |
| Summarization (API addon) | No                             | [gladia-audio-intelligence](../gladia-audio-intelligence/SKILL.md) — `summarization`                              |
| Named entity recognition  | No                             | [gladia-audio-intelligence](../gladia-audio-intelligence/SKILL.md) — `named_entity_recognition`                   |
| PII redaction             | No                             | [gladia-audio-intelligence](../gladia-audio-intelligence/SKILL.md) — `pii_redaction`                              |
| Sentiment analysis        | No                             | [gladia-audio-intelligence](../gladia-audio-intelligence/SKILL.md) — `sentiment_analysis`                         |
| Audio-to-LLM              | No                             | [gladia-audio-intelligence](../gladia-audio-intelligence/SKILL.md) — `audio_to_llm`                               |
| Custom vocabulary         | No                             | [gladia-audio-intelligence](../gladia-audio-intelligence/SKILL.md) — `custom_vocabulary`                          |
| Chapterization            | No                             | [gladia-audio-intelligence](../gladia-audio-intelligence/SKILL.md) — `chapterization`                             |
| Model selection           | `--model solaria-1\|solaria-3` | SDK `model` option on `transcribe()`                                                                              |

**Informal summarization:** the agent may summarize `-o text` output in conversation, but that is not the API summarization addon. For structured API summaries, use the SDK.

## Output and post-processing

| Need                       | CLI                     | SDK                                         |
| -------------------------- | ----------------------- | ------------------------------------------- |
| Plain text transcript      | `-o text` (default)     | `result.transcription.full_transcript`      |
| Utterances with timestamps | `-o json`               | `result.transcription.utterances`           |
| Full API response JSON     | `-o json-full`          | Full `get()` / `transcribe()` response      |
| SRT subtitles              | `-o srt`                | `subtitles` addon or format from utterances |
| VTT subtitles              | `-o vtt`                | `subtitles` addon or format from utterances |
| Diarized SRT/VTT           | `--diarize -o srt\|vtt` | `diarization: true` + subtitles config      |

## When to escalate

Escalate from CLI to SDK skills when:

1. User requests a feature with **no CLI flag** (see table above)
2. User is building an **application**, not a one-off terminal task
3. User needs **advanced diarization config** (exact speaker count, min/max speakers beyond CLI defaults)
4. User needs **async delivery** (webhooks, job polling in code)
5. `gladia` CLI is **not installed** and user does not want to install it

For setup and SDK vs raw API decisions, start with [gladia-sdk-integration](../gladia-sdk-integration/SKILL.md).
