---
name: gladia-documentation-auto
description: Comprehensive Gladia speech-to-text reference auto-synced from docs.gladia.io. Use as a general-purpose fallback when other specialized skills don't match, or when the user needs a broad overview of Gladia capabilities, endpoints, decision guidance, or workflows. Always prefer the official SDK; fall back to raw REST/WebSocket only when SDK cannot satisfy the requirement.
license: MIT
metadata:
  source: https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
  digest: sha256:9e5ca5fb1b03bd2d1925414ba826fb1e5ac005bb112c25ff88a66274a1b18b9a
  synced: "2026-11-07"
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
description: Use when building speech-to-text transcription features, processing
  audio or video files asynchronously, streaming live audio for real-time
  transcription, extracting insights from audio (diarization, translation,
  sentiment), or integrating transcription into voice agents and meeting
  applications.
metadata:
  mintlify-proj: gladia
  version: "1.0"
---

# Gladia Skill

## Product summary

Gladia is a speech-to-text (STT) API that transcribes audio and video into text with optional audio intelligence features (diarization, translation, sentiment analysis, PII redaction, summarization). It supports two modes: **pre-recorded** (asynchronous batch transcription) and **live** (real-time WebSocket streaming). Use the official SDKs (JavaScript/TypeScript, Python), REST API, CLI, or integrations with platforms like Pipecat, Livekit, Vapi, and Twilio. Authentication uses a single API key passed in the `x-gladia-key` header. Primary docs: https://docs.gladia.io

## When to use

Reach for Gladia when:
- Building asynchronous transcription for uploaded audio/video files (pre-recorded mode)
- Implementing real-time speech-to-text for live audio streams (live mode with WebSocket)
- Extracting structured data from audio (speaker identification, translation, sentiment, entities)
- Integrating transcription into voice agents, meeting recorders, or telephony systems
- Processing audio in batch workflows with webhooks or callbacks for result notification
- Needing multi-language support (100+ languages with Solaria-1, or 5 European languages with Solaria-3)
- Requiring speaker diarization, PII redaction, or subtitle generation

Do not use Gladia for: real-time text-to-speech synthesis, video editing, or audio generation.

## Quick reference

### Models

| Model | Best for | Modes | Languages | Code switching | Accuracy focus |
|-------|----------|-------|-----------|-----------------|-----------------|
| **solaria-3** | European real-world audio (calls, meetings) | Pre-recorded only | EN, FR, DE, ES, IT | No | Highest accuracy on noisy production audio |
| **solaria-1** (default) | Global coverage, clean speech | Pre-recorded + live | 100+ | Yes | Multilingual, live streaming, formal speech |

### Authentication

```bash
# Pass API key in header for all requests
x-gladia-key: YOUR_GLADIA_API_KEY

# Or set environment variable
export GLADIA_API_KEY=your_key
```

### Pre-recorded workflow (async)

1. **Upload** (optional): `POST /v2/upload` with audio file → returns `audio_url`
2. **Create job**: `POST /v2/pre-recorded` with `audio_url` and options → returns `id`
3. **Poll/wait**: `GET /v2/pre-recorded/{id}` until status is `done`, or use webhooks/callbacks

### Live workflow (streaming)

1. **Init session**: `POST /v2/live` with audio config → returns WebSocket URL
2. **Connect**: Open WebSocket, send audio chunks
3. **Read messages**: Receive transcript events (partial and final)
4. **Stop**: Send `stop_recording` message, WebSocket closes with code 1000

### CLI commands

```bash
gladia auth set YOUR_API_KEY
gladia transcribe audio.mp3                    # text output
gladia transcribe audio.mp3 -o json            # JSON output
gladia transcribe audio.mp3 -o srt             # subtitles
gladia transcribe audio.mp3 --diarize          # speaker labels
gladia transcribe audio.mp3 --language en,fr   # constrain detection
gladia languages                               # list supported codes
```

### Audio Intelligence features

| Feature | Pre-recorded | Live | Use case |
|---------|--------------|------|----------|
| **Diarization** | ✓ | ✓ | Identify speakers, label who said what |
| **Translation** | ✓ | ✓ | Translate transcript to 100+ languages |
| **Sentiment analysis** | ✓ | ✓ | Extract tone and emotions per sentence |
| **PII redaction** | ✓ | — | Anonymize names, emails, IDs |
| **Subtitles** | ✓ | — | Generate SRT/VTT files |
| **Custom vocabulary** | ✓ | ✓ | Boost accuracy for domain terms |
| **Custom spelling** | ✓ | ✓ | Replace text variants |
| **Named entity recognition** | ✓ | ✓ | Extract people, organizations, dates |
| **Summarization** | ✓ | — | Generate summaries or bullet points |

## Decision guidance

### When to use pre-recorded vs. live

| Scenario | Use pre-recorded | Use live |
|----------|-----------------|----------|
| Uploaded file, batch processing | ✓ | — |
| Real-time streaming audio | — | ✓ |
| Need highest accuracy (solaria-3) | ✓ | — |
| Need 100+ languages or code switching | ✓ (solaria-1) | ✓ (solaria-1 only) |
| Voice agent, telephony, live captions | — | ✓ |
| Can wait for async result | ✓ | — |
| Need immediate partial transcripts | — | ✓ |

### When to use polling vs. webhooks vs. callbacks

| Approach | Best for | Trade-off |
|----------|----------|-----------|
| **Polling** (`GET /v2/pre-recorded/{id}`) | Simple integrations, low volume | Requires client-side loop, higher latency |
| **Webhooks** (dashboard config) | Production, event-driven | Requires public endpoint, one webhook per account |
| **Callbacks** (per-job config) | Flexible, per-job routing | Must handle retries, no built-in deduplication |

### When to use SDK vs. raw API

| Approach | Best for | Trade-off |
|----------|----------|-----------|
| **SDK** (Python, JavaScript) | Most integrations | Abstracts upload, polling, WebSocket lifecycle |
| **Raw API** (REST, WebSocket) | Custom workflows, non-SDK languages | More control, manual job management |
| **CLI** | Terminal, scripts, CI/CD | Pre-recorded only, no programmatic control |

## Workflow

### Pre-recorded transcription (SDK)

1. **Install SDK**: `npm install @gladiaio/sdk` or `pip install gladiaio-sdk`
2. **Set API key**: Environment variable `GLADIA_API_KEY` or pass to client
3. **Transcribe in one call**: `gladia.preRecorded().transcribe("audio.mp3")` handles upload, job creation, and polling
4. **Add options**: Pass `model`, `language_config`, `diarization`, `translation`, etc.
5. **Handle result**: Access `result.transcription.full_transcript` and audio intelligence fields
6. **Verify**: Check `result.status` is `done` and no error fields present

### Live transcription (SDK)

1. **Install SDK**: `npm install @gladiaio/sdk` or `pip install gladiaio-sdk`
2. **Start session**: `gladia.liveV2().startSession({encoding, sample_rate, bit_depth, channels})`
3. **Register handlers**: Listen for `message`, `error`, `started`, `ended` events
4. **Send audio**: Call `session.sendAudio(chunk)` in a loop as audio arrives
5. **Read transcripts**: On `message` events, check `message.type === "transcript"` and `message.data.is_final`
6. **Stop session**: Call `session.stopRecording()` when done
7. **Verify**: Confirm WebSocket closes with code 1000 and final results received

### Pre-recorded with raw API

1. **Upload audio** (if local file): `POST /v2/upload` with multipart form data
2. **Create job**: `POST /v2/pre-recorded` with `audio_url` and config
3. **Store job ID**: Save the returned `id` for polling or callback routing
4. **Poll or wait**: Either `GET /v2/pre-recorded/{id}` in a loop, or configure webhooks/callbacks
5. **Check status**: Poll until `status === "done"` or receive success webhook
6. **Extract results**: Access `transcription.full_transcript`, `diarization`, `translation`, etc.
7. **Handle errors**: If status is `error`, inspect error details before retrying

## Common gotchas

- **Do not resubmit after 200 or `transcription.created`**: Once `POST /v2/pre-recorded` returns 200 or you receive the webhook, the job is queued. Resubmitting creates a duplicate billable job. Store the ID and wait on that job.
- **Solaria-3 requires single language**: With `solaria-3`, pass exactly one language in `language_config.languages` (e.g., `["fr"]`). Do not enable code switching or pass multiple languages.
- **Live sessions max 3 hours**: A single live WebSocket session cannot exceed 3 hours. Start a new session for longer events.
- **Audio config must match actual audio**: `encoding`, `sample_rate`, `bit_depth`, and `channels` in the init request must match the audio you send. Mismatch causes silent failures or garbled transcripts.
- **Partial transcripts are provisional**: With `receive_partial_transcripts: true`, partial transcripts (`is_final: false`) may change. Only finalize display when `is_final: true`.
- **Concurrency limits apply**: Free tier: 3 pre-recorded jobs, 1 live session. Paid: 25 pre-recorded, 30 live. HTTP 429 means you hit the limit; back off and retry.
- **No deduplication**: Gladia does not deduplicate jobs. Each accepted POST is a separate billable job, even if the audio is identical.
- **Multi-channel billing**: Transcribing a 2-channel audio file is billed as 2 audios. Same for live multi-channel streams.
- **Callback retries**: Callbacks are not automatically retried by Gladia. Your endpoint must handle retries and idempotency.
- **Language detection adds latency**: If you don't specify a language, Gladia auto-detects it, which adds processing time. Set `language_config.languages` if you know the language.

## Verification checklist

Before submitting transcription work:

- [ ] API key is set in environment or passed to client
- [ ] Audio file is accessible (local path or valid URL)
- [ ] Audio format is supported (MP3, WAV, FLAC, OGG, M4A, etc.)
- [ ] For pre-recorded: job ID is stored after POST returns 200
- [ ] For live: audio config (`encoding`, `sample_rate`, `bit_depth`, `channels`) matches actual audio
- [ ] For solaria-3: exactly one language in `language_config.languages`, no code switching
- [ ] For live: WebSocket connection opens without errors
- [ ] Transcript messages are received and `is_final` field is checked
- [ ] Error handling is in place (check `status`, `error` fields, HTTP status codes)
- [ ] Concurrency limits are respected (no more than plan allows)
- [ ] Webhooks/callbacks are configured if using async notification
- [ ] Results are not re-polled after job is done (avoid unnecessary API calls)

## Resources

- **Full page navigation**: https://docs.gladia.io/llms.txt
- **SDK quickstart**: https://docs.gladia.io/chapters/how-to-use-gladia/sdk
- **Pre-recorded API reference**: https://docs.gladia.io/api-reference/v2/pre-recorded/init
- **Live API reference**: https://docs.gladia.io/api-reference/v2/live/init
- **Audio Intelligence features**: https://docs.gladia.io/chapters/audio-intelligence/
- **Concurrency and limits**: https://docs.gladia.io/chapters/limits-and-specifications/concurrency
- **Transcription process & retry policy**: https://docs.gladia.io/chapters/pre-recorded-stt/transcription-process

---

> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
---

> This file is auto-synced from https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
> Do not edit manually — changes will be overwritten by CI.
> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
