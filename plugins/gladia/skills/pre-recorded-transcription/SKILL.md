---
name: pre-recorded-transcription
description: Transcribe pre-recorded audio files or URLs with Gladia. Use when the user needs batch/async transcription, speaker diarization, subtitles (SRT/VTT), PII redaction, translation, NER, summarization, chapterization, audio-to-LLM, or any audio intelligence on pre-recorded content. Always prefer the official SDK; fall back to raw REST only when SDK cannot satisfy the requirement.
license: MIT
---

# Pre-Recorded Transcription

Gladia's pre-recorded API transcribes audio and video files asynchronously.

> **SDK-first**: always use the official SDK — see [sdk-integration](../sdk-integration/SKILL.md) for policy, setup, and fallback criteria.

## When to Use

- User has an existing audio or video file (local file, URL, YouTube/social video) to transcribe
- Batch or async transcription workflows — processing recordings after they are captured
- Need audio intelligence features: speaker diarization, PII redaction, subtitles (SRT/VTT), summarization, translation, NER, chapterization, audio-to-LLM
- File-based uploads from disk, cloud storage, or user-submitted content

**When NOT to use:** If the user needs real-time / live transcription of a stream, microphone, or ongoing audio feed, use the [live-transcription skill](../live-transcription/SKILL.md) instead. Live transcription uses WebSocket sessions, not the pre-recorded API.

## References

Consult these resources as needed:

- ./references/transcription-options.md -- Full transcription options with JS/Python code examples
- ../audio-intelligence/SKILL.md -- Audio intelligence feature configs, availability matrix, and detailed per-feature reference
- ../sdk-integration/SKILL.md -- SDK setup, client initialization, error handling, retry/timeout config, and SDK vs raw API decision guide
- ../sdk-integration/references/sdk-versions.md -- Current SDK versions (auto-synced by CI)
- ../troubleshooting/SKILL.md -- Common errors, gotchas, and verification checklist

## API Endpoints (reference — prefer SDK methods instead)

The SDK wraps all these endpoints. Use them directly only when falling back to raw REST.

| Endpoint                     | Method | SDK equivalent                          |
| ---------------------------- | ------ | --------------------------------------- |
| `/v2/upload`                 | POST   | `transcribe()` auto-uploads local files |
| `/v2/pre-recorded`           | POST   | `create()` / `transcribe()`             |
| `/v2/pre-recorded/:id`       | GET    | `get()` / `poll()` / `transcribe()`     |
| `/v2/pre-recorded/:id`       | DELETE | `delete()`                              |
| `/v2/pre-recorded/:id/audio` | GET    | `getFile()`                             |

## Workflow

### Recommended (SDK)

The SDK `transcribe()` method handles upload, job creation, and polling in one call. **This is the default approach — use it unless you have a specific reason not to.** For SDK installation and client initialization, see the [sdk-integration skill](../sdk-integration/SKILL.md).

```typescript
const result = await client.preRecorded().transcribe("./audio.mp3", {
  language_config: { languages: ["en"] },
  diarization: true,
});

console.log(result.result?.transcription?.full_transcript);
```

```python
result = client.prerecorded().transcribe(
    "audio.mp3",
    {"language_config": {"languages": ["en"]}, "diarization": True},
)

print(result.result.transcription.full_transcript)
```

Audio input can be a local file path, HTTP(S) URL, YouTube/social video URL, or binary file object. For the full input types table, see the [sdk-integration skill](../sdk-integration/SKILL.md#audio-input-types). YouTube and social video URLs are passed as `audio_url`; Gladia extracts audio server-side.

### Fallback (raw REST — only when SDK is not feasible)

Use this path only when the SDK cannot satisfy the requirement (e.g., custom HTTP client, language without an SDK, or explicit user request for raw calls).

1. **Upload** (if local file): `POST /v2/upload` with multipart form data → get `audio_url`
2. **Create job**: `POST /v2/pre-recorded` with `audio_url` and config → get `id`
3. **Poll**: `GET /v2/pre-recorded/:id` until `status: "done"` (or use webhooks/callbacks)
4. **Parse results**: Extract `transcription`, `diarization`, `translation`, etc. from response

## Transcription Options

All options are passed as the second argument to `transcribe()`. Key options:

| Option            | Description                                 |
| ----------------- | ------------------------------------------- |
| `language_config` | Expected languages, code switching          |
| `diarization`     | Speaker identification (pre-recorded only)  |
| `translation`     | Translate to target languages               |
| `summarization`   | Generate bullet points or paragraph summary |
| `subtitles`       | Generate SRT/VTT files                      |
| `pii_redaction`   | Redact PII (pre-recorded only)              |
| `audio_to_llm`    | Run custom LLM prompts on transcript        |
| `callback_url`    | Async webhook delivery                      |

For the full options reference with JS/Python code examples, see [./references/transcription-options.md](./references/transcription-options.md). For detailed audio intelligence feature configuration, see [audio-intelligence](../audio-intelligence/SKILL.md). For client-level config (retry, timeouts), see [sdk-integration](../sdk-integration/SKILL.md#configuration-options).

## Response Structure

```json
{
  "id": "job-uuid",
  "status": "done",
  "result": {
    "transcription": {
      "full_transcript": "Hello, welcome to the meeting...",
      "utterances": [
        {
          "text": "Hello, welcome to the meeting",
          "language": "en",
          "start": 0.5,
          "end": 2.1,
          "speaker": 0,
          "words": [{ "word": "Hello", "start": 0.5, "end": 0.8, "confidence": 0.98 }]
        }
      ]
    },
    "diarization": { ... },
    "translation": { ... },
    "summarization": { ... },
    "sentiment_analysis": { ... }
  }
}
```

## Limits and Specifications

| Constraint              | Value                                                 |
| ----------------------- | ----------------------------------------------------- |
| Max file size           | 1000 MB                                               |
| Max duration            | 135 minutes (120 min for YouTube)                     |
| Enterprise max duration | 4h15                                                  |
| Supported audio formats | AAC, AC3, FLAC, M4A, MP2, MP3, OGG, Opus, WAV         |
| Supported video formats | MP4, MOV, AVI, FLV, WebM, Matroska, 3GP               |
| Online platforms        | YouTube, TikTok, Instagram, Facebook, Vimeo, LinkedIn |
| Concurrency (paid)      | 25 concurrent jobs                                    |
| Concurrency (free)      | 3 concurrent jobs                                     |

## Polling Best Practices

The SDK handles polling automatically — `transcribe()` polls until the job completes with configurable `interval` and `timeout`:

```typescript
const result = await client.preRecorded().transcribe(audio, options, {
  interval: 5000, // Poll every 5s
  timeout: 600000, // Timeout after 10 minutes
});
```

If using raw REST instead of the SDK:

- Use webhooks or callbacks instead of polling when possible
- If polling, implement exponential backoff (start at 3s, max 30s)

## Webhooks and Callbacks

**Callback** (sent to `callback_url` in request body):

- `transcription.success` — job completed successfully
- `transcription.error` — job failed

**Webhook** (configured in dashboard → Account → Webhooks):

- `transcription.created` — job queued
- `transcription.success` — job done
- `transcription.error` — job failed

Webhooks are powered by Svix with signed requests for verification.

## Common Mistakes

- **Code switching without language list**: enabling `code_switching: true` with empty `languages` triggers 100+ language evaluation. Always provide 3-5 expected languages.
- **Custom vocabulary intensity too high**: values above 0.6 cause false positives. Keep at 0.4-0.6.
- **Polling without backoff**: rapid polling wastes requests and may trigger 429s. The SDK handles this; for raw REST, use webhooks or exponential backoff.
- **Expecting live-only features**: diarization, PII redaction, and subtitles are pre-recorded only — not available in live mode.

For the full list of gotchas and diagnostics, see the [troubleshooting skill](../troubleshooting/SKILL.md).

## Further Reading

- [Pre-recorded quickstart](https://docs.gladia.io/chapters/pre-recorded-stt/quickstart)
- [Audio intelligence overview](https://docs.gladia.io/chapters/pre-recorded-stt/audio-intelligence)
- [Supported formats](https://docs.gladia.io/chapters/limits-and-specifications/supported-formats)
- [API reference](https://docs.gladia.io/api-reference/v2/pre-recorded/init)
