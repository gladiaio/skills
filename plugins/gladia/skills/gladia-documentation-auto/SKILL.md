---
name: gladia-documentation-auto
description: Comprehensive Gladia speech-to-text reference auto-synced from docs.gladia.io. Use as a general-purpose fallback when other specialized skills don't match, or when the user needs a broad overview of Gladia capabilities, endpoints, decision guidance, or workflows. Always prefer the official SDK; fall back to raw REST/WebSocket only when SDK cannot satisfy the requirement.
license: MIT
metadata:
  source: https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
  digest: sha256:e0d6b665b502656deb4add69bdd6f534c0c997e13acdd35dcefb24296e23f307
  synced: "2026-09-06"
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
description: Use when building speech-to-text transcription features, processing audio/video files, streaming live audio, extracting insights from audio (diarization, translation, sentiment), or integrating transcription into voice agents, meeting recorders, or call center applications.
metadata:
    mintlify-proj: gladia
    version: "1.0"
---

# Gladia Skill

## Product summary

Gladia is a speech-to-text API that transcribes audio and video in real-time (live streaming) or asynchronously (pre-recorded files). It supports 100+ languages, offers two models (Solaria-1 for broad coverage, Solaria-3 for European accuracy), and includes audio intelligence features like speaker diarization, translation, sentiment analysis, and summarization. Use the official SDKs (Python/JavaScript) for simple integration, or call REST endpoints directly. Authenticate with `x-gladia-key` header. Primary docs: https://docs.gladia.io

**Key files and commands:**
- SDK: `npm install @gladiaio/sdk` (JavaScript) or `pip install gladiaio-sdk` (Python)
- CLI: `gladia transcribe audio.mp3` (requires API key in env or `~/.gladia`)
- API endpoints: `POST /v2/pre-recorded` (async), `POST /v2/live` (streaming), `GET /v2/pre-recorded/:id` (fetch results)
- Authentication: Pass `x-gladia-key: YOUR_API_KEY` in request headers

## When to use

Reach for Gladia when:
- Building transcription into an app (voice agents, meeting recorders, call centers, podcasts)
- Processing pre-recorded audio/video files asynchronously (upload once, poll or webhook for results)
- Streaming live audio in real-time (WebSocket-based, with partial transcripts available)
- Extracting structured data from audio (speaker identification, entity names, sentiment, summaries)
- Handling multilingual content or code-switching (mixed languages in one audio)
- Generating subtitles (SRT/VTT) or translating transcripts to multiple languages
- Testing transcription quality before full integration (use the Playground at https://app.gladia.io/playground)

## Quick reference

### Model selection

| Model | Use case | Languages | Code switching | Live? | Pre-recorded? |
|-------|----------|-----------|-----------------|-------|---------------|
| **Solaria-3** | Highest accuracy on real-world European audio | EN, FR, DE, ES, IT only | No | ❌ | ✅ |
| **Solaria-1** | Broad coverage, any domain | 100+ languages | Yes | ✅ | ✅ |

### Pre-recorded workflow (SDK)

```javascript
// One-call transcription
const result = await gladia.preRecorded().transcribe("audio.mp3");
console.log(result.result.transcription.full_transcript);

// With options
const result = await gladia.preRecorded().transcribe("audio.mp3", {
  model: "solaria-3",
  language_config: { languages: ["en"] },
  diarization: true,
  summarization: true,
});
```

### Live workflow (SDK)

```javascript
const session = gladia.liveV2().startSession({
  encoding: "wav/pcm",
  sample_rate: 16000,
  bit_depth: 16,
  channels: 1,
});

session.on("message", (message) => {
  if (message.type === "transcript" && message.data.is_final) {
    console.log(message.data.utterance.text);
  }
});

// session.sendAudio(chunk)
// session.stopRecording()
```

### Audio intelligence features

| Feature | Pre-recorded | Live | Purpose |
|---------|-------------|------|---------|
| **Diarization** | ✅ | ✅ | Identify speakers (speaker 0, 1, 2...) |
| **Translation** | ✅ | ✅ | Translate to 100+ target languages |
| **Sentiment analysis** | ✅ | ✅ | Extract emotion and tone per utterance |
| **Named entity recognition** | ✅ | ✅ | Detect people, organizations, dates |
| **Summarization** | ✅ | ✅ | Generate general, bullet-point, or concise summaries |
| **Subtitles** | ✅ | ❌ | Generate SRT/VTT files with timing |
| **Custom vocabulary** | ✅ | ✅ | Boost accuracy for domain-specific terms |
| **PII redaction** | ✅ | ❌ | Anonymize names, emails, IDs |

### Result retrieval methods

| Method | Best for | Code |
|--------|----------|------|
| **Polling** | Small batches, immediate feedback | `gladia.preRecorded().poll(jobId)` |
| **Webhooks** | Production, async workflows | Configure at https://app.gladia.io/webhooks |
| **Callbacks** | Per-request notifications | Pass `callback_config.url` in init request |

## Decision guidance

### When to use pre-recorded vs. live

| Scenario | Use pre-recorded | Use live |
|----------|-----------------|----------|
| File upload (MP3, WAV, etc.) | ✅ | ❌ |
| Streaming microphone or WebRTC | ❌ | ✅ |
| Need results immediately | ❌ | ✅ (partial transcripts) |
| Batch processing | ✅ | ❌ |
| Voice agent (low latency) | ❌ | ✅ |
| Meeting recording | ✅ (post-meeting) | ✅ (live captions) |

### When to use Solaria-3 vs. Solaria-1

| Condition | Choose Solaria-3 | Choose Solaria-1 |
|-----------|-----------------|-----------------|
| Audio is pre-recorded | ✅ | ✅ |
| Audio is live/streaming | ❌ | ✅ |
| Language is EN, FR, DE, ES, or IT | ✅ | ✅ |
| Language outside those five | ❌ | ✅ |
| Need code switching | ❌ | ✅ |
| Production real-world audio | ✅ | ✅ |
| Clean/formal speech | ✅ | ✅ |

### When to use diarization vs. multi-channel

| Approach | Use when | Limitation |
|----------|----------|-----------|
| **Diarization** | Single audio stream, multiple speakers | Slower, less accurate with 3+ speakers |
| **Multi-channel** | Separate audio feeds per speaker | Requires pre-split channels, billed per channel |

## Workflow

### Pre-recorded transcription (typical task)

1. **Get API key** — Copy from https://app.gladia.io/apikeys
2. **Choose model** — Solaria-3 (if EN/FR/DE/ES/IT pre-recorded) or Solaria-1 (other languages, live)
3. **Prepare audio** — Local file or remote URL (MP3, WAV, M4A, etc.)
4. **Build request** — Use SDK `transcribe()` for one-call, or manual steps for control:
   - Upload file (if local): `gladia.preRecorded().uploadFile(path)` → get `audio_url`
   - Create job: `gladia.preRecorded().create({ audio_url, model, language_config, ... })`
   - Retrieve result: `gladia.preRecorded().poll(jobId)` or wait for webhook
5. **Extract data** — Access `result.result.transcription.full_transcript` and audio intelligence results
6. **Verify** — Check `status` field (queued → processing → done/error), inspect `error` if failed

### Live transcription (typical task)

1. **Initialize session** — Call `POST /v2/live` with audio format (encoding, sample_rate, bit_depth, channels)
2. **Get WebSocket URL** — Response includes `url` and `id` for reconnection
3. **Connect** — Open WebSocket to the returned URL (SDK handles this)
4. **Send audio chunks** — Stream audio as binary or base64-encoded JSON
5. **Listen for messages** — Subscribe to `transcript` (check `is_final` flag), `speech_start`, `speech_end`, errors
6. **Stop recording** — Send `stop_recording` message or close WebSocket with code 1000
7. **Retrieve final results** — Call `GET /v2/live/:id` or wait for callback

## Common gotchas

- **Solaria-3 language constraint** — Pass exactly one language in `language_config.languages` (e.g., `["fr"]`). Do not pass multiple languages or enable code switching; the model will fail.
- **Audio format mismatch** — Specify `encoding`, `sample_rate`, `bit_depth`, and `channels` correctly in live init. Mismatches cause silent failures or garbled output.
- **Live session timeout** — A single live session cannot exceed 3 hours. For longer events, start a new session before the limit.
- **Polling without backoff** — Polling too aggressively wastes API quota. Use exponential backoff (start at 1s, cap at 10s) or webhooks instead.
- **Forgetting to enable features** — Audio intelligence features (diarization, translation, etc.) are opt-in. Set `diarization: true` or `translation: true` in the request; they don't run by default.
- **Code switching with empty language list** — Enabling `code_switching: true` without specifying `languages` forces detection across 100+ languages, causing frequent misdetections. Always provide a constrained list (e.g., `["en", "fr"]`).
- **Callback URL not HTTPS** — Webhooks and callbacks require HTTPS endpoints. HTTP will fail silently.
- **Concurrency limits** — Free tier: 3 pre-recorded, 1 live. Paid: 25 pre-recorded, 30 live. Queued jobs beyond concurrency limits wait; they don't fail.
- **Deprecated V1 endpoints** — `/audio/text/audio-transcription` and `/transcription` are deprecated. Use `/v2/pre-recorded` and `/v2/live` instead.
- **Multi-channel billing** — Transcribing a 2-channel stream is billed as 2× the audio duration. Plan accordingly.

## Verification checklist

Before submitting transcription work:

- [ ] API key is valid and passed in `x-gladia-key` header (not in URL or body)
- [ ] Model choice matches use case (Solaria-3 for pre-recorded EU audio, Solaria-1 for live or broad coverage)
- [ ] Audio format parameters (encoding, sample_rate, bit_depth, channels) match actual audio
- [ ] Language configuration is correct: single language for Solaria-3, list for Solaria-1, no code switching with Solaria-3
- [ ] Audio intelligence features are explicitly enabled if needed (diarization, translation, etc.)
- [ ] Result retrieval method is set (polling with backoff, webhook, or callback)
- [ ] For live: WebSocket URL is generated on backend, not exposed to client
- [ ] For pre-recorded: audio_url is valid (remote URL or uploaded file)
- [ ] Error handling covers 401 (auth), 422 (invalid params), 429 (rate limit), and network timeouts
- [ ] Concurrency limits are respected (queue jobs if needed)

## Resources

- **Full page navigation** — https://docs.gladia.io/llms.txt
- **API Reference** — https://docs.gladia.io/api-reference
- **Pre-recorded quickstart** — https://docs.gladia.io/chapters/pre-recorded-stt/quickstart
- **Live quickstart** — https://docs.gladia.io/chapters/live-stt/quickstart
- **Audio intelligence features** — https://docs.gladia.io/chapters/audio-intelligence
- **SDK samples** — https://github.com/gladiaio/gladia-samples

---

> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
---

> This file is auto-synced from https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
> Do not edit manually — changes will be overwritten by CI.
> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
