---
name: documentation-auto
description: Comprehensive Gladia speech-to-text reference auto-synced from docs.gladia.io. Use as a general-purpose fallback when other specialized skills don't match, or when the user needs a broad overview of Gladia capabilities, endpoints, decision guidance, or workflows. Always prefer the official SDK; fall back to raw REST/WebSocket only when SDK cannot satisfy the requirement.
license: MIT
metadata:
  source: https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
  digest: sha256:2242ad7aad2e0f154a45c2bfb3f78c59416e6a53377cfefc4fcebee31098093a
  synced: "2026-05-19"
---

> **SDK-first**: always use the official SDK — see [sdk-integration](../sdk-integration/SKILL.md) for policy, setup, and fallback criteria.

## References

Consult these sibling skills as needed:

- ../sdk-integration/SKILL.md -- SDK setup, client initialization, error handling, and SDK vs raw API decision guide
- ../sdk-integration/references/sdk-versions.md -- Current SDK versions (auto-synced by CI)
- ../troubleshooting/SKILL.md -- Common errors, gotchas, and verification checklist
- ../live-transcription/SKILL.md -- Live streaming transcription
- ../pre-recorded-transcription/SKILL.md -- Pre-recorded file transcription

# Gladia Speech-to-Text API

## Product summary

Gladia is a speech-to-text API that transcribes audio and video in real-time (WebSocket) or asynchronously (pre-recorded). It supports 100+ languages, multi-speaker detection, translation, sentiment analysis, and custom vocabulary. Use the official SDKs (`@gladiaio/sdk` for JavaScript, `gladiaio-sdk` for Python) or call REST endpoints directly. Authenticate with the `x-gladia-key` header. Primary endpoints: `POST /v2/live` (real-time init), `POST /v2/pre-recorded` (async transcription), `GET /v2/live/:id` and `GET /v2/pre-recorded/:id` (fetch results). See https://docs.gladia.io for full documentation.

## When to use

Reach for this skill when:

- Building real-time transcription for live audio streams (voice calls, meetings, broadcasts)
- Processing pre-recorded audio files asynchronously (podcasts, videos, archives)
- Extracting structured data from audio (speaker identification, sentiment, entities, summaries)
- Implementing multi-language support or code-switching detection
- Integrating with voice agents, meeting recorders, or call center platforms
- Needing to translate transcripts or generate subtitles
- Redacting PII from transcriptions for compliance

## Quick reference

### Authentication

Pass API key in header: `x-gladia-key: YOUR_GLADIA_API_KEY`

### SDK Installation

```bash
# JavaScript
npm install @gladiaio/sdk

# Python
pip install gladiaio-sdk
```

### Core Endpoints

| Endpoint               | Method | Purpose                                       |
| ---------------------- | ------ | --------------------------------------------- |
| `/v2/live`             | POST   | Initiate real-time session, get WebSocket URL |
| `/v2/live/:id`         | GET    | Fetch live session results                    |
| `/v2/pre-recorded`     | POST   | Create async transcription job                |
| `/v2/pre-recorded/:id` | GET    | Fetch pre-recorded results                    |
| `/v2/upload`           | POST   | Upload audio file (multipart form data)       |

### Audio Limits

- Pre-recorded: Max 135 minutes (120 min for YouTube), 1000 MB file size
- Live: Max 3 hours per session
- Enterprise: Up to 4h15 for pre-recorded

### Supported Formats

Audio: AAC, AC3, FLAC, M4A, MP2, MP3, OGG, Opus, WAV
Video: MP4, MOV, AVI, FLV, WebM, Matroska, 3GP, and online platforms (YouTube, TikTok, Instagram, Facebook, Vimeo, LinkedIn)

### Audio Intelligence Features

| Feature                  | Pre-recorded | Live | Config Key                       |
| ------------------------ | ------------ | ---- | -------------------------------- |
| Speaker diarization      | Yes          | No   | `diarization: true`              |
| Translation              | Yes          | Yes  | `translation: true`              |
| Sentiment analysis       | Yes          | Yes  | `sentiment_analysis: true`       |
| Named entity recognition | Yes          | Yes  | `named_entity_recognition: true` |
| Subtitles (SRT/VTT)      | Yes          | No   | `subtitles: true`                |
| Custom vocabulary        | Yes          | Yes  | `custom_vocabulary: true`        |
| PII redaction            | Yes          | No   | `pii_redaction: true`            |
| Chapterization           | Yes          | Yes  | `chapterization: true`           |
| Summarization            | Yes          | Yes  | `summarization: true`            |
| Audio-to-LLM             | Yes          | No   | `audio_to_llm: true`             |

## Decision guidance

### When to use Pre-recorded vs Live

| Scenario                         | Use Pre-recorded  | Use Live |
| -------------------------------- | ----------------- | -------- |
| Batch processing files           | Yes               |          |
| Real-time transcription needed   |                   | Yes      |
| Polling acceptable               | Yes               |          |
| Streaming audio input            |                   | Yes      |
| Diarization required             | Yes               |          |
| Need partial results immediately |                   | Yes      |
| Processing 3+ hours of audio     | Split into chunks | Yes      |

### When to enable code switching

| Condition                                  | Enable | Disable              |
| ------------------------------------------ | ------ | -------------------- |
| Speakers switch languages mid-conversation | Yes    |                      |
| Single language audio                      |        | Yes                  |
| Need per-utterance language detection      | Yes    |                      |
| Exactly one language specified             |        | Yes (ignored anyway) |
| Bilingual/multilingual content             | Yes    |                      |

Critical: Never enable code switching with empty `languages` list — it causes misdetections across 100+ languages. Always provide 3-5 expected languages.

### When to use custom vocabulary

| Use case                                     | Approach                                |
| -------------------------------------------- | --------------------------------------- |
| Domain-specific terms (medical, legal, tech) | Enable with pronunciations              |
| Brand names or proper nouns                  | Simple string list or intensity 0.4-0.6 |
| Accented or foreign pronunciations           | Add multiple pronunciations per term    |
| Multilingual audio                           | Set language per vocabulary entry       |
| Generic content                              | Disable (adds latency)                  |

## Workflow

### Pre-recorded transcription (SDK)

1. Initialize client: `new GladiaClient({ apiKey: "YOUR_KEY" })`
2. Transcribe in one call: Pass file path or URL to `transcribe()` method with options
3. Configure features: Add `diarization`, `translation`, `sentiment_analysis`, `custom_vocabulary_config` as needed
4. Handle result: Receive structured JSON with transcription, speakers, timestamps, and intelligence data

### Pre-recorded transcription (API)

1. Upload audio: POST to `/v2/upload` with multipart form data, get `audio_url`
2. Create job: POST to `/v2/pre-recorded` with `audio_url` and config
3. Poll or wait: GET `/v2/pre-recorded/:id` until `status: "done"` (or use webhooks/callbacks)
4. Extract results: Parse `transcription`, `diarization`, `translation`, `sentiment_analysis` from response

### Live transcription (SDK)

1. Initialize session: Call `startSession(config)` with encoding, sample_rate, bit_depth, channels
2. Connect: SDK handles WebSocket connection automatically
3. Send audio: Call `sendAudio(chunk)` for each audio buffer
4. Listen for messages: Register handlers for `message`, `started`, `ended`, `error` events
5. Stop recording: Call `stopRecording()` when done; SDK closes connection and triggers post-processing
6. Fetch final results: GET `/v2/live/:id` for complete transcript with all intelligence features

### Live transcription (API)

1. Init session: POST to `/v2/live` with audio config, get `id` and WebSocket `url`
2. Connect WebSocket: Use native WebSocket or library; connect to returned URL
3. Send audio: Send binary buffer or base64-encoded JSON chunks
4. Read messages: Parse incoming JSON messages; check `type` and `is_final` flag
5. Stop: Send `{ type: "stop_recording" }` or close with code 1000
6. Fetch results: GET `/v2/live/:id` after post-processing completes

## Common gotchas

- Code switching without language list: Enabling `code_switching: true` with empty `languages` array causes 100+ language evaluation and frequent misdetections. Always provide 3-5 expected languages.
- Custom vocabulary intensity too high: Values > 0.6 cause false positives where unrelated words get replaced. Keep 0.4-0.6; use pronunciations instead of raising intensity.
- Exceeding audio limits silently: Pre-recorded files > 135 minutes fail without clear error. Split long files into ~60-minute chunks before uploading.
- Multi-channel billing: Sending 2-channel audio is billed as 2x duration. Merge channels only if you need per-channel speaker identification.
- WebSocket reconnection: If connection drops, reconnect to the same URL to resume the session. Don't create a new session.
- Polling without backoff: Polling `/v2/pre-recorded/:id` too frequently wastes requests. Use webhooks or callbacks instead, or implement exponential backoff.
- Forgetting to stop recording: Leaving WebSocket open without sending `stop_recording` leaves the session hanging. Always explicitly stop or close with code 1000.
- Partial transcripts disabled by default: Real-time results come only as final transcripts. Enable `messages_config.receive_partial_transcripts: true` for low-latency partial results.
- Diarization not available in live: Speaker diarization only works for pre-recorded. For live multi-speaker, use multi-channel audio and track by channel.
- PII redaction pre-recorded only: PII redaction (`pii_redaction: true`) only works for pre-recorded transcription, not live.

## Verification checklist

Before submitting transcription work:

- [ ] API key is valid and passed in `x-gladia-key` header
- [ ] Audio file is under 1000 MB and pre-recorded audio under 135 minutes (or split into chunks)
- [ ] Audio format is supported (check MIME type in docs)
- [ ] If using code switching, `languages` list is provided with 3-5 expected languages
- [ ] If using custom vocabulary, intensity is 0.4-0.6 and pronunciations are provided for ambiguous terms
- [ ] For live transcription, WebSocket connection is properly closed with `stop_recording` or code 1000
- [ ] Callbacks or webhooks are configured if polling is not acceptable
- [ ] Multi-channel audio is intentional and billing impact is understood
- [ ] Results include expected fields: `transcription.utterances`, `speaker` (if diarization), `language` (if code switching)
- [ ] Error responses are checked: `status`, `error_message`, and `error_code` fields

## Resources

- Full navigation: https://docs.gladia.io/llms.txt — comprehensive page-by-page reference for all endpoints and features
- API Reference: https://docs.gladia.io/api-reference — authentication, endpoint schemas, callback definitions
- Pre-recorded Quickstart: https://docs.gladia.io/chapters/pre-recorded-stt/quickstart — SDK and API examples for async transcription
- Live Quickstart: https://docs.gladia.io/chapters/live-stt/quickstart — SDK and API examples for real-time streaming

---

> This file is auto-synced from https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
> Do not edit manually — changes will be overwritten by CI.
> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
