---
name: gladia-documentation-auto
description: Comprehensive Gladia speech-to-text reference auto-synced from docs.gladia.io. Use as a general-purpose fallback when other specialized skills don't match, or when the user needs a broad overview of Gladia capabilities, endpoints, decision guidance, or workflows. Always prefer the official SDK; fall back to raw REST/WebSocket only when SDK cannot satisfy the requirement.
license: MIT
metadata:
  source: https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
  digest: sha256:9e5ca5fb1b03bd2d1925414ba826fb1e5ac005bb112c25ff88a66274a1b18b9a
  synced: "2026-11-14"
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
  audio or video files asynchronously, streaming live audio in real-time,
  extracting insights from audio (diarization, translation, sentiment), or
  integrating transcription into voice agents and meeting applications.
metadata:
  mintlify-proj: gladia
  version: "1.0"
---

# Gladia Skill

## Product summary

Gladia is a speech-to-text (STT) API that transcribes audio and video into text with optional audio intelligence features (diarization, translation, sentiment analysis, PII redaction, summarization). It supports two modes: **pre-recorded** (async batch transcription) and **live** (real-time WebSocket streaming). Use the official SDKs (Python/JavaScript), REST API, CLI, or integrations with platforms like Pipecat, LiveKit, Vapi, and Twilio. Primary docs: https://docs.gladia.io

**Key files and commands:**
- API key: `x-gladia-key` header (get from https://app.gladia.io/apikeys)
- SDK: `npm install @gladiaio/sdk` (JavaScript) or `pip install gladiaio-sdk` (Python)
- CLI: `gladia transcribe <file-or-url>` (install from GitHub releases)
- Endpoints: `POST /v2/pre-recorded`, `POST /v2/live`, `GET /v2/pre-recorded/:id`, `GET /v2/live/:id`
- Models: `solaria-3` (pre-recorded only, 5 European languages, highest accuracy) or `solaria-1` (default, 100+ languages, live + pre-recorded, code switching)

## When to use

Reach for Gladia when:
- Building transcription into a product (voice agents, meeting apps, accessibility features)
- Processing recorded audio/video files asynchronously (batch jobs, webhooks, callbacks)
- Streaming live audio in real-time (WebSocket sessions, up to 3 hours per session)
- Extracting structured data from speech (speaker labels, translations, entities, sentiment, summaries)
- Migrating from Deepgram or AssemblyAI (migration guides available)
- Testing transcription quality before full integration (use the playground at https://app.gladia.io/playground)

Do not use Gladia for: authentication, billing, dashboard-only operations, or account management.

## Quick reference

### SDK usage (simplest path)

**Pre-recorded (one call):**
```javascript
const gladia = new GladiaClient({ apiKey: "YOUR_KEY" });
const result = await gladia.preRecorded().transcribe("audio.mp3");
```

**Live (streaming):**
```javascript
const session = gladia.liveV2().startSession({
  encoding: "wav/pcm", sample_rate: 16000, bit_depth: 16, channels: 1
});
session.on("message", (msg) => { if (msg.type === "transcript") console.log(msg); });
session.sendAudio(chunk);
session.stopRecording();
```

### CLI (no code)

```bash
gladia auth set YOUR_KEY
gladia transcribe meeting.wav                    # text to stdout
gladia transcribe podcast.mp3 -o srt             # subtitles
gladia transcribe call.wav --diarize --model solaria-3 --language en
```

### API authentication

All requests require the `x-gladia-key` header:
```bash
curl -H "x-gladia-key: YOUR_KEY" https://api.gladia.io/v2/pre-recorded
```

### Pre-recorded workflow

1. **Upload** (optional): `POST /v2/upload` → get `audio_url`
2. **Create job**: `POST /v2/pre-recorded` with `audio_url` and options → get `id`
3. **Poll or wait**: `GET /v2/pre-recorded/:id` until `status: "done"` OR use webhooks/callbacks

### Live workflow

1. **Init session**: `POST /v2/live` with audio config → get WebSocket `url` and `id`
2. **Connect**: Open WebSocket to `url`
3. **Send audio**: Send binary or base64-encoded chunks
4. **Read messages**: Handle `transcript`, `speech_start`, `speech_end`, etc.
5. **Stop**: Send `stop_recording` message or close socket with code 1000

### Model selection

| Use case | Model | Languages | Modes | Code switching |
|----------|-------|-----------|-------|---|
| European real-world audio, highest accuracy | `solaria-3` | EN, FR, DE, ES, IT | Pre-recorded only | No |
| Global coverage, live streaming, mixed languages | `solaria-1` | 100+ | Pre-recorded + live | Yes |

### Audio intelligence features

Enable on pre-recorded or live by adding to init request:
- `diarization: true` — speaker labels
- `translation: true` + `translation_config.target_languages: ["fr", "es"]` — multilingual
- `pii_redaction: true` — anonymize sensitive data
- `sentiment_analysis: true` — tone and emotions
- `summarization: true` — bullet points or concise summary
- `named_entity_recognition: true` — extract people, orgs, dates
- `custom_vocabulary: true` + `custom_vocabulary_config.vocabulary: ["Gladia", "Solaria"]` — boost accuracy for domain terms
- `subtitles: true` + `subtitles_config.formats: ["srt", "vtt"]` — SRT/VTT export

### Limits and concurrency

| Limit | Free tier | Paid tier |
|-------|-----------|-----------|
| Pre-recorded concurrency | 3 parallel jobs | 25 parallel jobs |
| Pre-recorded queue | — | 300 queued jobs |
| Live sessions | 1 concurrent | 30 concurrent |
| Session duration | 3 hours max | 3 hours max |
| Audio duration per job | 135 minutes | 135 minutes (4h15 for enterprise) |
| File size | 1000 MB max | 1000 MB max |
| Channels | 2 (mono/stereo) | 2 pre-recorded, 8 live |

## Decision guidance

### When to use pre-recorded vs live

| Scenario | Use pre-recorded | Use live |
|----------|---|---|
| Batch processing recorded files | ✓ | — |
| Real-time voice agent or call transcription | — | ✓ |
| Meeting recording after the fact | ✓ | — |
| Live captions or streaming | — | ✓ |
| Need solaria-3 accuracy | ✓ | — (not available) |
| Need 100+ languages or code switching | ✓ | ✓ (solaria-1 only) |

### When to use polling vs webhooks vs callbacks

| Approach | Best for | Trade-off |
|----------|----------|-----------|
| **Polling** (`GET /v2/pre-recorded/:id`) | Simple scripts, testing | Wastes API calls; higher latency |
| **Webhooks** (dashboard config) | Production, multiple jobs | Requires public endpoint; one-time setup |
| **Callbacks** (per-job config) | Production, per-job control | Requires public endpoint; more flexible |

### When to use SDK vs raw API

| Approach | Best for | Trade-off |
|----------|----------|-----------|
| **SDK** | Most integrations; handles retries, uploads, polling | Requires Node/Python; less control |
| **Raw API** (cURL, fetch) | Custom workflows, CI/CD, language not supported | Manual retry logic, more boilerplate |
| **CLI** | One-off transcriptions, shell scripts, testing | Pre-recorded only; no programmatic control |

## Workflow

### Transcribe a pre-recorded file (SDK)

1. **Get API key** from https://app.gladia.io/apikeys
2. **Install SDK**: `npm install @gladiaio/sdk` or `pip install gladiaio-sdk`
3. **Call transcribe()** with file path or URL:
   ```javascript
   const result = await gladia.preRecorded().transcribe("audio.mp3", {
     model: "solaria-3",
     language_config: { languages: ["en"] },
     diarization: true
   });
   ```
4. **Access result**: `result.result.transcription.full_transcript` (or utterances, diarization, etc.)
5. **Handle errors**: Check `result.status` for "done", "error", "processing", "queued"

### Stream live audio (SDK)

1. **Initialize session** with audio format:
   ```javascript
   const session = gladia.liveV2().startSession({
     encoding: "wav/pcm", sample_rate: 16000, bit_depth: 16, channels: 1,
     language_config: { languages: ["en"] }
   });
   ```
2. **Register handlers** for messages, errors, session lifecycle
3. **Send audio chunks** as they arrive: `session.sendAudio(buffer)`
4. **Process transcripts** as they arrive (partial and final)
5. **Stop recording** when done: `session.stopRecording()`
6. **Retrieve final results** via `GET /v2/live/:id` or wait for post-processing messages

### Transcribe via CLI

1. **Install**: `curl -fsSL https://github.com/gladiaio/gladia-cli/releases/latest/download/install.sh | sh`
2. **Set key**: `gladia auth set YOUR_KEY` or `export GLADIA_API_KEY=YOUR_KEY`
3. **Transcribe**: `gladia transcribe file.wav -o json` (or `-o srt`, `-o text`)
4. **Pipe output**: `gladia transcribe podcast.mp3 -o json | jq '.transcription'`

### Transcribe via raw API

1. **Upload audio** (if local file):
   ```bash
   curl -X POST https://api.gladia.io/v2/upload \
     -H "x-gladia-key: YOUR_KEY" \
     -F "audio=@file.mp3"
   ```
2. **Create job** with returned `audio_url`:
   ```bash
   curl -X POST https://api.gladia.io/v2/pre-recorded \
     -H "x-gladia-key: YOUR_KEY" \
     -H "Content-Type: application/json" \
     -d '{"audio_url": "...", "model": "solaria-3", "language_config": {"languages": ["en"]}}'
   ```
3. **Poll for result**:
   ```bash
   curl -H "x-gladia-key: YOUR_KEY" https://api.gladia.io/v2/pre-recorded/JOB_ID
   ```
4. **Check status** until `"done"` or `"error"`

## Common gotchas

- **Solaria-3 language constraint**: With `solaria-3`, pass exactly one language in `language_config.languages` (e.g., `["fr"]`). Do not enable code switching or pass multiple languages — it will fail.
- **Live sessions max 3 hours**: A single WebSocket session cannot exceed 3 hours. For longer events, start a new session before the limit.
- **Audio format mismatch**: The `encoding`, `sample_rate`, `bit_depth`, and `channels` in the init request must exactly match the audio you send. Mismatches cause silent failures or garbled transcripts.
- **Don't resubmit after 200 or webhook**: If `POST /v2/pre-recorded` returns 200 or you receive a `transcription.created` webhook, the job is accepted. Do not resubmit the same audio — wait on that job ID. Resubmitting creates duplicate billable jobs.
- **Polling vs webhooks**: Polling wastes API calls and increases latency. Use webhooks (dashboard) or callbacks (per-job) for production.
- **429 errors**: Both concurrency exhaustion and request-rate limiting return 429. Reduce parallel jobs/sessions for concurrency; slow the request rate for rate limiting. Use bounded retries with backoff.
- **Transient vs permanent errors**: Transient errors (timeouts, temporary 5xx) can be retried with backoff. Request/media errors (bad URL, corrupt file, invalid params) require fixing the input first — resubmitting the same payload will fail again.
- **Multi-channel billing**: Transcribing multi-channel audio (e.g., 2 channels) is billed as 2× the duration. Each channel is transcribed separately.
- **Partial vs final transcripts**: Partial transcripts are provisional; final transcripts complete an utterance. Use `is_final` to distinguish. Partial transcripts have the same `data.id` as their final counterpart.
- **Endpointing tuning**: Default `endpointing: 0.05` (50ms silence closes utterance) can split sentences if the speaker hesitates. Increase to 0.5–1.0 for more complete utterances; decrease for faster response.
- **Custom vocabulary phoneme-based**: Custom vocabulary works by matching how words sound, not spelling. Provide pronunciations if the default phoneme matching fails.

## Verification checklist

Before submitting transcription work:

- [ ] API key is set in environment, config file, or request header (`x-gladia-key`)
- [ ] Audio format (encoding, sample_rate, bit_depth, channels) matches the actual audio being sent
- [ ] For `solaria-3`: exactly one language in `language_config.languages`, no code switching
- [ ] For `solaria-1` with code switching: `code_switching: true` is set
- [ ] Pre-recorded: audio file is under 135 minutes and 1000 MB; live: session under 3 hours
- [ ] Callback or webhook URL is reachable and returns 2xx status (if using callbacks/webhooks)
- [ ] Job status is checked before resubmitting (do not resubmit after 200 or `transcription.created`)
- [ ] Audio intelligence features (diarization, translation, etc.) are enabled if needed
- [ ] For live: WebSocket connection is open and audio chunks are being sent
- [ ] Error handling is in place for transient (retry) vs permanent (fix input) failures
- [ ] Concurrency limits are respected (3–25 pre-recorded jobs, 1–30 live sessions depending on plan)

## Resources

**Comprehensive page listing:** https://docs.gladia.io/llms.txt

**Critical documentation:**
1. [Pre-recorded STT quickstart](https://docs.gladia.io/chapters/pre-recorded-stt/quickstart) — upload, create jobs, poll, webhooks, audio intelligence
2. [Live STT quickstart](https://docs.gladia.io/chapters/live-stt/quickstart) — WebSocket streaming, partial transcripts, session lifecycle
3. [API Reference](https://docs.gladia.io/api-reference/) — authentication, endpoints, request/response schemas

---

> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
---

> This file is auto-synced from https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
> Do not edit manually — changes will be overwritten by CI.
> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
