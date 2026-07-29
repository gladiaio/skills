---
name: gladia-documentation-auto
description: Comprehensive Gladia speech-to-text reference auto-synced from docs.gladia.io. Use as a general-purpose fallback when other specialized skills don't match, or when the user needs a broad overview of Gladia capabilities, endpoints, decision guidance, or workflows. Always prefer the official SDK; fall back to raw REST/WebSocket only when SDK cannot satisfy the requirement.
license: MIT
metadata:
  source: https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
  digest: sha256:cb36d328ed195df119a37786db62d66aa0a88eda05f8520ddac648e26a00601f
  synced: "2026-07-29"
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
description: Use when building speech-to-text transcription features, processing audio or video files, implementing real-time transcription, extracting insights from audio (speaker identification, translation, sentiment, summaries), or integrating transcription into applications via API or SDK.
metadata:
    mintlify-proj: gladia
    version: "1.0"
---

# Gladia Skill

## Product summary

Gladia is a speech-to-text (STT) API that transcribes audio and video files in two modes: **pre-recorded** (asynchronous, for files) and **live** (real-time, for streaming audio). Beyond transcription, it offers audio intelligence features like speaker diarization, translation, sentiment analysis, summarization, and named entity recognition. Use the **JavaScript/TypeScript SDK** (`@gladiaio/sdk`) or **Python SDK** (`gladiaio-sdk`) for simplified integration, or call the REST API directly. Authenticate with the `x-gladia-key` header. Primary docs: https://docs.gladia.io

**Key endpoints:**
- Pre-recorded: `POST /v2/pre-recorded` (create job), `GET /v2/pre-recorded/:id` (get result)
- Live: `POST /v2/live` (init session), WebSocket connection for streaming
- Upload: `POST /v2/upload` (for local files)

**Models:** `solaria-3` (pre-recorded, highest quality), `solaria-1` (live only)

## When to use

Reach for Gladia when:
- **Transcribing files**: User uploads an audio/video file and needs a transcript (meetings, podcasts, interviews, call recordings)
- **Real-time transcription**: Streaming audio from a microphone, phone call, or live event that needs instant captions or transcripts
- **Extracting insights**: Need to identify speakers, translate to other languages, detect sentiment, summarize, or extract entities from audio
- **Multi-language support**: Audio contains multiple languages or needs translation to 100+ target languages
- **Subtitle generation**: Creating SRT/VTT subtitle files for video content
- **Integration**: Building a feature into an app or workflow (via SDK or API)

Do **not** use Gladia for: text-only processing, image analysis, or non-audio content.

## Quick reference

### Authentication
```bash
# All requests require the x-gladia-key header
curl -H "x-gladia-key: YOUR_API_KEY" https://api.gladia.io/v2/...
```

### Pre-recorded workflow (SDK)
```javascript
const { GladiaClient } = require("@gladiaio/sdk");
const client = new GladiaClient({ apiKey: "YOUR_KEY" });

// One-call transcription
const result = await client.preRecorded().transcribe("audio.mp3", {
  model: "solaria-3",
  language_config: { languages: ["en"] },
  diarization: true,
  translation: true,
  translation_config: { target_languages: ["fr"] }
});
```

### Live workflow (SDK)
```javascript
const config = {
  model: "solaria-1",
  encoding: "wav/pcm",
  sample_rate: 16000,
  bit_depth: 16,
  channels: 1,
  language_config: { languages: ["en"] }
};

const session = client.liveV2().startSession(config);
session.on("message", (msg) => console.log(msg));
session.sendAudio(audioChunk);
session.stopRecording();
```

### Supported audio formats
MP3, WAV, FLAC, OGG, OPUS, AAC, M4A, AC3, EAC3, MP2

### Supported video formats
MP4, MOV, AVI, MKV, FLV, 3GP, WMV, M4V

### Language codes
Use 2-letter ISO 639-1 codes: `en`, `fr`, `de`, `es`, `zh`, `ja`, etc. Full list at `/chapters/language/supported-languages`

### Audio intelligence features
| Feature | Pre-recorded | Live | Config key |
|---------|--------------|------|-----------|
| Speaker diarization | ✓ | ✗ | `diarization` |
| Translation | ✓ | ✓ | `translation` |
| Sentiment analysis | ✓ | ✗ | `sentiment_analysis` |
| Summarization | ✓ | ✗ | `summarization` |
| Named entity recognition | ✓ | ✓ | `named_entity_recognition` |
| PII redaction | ✓ | ✗ | `pii_redaction` |
| Subtitles (SRT/VTT) | ✓ | ✗ | `subtitles` |
| Chapterization | ✓ | ✗ | `chapterization` |
| Custom vocabulary | ✓ | ✓ | `custom_vocabulary` |

## Decision guidance

### Pre-recorded vs. Live

| Scenario | Use pre-recorded | Use live |
|----------|------------------|----------|
| User uploads a file | ✓ | ✗ |
| Streaming microphone input | ✗ | ✓ |
| Phone call transcription | ✗ | ✓ |
| Podcast/interview processing | ✓ | ✗ |
| Real-time captions | ✗ | ✓ |
| Need diarization | ✓ | ✗ (use multi-channel instead) |
| Need sentiment/summarization | ✓ | ✗ |

### Model selection

| Model | Use case | Availability |
|-------|----------|--------------|
| `solaria-3` | Highest accuracy, best for meetings/calls/podcasts | Pre-recorded only |
| `solaria-1` | Real-time streaming, lower latency | Live only |

### Language detection vs. explicit language

| Approach | When to use |
|----------|------------|
| Explicit `language_config.languages: ["en"]` | Language is known; avoids detection overhead |
| Auto-detect (omit languages) | Language unknown or mixed; slower but flexible |
| Code switching `code_switching: true` | Audio mixes multiple languages; requires language list |

### Diarization vs. multi-channel

| Approach | When to use |
|----------|------------|
| `diarization: true` | Single audio stream, multiple speakers (meetings, interviews) |
| Multi-channel (channels > 1) | Separate audio tracks per speaker (phone calls with distinct channels) |

## Workflow

### Pre-recorded transcription (file upload)

1. **Prepare the file**: Ensure audio/video is in a supported format (MP3, WAV, MP4, etc.) and under 1000 MB. For files >135 minutes, split into ~60-minute chunks.

2. **Upload the file** (if local):
   ```bash
   curl -X POST https://api.gladia.io/v2/upload \
     -H "x-gladia-key: YOUR_KEY" \
     -F "audio=@audio.mp3"
   ```
   Save the returned `audio_url`.

3. **Create transcription job**:
   ```bash
   curl -X POST https://api.gladia.io/v2/pre-recorded \
     -H "x-gladia-key: YOUR_KEY" \
     -H "Content-Type: application/json" \
     -d '{
       "audio_url": "https://api.gladia.io/file/...",
       "model": "solaria-3",
       "language_config": { "languages": ["en"] },
       "diarization": true
     }'
   ```
   Save the returned `id`.

4. **Poll for results**:
   ```bash
   curl https://api.gladia.io/v2/pre-recorded/ID \
     -H "x-gladia-key: YOUR_KEY"
   ```
   Repeat until `status: "done"`. Or use webhooks/callbacks instead of polling.

5. **Extract data**: Parse `result.transcription.utterances` for text, timing, speaker, language, and confidence.

### Live transcription (streaming)

1. **Initialize session**:
   ```bash
   curl -X POST https://api.gladia.io/v2/live \
     -H "x-gladia-key: YOUR_KEY" \
     -H "Content-Type: application/json" \
     -d '{
       "encoding": "wav/pcm",
       "sample_rate": 16000,
       "bit_depth": 16,
       "channels": 1
     }'
   ```
   Save the returned WebSocket `url`.

2. **Connect WebSocket**: Open a WebSocket connection to the URL.

3. **Send audio chunks**: Stream audio in the specified encoding/sample rate.

4. **Read messages**: Listen for `transcript`, `translation`, `sentiment_analysis`, etc. messages.

5. **Stop recording**: Send `{ "type": "stop_recording" }` or close the socket with code 1000.

6. **Retrieve final results**: Call `GET /v2/live/:id` to get the complete result.

## Common gotchas

- **Model mismatch**: `solaria-3` is pre-recorded only; `solaria-1` is live only. Using the wrong model will fail.
- **Language config with solaria-3**: Must set exactly **one** language in `language_config.languages` (e.g., `["fr"]`). Do not pass multiple languages or enable code switching with solaria-3.
- **Polling timeout**: Pre-recorded jobs can take minutes. Don't assume instant results; use webhooks or callbacks for production.
- **File size limits**: Max 1000 MB per file; max 135 minutes per job (enterprise plans support 4h15). Split large files.
- **Live session duration**: Max 3 hours per WebSocket session. Start a new session before hitting the limit.
- **Audio format mismatch**: Ensure `encoding`, `sample_rate`, `bit_depth`, and `channels` match your actual audio stream, or transcription will fail silently.
- **Diarization hints are not constraints**: `number_of_speakers`, `min_speakers`, `max_speakers` are hints, not hard limits. The model may detect a different count.
- **Translation with code switching**: Do not enable both `code_switching: true` and `translation` on the same request without a constrained language list.
- **Concurrency limits**: Free tier: 3 concurrent pre-recorded, 1 live. Paid: 25 concurrent pre-recorded, 30 live. Queued requests beyond concurrency limits will wait.
- **Deprecated V1 API**: Old endpoints like `/audio/text/audio-transcription/` no longer work. Use `/v2/pre-recorded` and `/v2/live`.

## Verification checklist

Before submitting transcription work:

- [ ] API key is valid and passed in `x-gladia-key` header
- [ ] Audio file is in a supported format (MP3, WAV, MP4, etc.)
- [ ] File size is under 1000 MB; duration under 135 minutes (or split into chunks)
- [ ] For pre-recorded: `model` is `solaria-3`; for live: `model` is `solaria-1`
- [ ] Language code is valid ISO 639-1 (e.g., `en`, `fr`, `de`)
- [ ] For solaria-3: exactly one language in `language_config.languages`
- [ ] Audio encoding, sample rate, bit depth, and channels match the actual stream (live only)
- [ ] Diarization is enabled if speaker identification is needed
- [ ] Translation target languages are in the supported list
- [ ] Webhook/callback URL is reachable (if using async notifications)
- [ ] Job status is `done` before reading results
- [ ] Response contains `result.transcription.utterances` with text and timing

## Resources

- **Full page navigation**: https://docs.gladia.io/llms.txt
- **API Reference**: https://docs.gladia.io/api-reference/
- **Pre-recorded quickstart**: https://docs.gladia.io/chapters/pre-recorded-stt/quickstart
- **Live transcription quickstart**: https://docs.gladia.io/chapters/live-stt/quickstart
- **Audio intelligence features**: https://docs.gladia.io/chapters/audio-intelligence/
- **Recommended parameters by use case**: https://docs.gladia.io/chapters/pre-recorded-stt/recommended-parameters
- **Supported languages**: https://docs.gladia.io/chapters/language/supported-languages
- **SDK (JavaScript)**: https://www.npmjs.com/package/@gladiaio/sdk
- **SDK (Python)**: https://pypi.org/project/gladiaio-sdk/

---

> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
---

> This file is auto-synced from https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
> Do not edit manually — changes will be overwritten by CI.
> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
