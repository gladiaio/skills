---
name: gladia-documentation-auto
description: Comprehensive Gladia speech-to-text reference auto-synced from docs.gladia.io. Use as a general-purpose fallback when other specialized skills don't match, or when the user needs a broad overview of Gladia capabilities, endpoints, decision guidance, or workflows. Always prefer the official SDK; fall back to raw REST/WebSocket only when SDK cannot satisfy the requirement.
license: MIT
metadata:
  source: https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
  digest: sha256:9b1e6b8c785c6daa380a50f9d4cc3251b4375beff2c2a56365f8d05bee7e7fc6
  synced: "2026-09-19"
---

> **SDK-first**: always use the official SDK — see [gladia-sdk-integration](../gladia-sdk-integration/SKILL.md) for policy, setup, and fallback criteria.

## References

Consult these sibling skills as needed:

- ../gladia-sdk-integration/SKILL.md -- SDK setup, client initialization, error handling, and SDK vs raw API decision guide
- ../gladia-sdk-integration/references/sdk-versions.md -- Current SDK versions (auto-synced by CI)
- ../gladia-troubleshooting/SKILL.md -- Common errors, gotchas, and verification checklist
- ../gladia-live-transcription/SKILL.md -- Live streaming transcription
- ../gladia-pre-recorded-transcription/SKILL.md -- Pre-recorded file transcription

---
name: gladia
description: Use when building speech-to-text transcription features, processing audio files or live streams, extracting insights from audio (diarization, translation, sentiment), or integrating real-time voice capabilities into applications. Agents should reach for this skill when users need to transcribe audio, analyze conversations, or build voice-enabled features.
metadata:
    mintlify-proj: gladia
    version: "1.0"
---

# Gladia Skill

## Product summary

Gladia is a speech-to-text (STT) API that transcribes audio files and live streams with optional audio intelligence features (diarization, translation, sentiment analysis, summarization, entity recognition). It offers two transcription modes: **pre-recorded** (async, batch) and **live** (real-time streaming via WebSocket). Two models are available: **Solaria-3** (highest accuracy on European real-world audio, pre-recorded only, 5 languages) and **Solaria-1** (default, 100+ languages, live + pre-recorded, code switching support). Use the official SDKs (Python/JavaScript) for simplest integration, or call REST/WebSocket APIs directly. Authentication uses `x-gladia-key` header. Primary docs: https://docs.gladia.io

## When to use

Reach for this skill when:
- User needs to transcribe audio files (MP3, WAV, etc.) or live audio streams
- Building voice agents, meeting transcription, call recording, or live captions
- Extracting structured data from audio (speaker identification, translations, sentiment, key entities, summaries)
- Choosing between async (pre-recorded) vs real-time (live) transcription modes
- Selecting between Solaria-3 (European business audio) or Solaria-1 (global, multilingual)
- Handling multi-language or code-switching scenarios
- Implementing webhooks/callbacks instead of polling for job completion
- Working with multi-channel audio (multiple speakers/participants)

## Quick reference

### SDKs and CLI

| Tool | Install | Use case |
|------|---------|----------|
| **Python SDK** | `pip install gladiaio-sdk` | Pre-recorded and live transcription in Python |
| **JavaScript SDK** | `npm install @gladiaio/sdk` | Pre-recorded and live transcription in TypeScript/JS |
| **Gladia CLI** | Download from GitHub releases | Terminal-based transcription without app code |
| **REST API** | Direct HTTP calls | Language-agnostic, full control |

### Authentication

Pass API key in header for all requests:
```
x-gladia-key: YOUR_GLADIA_API_KEY
```

Get key from Gladia dashboard or set `GLADIA_API_KEY` environment variable.

### Pre-recorded workflow (async)

1. **Upload** audio file → get `audio_url`
2. **Create job** with `audio_url` + options → get job `id`
3. **Poll or callback** until status is `done`
4. **Retrieve result** with transcription, metadata, and audio intelligence

### Live workflow (real-time)

1. **Init session** with audio config (encoding, sample_rate, bit_depth, channels) → get WebSocket URL
2. **Connect** to WebSocket
3. **Send audio chunks** as binary or base64-encoded JSON
4. **Receive messages** (transcripts, events, post-processing results)
5. **Stop recording** when done; WebSocket closes after post-processing

### Model selection

| Model | Mode | Languages | Code switching | Best for |
|-------|------|-----------|-----------------|----------|
| **solaria-3** | Pre-recorded only | EN, FR, DE, ES, IT | No | European real-world audio (calls, meetings) |
| **solaria-1** | Pre-recorded + live | 100+ | Yes | Global coverage, live streaming, clean speech |

### Audio Intelligence features

| Feature | Pre-recorded | Live | Config key |
|---------|--------------|------|-----------|
| Speaker diarization | ✓ | ✗ | `diarization: true` |
| Translation | ✓ | ✓ | `translation: true` + `translation_config` |
| Sentiment analysis | ✓ | ✓ | `sentiment_analysis: true` |
| Named entity recognition | ✓ | ✓ | `named_entity_recognition: true` |
| Summarization | ✓ | ✓ | `summarization: true` + `summarization_config` |
| Subtitles (SRT/VTT) | ✓ | ✗ | `subtitles: true` + `subtitles_config` |
| Custom vocabulary | ✓ | ✓ | `custom_vocabulary: true` + `custom_vocabulary_config` |
| Custom spelling | ✓ | ✓ | `custom_spelling: true` + `custom_spelling_config` |
| PII redaction | ✓ | ✗ | `pii_redaction: true` + `pii_redaction_config` |

### Job status values

- `queued` — waiting to process
- `processing` — actively transcribing
- `done` — complete, result available
- `error` — failed, check error_code and message

### Limits and quotas

| Limit | Value | Notes |
|-------|-------|-------|
| Pre-recorded max duration | 135 min (standard), 4h15 (enterprise) | Longer files rejected |
| Live session max duration | 3 hours | Start new session after 3h |
| Pre-recorded max file size | 1000 MB | Larger files rejected |
| Pre-recorded channels | 1–2 (mono/stereo) | Billed per channel |
| Live channels | 1–8 | Billed per channel |
| Concurrent live sessions | 30 (default) | Contact sales for increase |
| Concurrent pre-recorded jobs | 25 (default) | 300 queued max |

## Decision guidance

### When to use pre-recorded vs live

| Scenario | Use | Reason |
|----------|-----|--------|
| User uploads a file to transcribe | Pre-recorded | Async, simpler workflow, supports all features |
| Real-time voice call or meeting | Live | Streaming, low-latency, WebSocket-based |
| Batch processing many files | Pre-recorded | Queue up to 300 jobs, process 25 concurrently |
| Immediate transcription needed | Live | Partial transcripts available in real-time |

### When to use Solaria-3 vs Solaria-1

| Scenario | Use | Reason |
|----------|-----|--------|
| European business audio (calls, meetings) | Solaria-3 | Highest accuracy on real-world noisy audio |
| Need live transcription | Solaria-1 | Solaria-3 pre-recorded only |
| Multiple languages or code switching | Solaria-1 | Solaria-3 limited to EN/FR/DE/ES/IT |
| Global coverage, any domain | Solaria-1 | 100+ languages, default model |
| Clean, formal speech | Solaria-1 | Optimized for read speech |

### When to use polling vs callbacks vs webhooks

| Method | Use | Pros | Cons |
|--------|-----|------|------|
| **Polling** | Simple scripts, testing | No setup, immediate control | Wastes resources, high latency |
| **Callbacks** | Per-job notification | Lightweight, per-request config | Requires public endpoint |
| **Webhooks** | Global notification | Centralized, reusable | Requires dashboard setup |

## Workflow

### Pre-recorded transcription (SDK)

1. **Initialize client** with API key
2. **Call `transcribe()`** with audio file/URL and options (model, language, features)
3. **SDK handles** upload, job creation, polling, result retrieval
4. **Extract result** from response: `result.result.transcription.full_transcript`

### Pre-recorded transcription (API)

1. **POST to `/v2/upload`** with audio file → get `audio_url`
2. **POST to `/v2/pre-recorded`** with `audio_url` + config → get job `id`
3. **Poll `GET /v2/pre-recorded/{id}`** until `status === "done"`
4. **Extract transcription** from `result.transcription.full_transcript`

### Live transcription (SDK)

1. **Initialize client** with API key
2. **Call `liveV2().startSession(config)`** with audio format (encoding, sample_rate, bit_depth, channels)
3. **Register message handlers** (`on("message")`, `on("error")`, `on("started")`, `on("ended")`)
4. **Send audio chunks** via `session.sendAudio(chunk)`
5. **Process messages** as they arrive; check `message.data.is_final` to distinguish partial vs final
6. **Call `session.stopRecording()`** when done; WebSocket closes after post-processing

### Live transcription (API)

1. **POST to `/v2/live`** with audio config → get WebSocket `url` and `id`
2. **Connect WebSocket** to returned `url`
3. **Send audio** as binary or JSON with base64-encoded chunk
4. **Listen for messages** (JSON-formatted); parse `type` and `data` fields
5. **Send `stop_recording`** message or close socket with code 1000
6. **Retrieve final result** via `GET /v2/live/{id}` after post-processing

## Common gotchas

- **Solaria-3 with multiple languages**: Solaria-3 does not support code switching. Pass exactly one language in `language_config.languages` (e.g., `["fr"]`), not multiple.
- **Live session timeout**: Live sessions terminate after 3 hours. For longer events, start a new session before hitting the limit.
- **Audio format mismatch**: Encoding, sample_rate, bit_depth, and channels must match actual audio. Mismatches cause silent failures or garbled transcription.
- **Multi-channel billing**: Each channel is billed separately. A 2-channel 10-minute audio costs 20 minutes of billing time.
- **Polling without SDK**: Manual polling can be slow and resource-intensive. Use callbacks or webhooks instead, or use the SDK's built-in polling.
- **API key in client code**: Never expose API key in frontend code. Generate WebSocket URL on backend, pass secure URL to client.
- **Partial transcripts disabled by default**: Set `messages_config.receive_partial_transcripts: true` to get low-latency intermediate results.
- **Job status not "done" immediately**: Pre-recorded jobs are async. Status starts as `queued`, moves to `processing`, then `done`. Poll or use callbacks.
- **Deprecated transcription endpoint**: `/v2/transcription` is deprecated. Use `/v2/pre-recorded` instead.
- **Audio intelligence errors silently**: If an audio intelligence feature fails (e.g., translation), the result includes an `error` object. Check `success` and `error` fields in response.

## Verification checklist

Before submitting work with Gladia:

- [ ] API key is set and valid (test with a simple request)
- [ ] Audio format (encoding, sample_rate, bit_depth, channels) matches actual audio
- [ ] Model choice matches use case (Solaria-3 for European audio, Solaria-1 for live or multilingual)
- [ ] Language config is correct (single language for Solaria-3, multiple or auto-detect for Solaria-1)
- [ ] Audio file size < 1000 MB and duration < 135 min (or 4h15 for enterprise)
- [ ] Live session duration < 3 hours (or split into multiple sessions)
- [ ] Callback/webhook URL is public and reachable (if using callbacks)
- [ ] Message config is set correctly (e.g., `receive_partial_transcripts: true` if partials needed)
- [ ] Audio intelligence features are enabled if required (diarization, translation, etc.)
- [ ] Error handling is in place (check job status, handle `error` responses, retry on network failure)
- [ ] SDK is used for simplicity, or API calls are properly authenticated with `x-gladia-key` header

## Resources

- **Full documentation**: https://docs.gladia.io/llms.txt (comprehensive page-by-page navigation)
- **Pre-recorded quickstart**: https://docs.gladia.io/chapters/pre-recorded-stt/quickstart
- **Live quickstart**: https://docs.gladia.io/chapters/live-stt/quickstart
- **API reference**: https://docs.gladia.io/api-reference/
- **SDK samples**: https://github.com/gladiaio/gladia-samples (Python, TypeScript, JavaScript)

---

> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
---

> This file is auto-synced from https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
> Do not edit manually — changes will be overwritten by CI.
> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
