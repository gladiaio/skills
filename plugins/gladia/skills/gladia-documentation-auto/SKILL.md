---
name: gladia-documentation-auto
description: Comprehensive Gladia speech-to-text reference auto-synced from docs.gladia.io. Use as a general-purpose fallback when other specialized skills don't match, or when the user needs a broad overview of Gladia capabilities, endpoints, decision guidance, or workflows. Always prefer the official SDK; fall back to raw REST/WebSocket only when SDK cannot satisfy the requirement.
license: MIT
metadata:
  source: https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
  digest: sha256:de92064bf3c62232f90967322ea13d2cf8989aa71a79fcec6caff5966da68a15
  synced: "2026-09-15"
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
name: Gladia
description: Use when building speech-to-text transcription features, processing audio/video files, streaming live audio, extracting insights from audio (diarization, translation, sentiment), or integrating transcription into voice AI applications. Agents should reach for this skill when users need to transcribe audio, build real-time voice features, or extract structured data from speech.
metadata:
    mintlify-proj: gladia
    version: "1.0"
---

# Gladia Skill

## Product summary

Gladia is a speech-to-text (STT) API that transcribes audio and video files (pre-recorded) and live audio streams in real time. It supports 100+ languages, offers two models (Solaria-1 for live/multilingual, Solaria-3 for pre-recorded European audio), and includes audio intelligence features like diarization, translation, sentiment analysis, and entity recognition. Use the official SDKs (JavaScript/TypeScript, Python) for simplest integration, or call REST/WebSocket APIs directly. Authentication is via `x-gladia-key` header. Primary docs: https://docs.gladia.io

**Key files and commands:**
- API key: Get from https://app.gladia.io/apikeys
- SDKs: `npm install @gladiaio/sdk` (JS) or `pip install gladiaio-sdk` (Python)
- CLI: `gladia transcribe audio.mp3` (terminal tool)
- Endpoints: `POST /v2/pre-recorded` (async), `POST /v2/live` (real-time WebSocket)
- Authentication header: `x-gladia-key: YOUR_API_KEY`

## When to use

Reach for this skill when:
- User asks to transcribe audio files, videos, or live audio streams
- Building voice AI agents, meeting recorders, or call transcription features
- Need to extract speaker identity (diarization), translate transcripts, or analyze sentiment
- Integrating with platforms like Pipecat, LiveKit, Vapi, Twilio, or no-code tools (Zapier, Make, n8n)
- User needs to choose between real-time vs batch transcription, or between language models
- Debugging transcription quality issues or configuring audio parameters (sample rate, encoding, endpointing)
- User needs to handle multi-language audio or code-switching scenarios

## Quick reference

### Model selection

| Model | Mode | Languages | Code switching | Best for |
|-------|------|-----------|-----------------|----------|
| **Solaria-3** | Pre-recorded only | EN, FR, DE, ES, IT | No | European real-world/business audio, highest accuracy |
| **Solaria-1** | Pre-recorded + live | 100+ | Yes | Global coverage, live streaming, multilingual |

### Audio parameters (live transcription)

| Parameter | Default | Range | Effect |
|-----------|---------|-------|--------|
| `encoding` | `wav/pcm` | `wav/pcm`, `wav/alaw`, `wav/ulaw` | Audio format |
| `sample_rate` | 16000 | 8000, 16000, 32000, 44100, 48000 | Hz |
| `bit_depth` | 16 | 8, 16, 24, 32 | Bits per sample |
| `channels` | 1 | 1+ | Number of audio channels |
| `endpointing` | 0.05 | 0.01–10 | Silence duration (seconds) to close utterance |
| `maximum_duration_without_endpointing` | 5 | 5–60 | Max utterance duration (seconds) |

### Audio Intelligence features (add-ons)

| Feature | Pre-recorded | Live | Use case |
|---------|--------------|------|----------|
| Diarization | ✓ | ✓ | Identify speakers (Speaker 0, 1, 2…) |
| Translation | ✓ | ✓ | Translate to 100+ target languages |
| Sentiment analysis | ✓ | ✓ | Extract emotion and tone per utterance |
| Named entity recognition | ✓ | ✓ | Detect people, organizations, dates |
| PII redaction | ✓ | ✗ | Anonymize names, emails, IDs |
| Summarization | ✓ | ✓ | Generate summaries or bullet points |
| Subtitles | ✓ | ✗ | Generate SRT/VTT files |
| Custom vocabulary | ✓ | ✓ | Boost accuracy for domain terms |
| Custom spelling | ✓ | ✓ | Force specific spellings |

### Limits and billing

| Limit | Value | Notes |
|-------|-------|-------|
| Pre-recorded max duration | 135 min (standard), 4h15 (enterprise) | Longer files rejected |
| Live session max duration | 3 hours | Restart session after 3h |
| File size | 1000 MB max | Larger files rejected |
| Live concurrency | 30 sessions | Contact sales for increase |
| Pre-recorded concurrency | 25 parallel + 300 queued | Paid plans only |
| Billing | Per audio duration × channels | Multi-channel streams billed per channel |
| Data retention | Audio/transcripts: 3 weeks; metadata: 1 year | Default for paid accounts |

### SDK quick patterns

**Pre-recorded (one call):**
```javascript
const result = await gladia.preRecorded().transcribe("audio.mp3");
```

**Pre-recorded (with options):**
```javascript
const result = await gladia.preRecorded().transcribe("audio.mp3", {
  model: "solaria-3",
  language_config: { languages: ["en"] },
  diarization: true,
});
```

**Live (start session):**
```javascript
const session = gladia.liveV2().startSession({
  encoding: "wav/pcm",
  sample_rate: 16000,
  bit_depth: 16,
  channels: 1,
});
session.on("message", (msg) => {
  if (msg.type === "transcript" && msg.data.is_final) {
    console.log(msg.data.utterance.text);
  }
});
```

## Decision guidance

### When to use pre-recorded vs live

| Scenario | Use pre-recorded | Use live |
|----------|------------------|----------|
| User uploads a file | ✓ | ✗ |
| Real-time voice call / meeting | ✗ | ✓ |
| Batch processing (podcasts, archives) | ✓ | ✗ |
| Voice agent / interactive app | ✗ | ✓ |
| Need Solaria-3 accuracy | ✓ | ✗ (not available) |
| Need 100+ languages | ✓ | ✓ (Solaria-1 only) |

### When to use Solaria-3 vs Solaria-1

| Condition | Use Solaria-3 | Use Solaria-1 |
|-----------|---------------|---------------|
| Pre-recorded audio | ✓ | ✓ |
| Live/real-time | ✗ | ✓ |
| European business/meeting audio | ✓ | ✗ |
| Language is EN, FR, DE, ES, IT | ✓ | ✓ |
| Language outside those five | ✗ | ✓ |
| Code switching (mixed languages) | ✗ | ✓ |
| Clean, formal speech | ✗ | ✓ |
| Noisy, conversational audio | ✓ | ✗ |

### When to use SDK vs API vs CLI

| Tool | When to use |
|------|------------|
| **SDK** (JS/Python) | Building production apps; automatic retries, error handling, upload abstraction |
| **API** (REST/WebSocket) | Custom integrations, non-SDK languages, fine-grained control |
| **CLI** | Terminal scripts, CI/CD pipelines, one-off transcriptions, testing |

### Result retrieval strategy

| Strategy | When to use |
|----------|------------|
| **Polling** (`poll()` in SDK) | Simple scripts, small batches, acceptable latency |
| **Webhooks** | Production systems; configure at https://app.gladia.io/webhooks |
| **Callbacks** | Per-job notifications; set `callback` and `callback_config` in request |

## Workflow

### Pre-recorded transcription (SDK)

1. **Get API key** from https://app.gladia.io/apikeys and set `GLADIA_API_KEY` env var
2. **Install SDK**: `npm install @gladiaio/sdk` or `pip install gladiaio-sdk`
3. **Choose model**: Solaria-3 (pre-recorded, 5 languages, highest accuracy) or Solaria-1 (default, 100+ languages)
4. **Transcribe in one call**:
   - Pass local file path, remote URL, or binary data
   - Specify `model`, `language_config.languages`, and audio intelligence options
   - SDK handles upload, job creation, polling, and result retrieval
5. **Access results**: `result.result.transcription.full_transcript` for text; check `result.result.diarization`, `result.result.translation`, etc. for add-ons
6. **Verify**: Check `result.status === "done"` and `result.error` is null

### Live transcription (SDK)

1. **Get API key** and install SDK
2. **Start session** with audio parameters (`encoding`, `sample_rate`, `bit_depth`, `channels`)
3. **Register message handlers**: `session.on("message", ...)`, `session.on("error", ...)`, `session.on("ended", ...)`
4. **Send audio chunks** via `session.sendAudio(chunk)` as they arrive
5. **Read transcripts**: Listen for `message.type === "transcript"` and check `message.data.is_final` (true = final, false = partial)
6. **Stop recording**: Call `session.stopRecording()` when done; SDK closes WebSocket and triggers post-processing
7. **Get final results**: Retrieve via `GET /v2/live/{id}` or wait for callback

### Pre-recorded transcription (API)

1. **Upload audio**: `POST /v2/upload` with multipart form data; get `audio_url` from response
2. **Create job**: `POST /v2/pre-recorded` with `audio_url`, `model`, `language_config`, and options
3. **Poll for result**: `GET /v2/pre-recorded/{id}` until `status === "done"`
4. **Handle errors**: Check `error_code` and `error` fields; retry if transient

### Live transcription (API)

1. **Init session**: `POST /v2/live` with audio parameters; get `id` and WebSocket `url`
2. **Connect WebSocket**: Open connection to returned `url`
3. **Send audio**: Send binary audio chunks or JSON with base64-encoded chunks
4. **Read messages**: Parse JSON messages; filter by `type` and `is_final`
5. **Stop**: Send `{"type": "stop_recording"}` or close with code 1000
6. **Retrieve results**: `GET /v2/live/{id}` after session ends

## Common gotchas

- **Solaria-3 language constraint**: With Solaria-3, pass exactly one language in `language_config.languages` (e.g., `["fr"]`). Do not pass multiple languages or enable code switching—the model will reject it.
- **Live model limitation**: Live transcription only supports Solaria-1. Solaria-3 is pre-recorded only.
- **Endpointing tuning**: Default `endpointing: 0.05` closes utterances fast; increase to 0.5–1.0 if sentences are being split mid-speech. Increase `maximum_duration_without_endpointing` (default 5s) for long monologues.
- **Multi-channel billing**: Transcribing a 2-channel audio stream is billed as 2× the duration. Merge channels on your side if you only need one speaker.
- **Session timeout**: Live sessions terminate after 3 hours. Plan for session restart in long-running applications.
- **Partial transcripts accuracy**: Partial transcripts use a faster model and are less accurate than finals. Disable `receive_partial_transcripts` if you only need final results.
- **Code switching with Solaria-3**: Not supported. Use Solaria-1 for mixed-language audio.
- **File size limit**: Pre-recorded files must be ≤1000 MB. Larger files are rejected before processing.
- **Audio format mismatch**: Ensure `encoding`, `sample_rate`, `bit_depth`, and `channels` match your actual audio. Mismatches cause garbled transcripts.
- **Callback URL must be public**: Webhooks and callbacks POST to your URL; it must be reachable from Gladia's servers.
- **API key in client code**: Generate WebSocket URLs on your backend and pass secure URLs to clients. Never expose API keys in frontend code.

## Verification checklist

Before submitting work:

- [ ] API key is set and valid (test with a simple request)
- [ ] Model choice matches use case (Solaria-3 for pre-recorded European audio; Solaria-1 for live or 100+ languages)
- [ ] Language configuration is correct (Solaria-3: one language; Solaria-1: one or multiple with code switching if needed)
- [ ] Audio parameters (`encoding`, `sample_rate`, `bit_depth`, `channels`) match the actual audio
- [ ] For pre-recorded: audio file is ≤1000 MB and ≤135 min (or ≤4h15 for enterprise)
- [ ] For live: session will not exceed 3 hours; plan for restart if needed
- [ ] Audio Intelligence features are enabled only if needed (diarization, translation, sentiment, etc.)
- [ ] Result retrieval strategy is chosen (polling, webhooks, or callbacks)
- [ ] Error handling is in place (check `status`, `error_code`, and retry logic)
- [ ] For live: WebSocket URL is generated on backend and not exposed in client code
- [ ] Callback URL (if used) is public and reachable
- [ ] Test with a small audio sample before processing large batches

## Resources

**Comprehensive navigation**: https://docs.gladia.io/llms.txt

**Critical documentation pages:**
1. [Pre-recorded STT quickstart](https://docs.gladia.io/chapters/pre-recorded-stt/quickstart) — Upload, create jobs, retrieve results
2. [Live STT quickstart](https://docs.gladia.io/chapters/live-stt/quickstart) — WebSocket streaming, partial transcripts, session lifecycle
3. [Models comparison](https://docs.gladia.io/chapters/introduction/models) — Solaria-3 vs Solaria-1 decision guide
4. [Audio Intelligence](https://docs.gladia.io/chapters/pre-recorded-stt/audio-intelligence) — Diarization, translation, sentiment, entities
5. [API Reference](https://docs.gladia.io/api-reference) — Full endpoint documentation
6. [Limits & Specifications](https://docs.gladia.io/chapters/limits-and-specifications/concurrency) — Concurrency, duration, file size limits

---

> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
---

> This file is auto-synced from https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
> Do not edit manually — changes will be overwritten by CI.
> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
