---
name: gladia-documentation-auto
description: Comprehensive Gladia speech-to-text reference auto-synced from docs.gladia.io. Use as a general-purpose fallback when other specialized skills don't match, or when the user needs a broad overview of Gladia capabilities, endpoints, decision guidance, or workflows. Always prefer the official SDK; fall back to raw REST/WebSocket only when SDK cannot satisfy the requirement.
license: MIT
metadata:
  source: https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
  digest: sha256:c555f4874bbc15a36a96c434a0d3852c3a94b95868381ae35fe823e80876a71e
  synced: "2026-07-09"
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
description: Use when building speech-to-text transcription features, processing audio/video files, implementing real-time transcription, extracting insights from audio (speaker identification, translation, sentiment), or integrating audio intelligence into applications. Agents should reach for this skill when users request transcription, audio analysis, or speech processing capabilities.
metadata:
    mintlify-proj: gladia
    version: "1.0"
---

# Gladia Skill

## Product summary

Gladia is a speech-to-text (STT) API that transcribes audio and video files in both pre-recorded (asynchronous) and live (real-time) modes. It provides transcription plus audio intelligence features: speaker diarization, translation, sentiment analysis, PII redaction, summarization, and more. Agents use Gladia to build transcription workflows, extract data from audio, and integrate speech processing into applications.

**Key entry points:**
- **Pre-recorded API**: `POST /v2/pre-recorded` (async transcription)
- **Live API**: `POST /v2/live` (WebSocket-based real-time)
- **Upload endpoint**: `POST /v2/upload` (for local files)
- **Authentication**: `x-gladia-key` header with API key
- **SDKs**: JavaScript/TypeScript (`@gladiaio/sdk`) and Python (`gladiaio-sdk`)
- **Primary docs**: https://docs.gladia.io

## When to use

Reach for this skill when:
- A user requests transcription of audio or video files (meetings, calls, podcasts, interviews)
- Building real-time transcription for live events or streaming audio
- Extracting speaker information, translations, or sentiment from audio
- Implementing PII redaction for compliance (GDPR, HIPAA, CCPA)
- Processing multi-language content with language detection or code switching
- Generating subtitles, summaries, or meeting notes from audio
- Integrating with platforms like Twilio, LiveKit, Vapi, or Pipecat
- Optimizing transcription quality with custom vocabulary or domain-specific terms

## Quick reference

### Models

| Model | Use case | Supports |
|-------|----------|----------|
| `solaria-3` | Pre-recorded (best quality, slower) | All features except live |
| `solaria-1` | Live/real-time only | Streaming, partial transcripts |

### Core parameters (pre-recorded)

```json
{
  "audio_url": "https://...",  // or upload first
  "model": "solaria-3",
  "language_config": {
    "languages": ["en"],  // ISO 639-1 codes
    "code_switching": false
  },
  "diarization": true,
  "diarization_config": {
    "number_of_speakers": 2,  // or min/max
    "min_speakers": 1,
    "max_speakers": 5
  }
}
```

### Core parameters (live)

```json
{
  "model": "solaria-1",
  "encoding": "wav/pcm",
  "sample_rate": 16000,
  "bit_depth": 16,
  "channels": 1,
  "language_config": {
    "languages": ["en"],
    "code_switching": false
  }
}
```

### Audio intelligence features

| Feature | Pre-recorded | Live | Use for |
|---------|--------------|------|---------|
| Speaker diarization | ✓ | ✓ | Identify who said what |
| Translation | ✓ | ✓ | Multi-language output |
| Subtitles | ✓ | ✗ | SRT/VTT files |
| Custom vocabulary | ✓ | ✓ | Domain-specific terms |
| Custom spelling | ✓ | ✓ | Normalize misspellings |
| PII redaction | ✓ | ✗ | GDPR/HIPAA compliance |
| Sentiment analysis | ✓ | ✓ | Emotion detection |
| Named entity recognition | ✓ | ✓ | Extract people, places, dates |
| Summarization | ✓ | ✗ | Auto-generate summaries |
| Chapterization | ✓ | ✗ | Break into sections |

### Supported formats

**Audio**: aac, ac3, eac3, flac, m4a, mp2, mp3, ogg, opus, wav  
**Video**: 3g2, 3gp, avi, flv, m4v, matroska, mov, mp4, wmv  
**Online**: TikTok, Instagram, Facebook, Vimeo, LinkedIn, Dailymotion, Sharechat, Likee

### Limits

| Limit | Free | Paid | Enterprise |
|-------|------|------|------------|
| Monthly usage | 10 hours | Unlimited | Unlimited |
| Pre-recorded concurrency | 3 | 25 | On demand |
| Live concurrency | 1 | 30 | On demand |
| Max file duration | 135 min | 135 min | 4h 15m |
| Max file size | 1000 MB | 1000 MB | 1000 MB |
| Max live session | 3 hours | 3 hours | 3 hours |

## Decision guidance

### When to use pre-recorded vs. live

| Scenario | Use |
|----------|-----|
| User uploads a file, wants transcript later | Pre-recorded (`/v2/pre-recorded`) |
| Real-time transcription of streaming audio | Live (`/v2/live` WebSocket) |
| Meeting recording to process after | Pre-recorded |
| Live event, conference, or call transcription | Live |
| Need subtitles or summarization | Pre-recorded (live doesn't support these) |

### When to use SDK vs. raw API

| Scenario | Use |
|----------|-----|
| Simple end-to-end transcription | SDK (handles upload, polling, retries) |
| Fine-grained control over each step | Raw API (upload, create job, poll separately) |
| Integrating into existing HTTP client | Raw API |
| Building with Node.js or Python | SDK (better DX) |

### Custom vocabulary vs. custom spelling

| Scenario | Use |
|----------|-----|
| Model outputs garbled/phonetically wrong text | Custom vocabulary (phoneme-based matching) |
| Model outputs recognizable but misspelled text | Custom spelling (literal text matching) |
| Brand names or proper nouns | Custom vocabulary with pronunciations |
| Normalizing variant spellings | Custom spelling |

### Diarization vs. multi-channel audio

| Scenario | Use |
|----------|-----|
| Single audio file, need to identify speakers | Diarization (`diarization: true`) |
| Multiple separate audio tracks (e.g., Zoom participants) | Multi-channel (merge into one stream, set `channels: N`) |
| Call recording with agent + customer | Diarization (simpler, set `number_of_speakers: 2`) |

### Language detection vs. explicit language

| Scenario | Use |
|----------|-----|
| Language is known in advance | Set `language_config.languages: ["en"]` (faster) |
| Language is unknown | Omit `languages` (auto-detect) |
| Multiple languages in one file | Set `languages: ["en", "fr"]` + `code_switching: true` |

## Workflow

### Pre-recorded transcription (SDK)

1. **Initialize client**: `new GladiaClient({ apiKey: "YOUR_KEY" })`
2. **Call transcribe()**: Pass audio URL or local path + options
3. **Wait for result**: SDK polls until job completes
4. **Extract data**: Access `transcription`, `translation`, `diarization`, etc. from result

### Pre-recorded transcription (API)

1. **Upload audio** (if local): `POST /v2/upload` → get `audio_url`
2. **Create job**: `POST /v2/pre-recorded` with `audio_url` + config
3. **Poll for result**: `GET /v2/pre-recorded/:id` until `status: "done"`
4. **Parse response**: Extract transcription, audio intelligence results

### Live transcription (SDK)

1. **Initialize session**: `gladiaClient.liveV2().startSession(config)`
2. **Register handlers**: Listen for `message`, `error`, `started`, `ended` events
3. **Send audio chunks**: `liveSession.sendAudio(chunk)` as data arrives
4. **Handle callbacks**: Process `transcript`, `translation`, `sentiment` messages in real-time
5. **Stop recording**: `liveSession.stopRecording()` when done
6. **Fetch final result** (optional): `GET /v2/live/:id` for complete output

### Live transcription (API)

1. **Initiate session**: `POST /v2/live` with encoding, sample rate, channels → get WebSocket URL
2. **Connect WebSocket**: Open connection to returned URL
3. **Send audio**: Send binary or base64-encoded audio chunks
4. **Read messages**: Parse JSON messages (transcript, translation, etc.)
5. **Stop recording**: Send `{ "type": "stop_recording" }` or close with code 1000
6. **Fetch final result** (optional): `GET /v2/live/:id`

### Adding audio intelligence

1. **Identify features needed**: diarization, translation, sentiment, PII redaction, etc.
2. **Add to request body**: Enable feature flag + config object
3. **For pre-recorded**: Add at top level (e.g., `"diarization": true`, `"translation": true`)
4. **For live**: Nest under `realtime_processing` (e.g., `"realtime_processing": { "translation": true }`)
5. **Parse results**: Features appear in response under their own keys

## Common gotchas

- **solaria-3 with multiple languages**: Set exactly ONE language in `language_config.languages` (e.g., `["fr"]`). Do not pass multiple languages or enable code switching with solaria-3.
- **Live sessions limited to 3 hours**: After 3 hours, session terminates. Start a new session before hitting the limit.
- **Polling without backoff**: Don't hammer the API. Implement exponential backoff or use webhooks/callbacks instead.
- **Custom vocabulary intensity too high**: Start at 0.4–0.6. Higher values cause false positives across unrelated words.
- **Forgetting to set encoding/sample_rate for live**: These are required to parse audio chunks correctly. Mismatch causes garbled transcription.
- **Multi-channel audio billing**: Transcribing N-channel audio is billed as N × duration. Merge channels only if necessary.
- **File size near 1000 MB**: Split into ~60-minute chunks before uploading. Larger files fail silently.
- **Language detection on code-switched audio**: Don't enable code switching with an empty `languages` list. Specify 3–5 expected languages.
- **PII redaction only for pre-recorded**: Not available for live transcription. Plan compliance workflows accordingly.
- **Webhooks vs. callbacks**: Webhooks are configured in the dashboard; callbacks are per-request. Use callbacks for one-off jobs.

## Verification checklist

Before submitting work with Gladia:

- [ ] API key is set in `x-gladia-key` header (not in body or query params)
- [ ] Audio URL is valid and accessible (or file uploaded successfully)
- [ ] Model matches use case: `solaria-3` for pre-recorded, `solaria-1` for live
- [ ] Language config is set correctly (explicit language or auto-detect, not both)
- [ ] If using solaria-3 with multiple languages, set exactly one language
- [ ] Diarization config (min/max speakers) is reasonable for the content
- [ ] Custom vocabulary entries have pronunciations for phonetically wrong terms
- [ ] PII redaction entity types match compliance requirements (GDPR, HIPAA, etc.)
- [ ] Live session encoding/sample_rate/bit_depth match actual audio format
- [ ] File size is under 1000 MB; duration under 135 min (or 4h 15m for enterprise)
- [ ] Polling includes backoff or uses webhooks/callbacks
- [ ] Response parsing handles both success and error states
- [ ] Transcription quality is acceptable (test with sample audio first)

## Resources

**Comprehensive navigation**: https://docs.gladia.io/llms.txt

**Critical pages**:
- [Pre-recorded quickstart](https://docs.gladia.io/chapters/pre-recorded-stt/quickstart) — end-to-end transcription workflow
- [Live quickstart](https://docs.gladia.io/chapters/live-stt/quickstart) — real-time transcription setup
- [Audio intelligence features](https://docs.gladia.io/chapters/audio-intelligence/) — diarization, translation, sentiment, PII redaction, and more

---

> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
---

> This file is auto-synced from https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
> Do not edit manually — changes will be overwritten by CI.
> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
