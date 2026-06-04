---
name: documentation-auto
description: Comprehensive Gladia speech-to-text reference auto-synced from docs.gladia.io. Use as a general-purpose fallback when other specialized skills don't match, or when the user needs a broad overview of Gladia capabilities, endpoints, decision guidance, or workflows. Always prefer the official SDK; fall back to raw REST/WebSocket only when SDK cannot satisfy the requirement.
license: MIT
metadata:
  source: https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
  digest: sha256:f87953eda33c8e6132d8a78343532fddc576ebeab7bf83bb3b6fb3aca2b5c96b
  synced: "2026-06-04"
---

> **SDK-first**: always use the official SDK — see [sdk-integration](../sdk-integration/SKILL.md) for policy, setup, and fallback criteria.

## References

Consult these sibling skills as needed:

- ../sdk-integration/SKILL.md -- SDK setup, client initialization, error handling, and SDK vs raw API decision guide
- ../sdk-integration/references/sdk-versions.md -- Current SDK versions (auto-synced by CI)
- ../troubleshooting/SKILL.md -- Common errors, gotchas, and verification checklist
- ../live-transcription/SKILL.md -- Live streaming transcription
- ../pre-recorded-transcription/SKILL.md -- Pre-recorded file transcription

---
name: Gladia
description: Use when building speech-to-text transcription features, processing audio or video files, implementing real-time transcription, extracting insights from audio (translation, summarization, speaker identification), or integrating audio intelligence into applications.
metadata:
    mintlify-proj: gladia
    version: "1.0"
---

# Gladia Skill

## Product summary

Gladia is a speech-to-text API that transcribes audio and video files in two modes: **pre-recorded** (asynchronous, batch) and **live** (real-time, WebSocket-based). The API returns structured transcripts with word-level timing, confidence scores, and optional audio intelligence features (translation, diarization, summarization, entity recognition, sentiment analysis, PII redaction, subtitles). Use the JavaScript/TypeScript SDK (`@gladiaio/sdk`) or Python SDK (`gladiaio-sdk`) for simplified integration, or call REST/WebSocket endpoints directly. Authenticate with `x-gladia-key` header. Primary docs: https://docs.gladia.io

## When to use

- **Pre-recorded transcription**: Transcribe uploaded audio/video files (MP3, WAV, MP4, YouTube links, etc.) asynchronously. Typical latency: seconds to minutes depending on file length.
- **Live transcription**: Stream audio in real-time via WebSocket for immediate transcripts (e.g., call centers, live events, voice assistants).
- **Audio intelligence**: Extract metadata from transcripts — translate to multiple languages, identify speakers, detect sentiment, redact PII, generate summaries, create subtitles, recognize named entities.
- **Custom vocabulary**: Improve accuracy for domain-specific terms, brand names, proper nouns by providing phonetic hints.
- **Multi-speaker scenarios**: Use diarization to attribute speech to individual speakers, or send multi-channel audio to preserve speaker identity.

## Quick reference

### Authentication
```bash
# All requests require x-gladia-key header
curl -H "x-gladia-key: YOUR_API_KEY" https://api.gladia.io/v2/pre-recorded
```

### Pre-recorded workflow (SDK)
```javascript
import { GladiaClient } from "@gladiaio/sdk";
const client = new GladiaClient({ apiKey: "YOUR_KEY" });
const result = await client.preRecorded().transcribe("audio_url_or_local_path");
```

### Live workflow (SDK)
```javascript
const session = client.liveV2().startSession({
  encoding: "wav/pcm",
  sample_rate: 16000,
  bit_depth: 16,
  channels: 1,
  language_config: { languages: ["en"] }
});
session.on("message", (msg) => console.log(msg));
session.sendAudio(audioChunk);
session.stopRecording();
```

### Audio formats
| Type | Examples |
|------|----------|
| Audio | MP3, WAV, FLAC, AAC, OGG, Opus |
| Video | MP4, MOV, AVI, WebM, Matroska |
| Online | YouTube, TikTok, Instagram, Facebook, Vimeo, LinkedIn |

### Limits
| Limit | Value |
|-------|-------|
| Pre-recorded max duration | 135 minutes (free/paid); 4h15 (enterprise) |
| Pre-recorded max file size | 1000 MB |
| Live session max duration | 3 hours |
| Free tier monthly usage | 10 hours |
| Concurrent pre-recorded jobs (free) | 3 |
| Concurrent pre-recorded jobs (paid) | 25 |
| Concurrent live sessions (free) | 1 |
| Concurrent live sessions (paid) | 30 |

### Audio intelligence features
| Feature | Pre-recorded | Live | Purpose |
|---------|--------------|------|---------|
| Diarization | ✓ | ✗ | Identify speakers |
| Translation | ✓ | ✓ | Multi-language output |
| Summarization | ✓ | ✗ | Generate summaries/bullet points |
| Sentiment analysis | ✓ | ✓ | Detect emotions and tone |
| Named entity recognition | ✓ | ✓ | Extract people, orgs, dates |
| PII redaction | ✓ | ✗ | Anonymize sensitive data |
| Subtitles | ✓ | ✗ | Generate SRT/VTT files |
| Custom vocabulary | ✓ | ✓ | Improve domain-specific terms |
| Custom spelling | ✓ | ✓ | Normalize misspellings |
| Chapterization | ✓ | ✗ | Segment long audio into chapters |
| Audio-to-LLM | ✓ | ✗ | Run custom prompts on transcript |

## Decision guidance

### When to use pre-recorded vs. live

| Scenario | Pre-recorded | Live |
|----------|--------------|------|
| Batch processing uploaded files | ✓ | ✗ |
| Real-time streaming (calls, events) | ✗ | ✓ |
| Need diarization | ✓ | ✗ |
| Need immediate partial results | ✗ | ✓ (with `receive_partial_transcripts: true`) |
| Need summarization | ✓ | ✗ |
| Multi-hour content | ✓ (up to 135 min) | ✓ (up to 3 hours per session) |

### When to use SDK vs. raw API

| Approach | Best for |
|----------|----------|
| SDK | Rapid development, automatic error handling, built-in polling/retry logic |
| Raw API | Custom workflows, specific language/framework, fine-grained control |

### When to use diarization vs. multi-channel audio

| Approach | Use when |
|----------|----------|
| Diarization | Single audio file with multiple speakers; you want the API to separate them |
| Multi-channel | Multiple audio sources (e.g., separate participant feeds); you can merge them into one multi-channel stream |

### When to use custom vocabulary vs. custom spelling

| Feature | Use when |
|---------|----------|
| Custom vocabulary | Word is mispronounced/garbled; you provide phonetic hints (e.g., "Nietzsche" → ["Niche", "Neechee"]) |
| Custom spelling | Word is recognized but misspelled (e.g., "Salesforce" → "Sales Force"); literal text matching |

## Workflow

### Pre-recorded transcription (typical task)

1. **Prepare audio**: Ensure file is under 1000 MB and 135 minutes. Supported formats: MP3, WAV, MP4, YouTube URL, etc.
2. **Choose delivery method**: Use SDK for simplicity, or raw API for control.
3. **Configure transcription**:
   - Set `language_config.languages` explicitly if known (avoids detection overhead).
   - Enable `diarization: true` if multiple speakers.
   - Add `custom_vocabulary` for domain terms.
   - Enable audio intelligence features (translation, summarization, etc.) as needed.
4. **Submit job**: Call `transcribe()` (SDK) or `POST /v2/pre-recorded` (API).
5. **Retrieve results**: Poll `GET /v2/pre-recorded/:id` or configure webhooks/callbacks.
6. **Parse response**: Extract `transcription.utterances[]` for text and timing, plus any audio intelligence results.

### Live transcription (typical task)

1. **Initialize session**: Call `POST /v2/live` with audio config (encoding, sample_rate, bit_depth, channels).
2. **Connect WebSocket**: Use returned URL to open WebSocket connection.
3. **Configure messages**: Set `messages_config` to specify which message types to receive (transcripts, partial transcripts, post-processing events).
4. **Stream audio**: Send audio chunks via `sendAudio()` (SDK) or binary/base64 JSON (raw API).
5. **Handle messages**: Listen for `transcript` messages; check `is_final` to distinguish partials from finals.
6. **Stop recording**: Call `stopRecording()` to trigger post-processing (diarization, translation, etc.).
7. **Retrieve final result**: Poll `GET /v2/live/:id` or wait for callback with complete result.

### Adding custom vocabulary

1. **Identify problem terms**: Transcribe without custom vocabulary; note mis-transcribed words.
2. **Categorize**: Garbled/phonetically wrong → custom vocabulary; recognizable but misspelled → custom spelling.
3. **Build vocabulary list**:
   ```json
   {
     "custom_vocabulary": true,
     "custom_vocabulary_config": {
       "vocabulary": [
         "Gladia",
         { "value": "Salesforce", "pronunciations": ["sell force"], "intensity": 0.5 }
       ],
       "default_intensity": 0.4
     }
   }
   ```
4. **Test**: Transcribe again; confirm targets appear and check for false positives.
5. **Refine**: Lower intensity, add pronunciations, or move stubborn terms to custom spelling.

## Common gotchas

- **Language detection overhead**: Always set `language_config.languages` explicitly if you know the language. Auto-detection adds latency and can fail if audio starts with silence or music.
- **Code switching without language list**: Never enable `code_switching: true` with an empty `languages` array — the model will evaluate against 100+ languages, causing frequent misdetections. Always provide a constrained list (3–5 languages).
- **Diarization hints are not hard constraints**: `number_of_speakers`, `min_speakers`, `max_speakers` are hints, not guarantees. The model may detect a different count.
- **Custom vocabulary intensity tuning**: Start at `default_intensity: 0.4` and adjust per-entry only. Raising intensity globally increases false positives. Add `pronunciations` variants before raising intensity.
- **Live session 3-hour limit**: A single WebSocket session cannot exceed 3 hours. For longer events, close the session and start a new one before hitting the limit.
- **Pre-recorded 135-minute limit**: Files longer than 135 minutes will fail. Split into ~60-minute chunks using ffmpeg or similar tools.
- **Audio format conversion overhead**: Large video files (e.g., AVI, MOV) take ~1 minute to convert to WAV/PCM. Plan for this latency.
- **Polling without webhooks**: If you poll `GET /v2/pre-recorded/:id` in a tight loop, you'll hit rate limits. Use webhooks or callbacks instead, or poll with exponential backoff.
- **Multi-channel billing**: Transcribing multi-channel audio is billed as `duration × number_of_channels`. A 10-minute 3-channel stream costs 30 minutes of usage.
- **Partial transcripts in live mode**: Partial transcripts are low-latency but less accurate. Always check `is_final: true` before using a transcript for critical decisions.
- **Missing audio_url on upload**: After uploading a file, the response includes `audio_url` — use this URL in the transcription request, not the local file path.
- **WebSocket reconnection**: If the WebSocket disconnects, reconnect to the same URL (returned from init) to resume the session without losing context.

## Verification checklist

Before submitting transcription work:

- [ ] API key is valid and passed in `x-gladia-key` header.
- [ ] Audio file is under 1000 MB and 135 minutes (pre-recorded) or 3 hours (live).
- [ ] Audio format is supported (MP3, WAV, MP4, etc.).
- [ ] Language is set explicitly in `language_config.languages` if known.
- [ ] If using code switching, `languages` list is constrained to 3–5 expected languages.
- [ ] Diarization is enabled if multiple speakers need attribution.
- [ ] Custom vocabulary entries have realistic `intensity` (0.4–0.6) and `pronunciations`.
- [ ] Webhooks or callbacks are configured if polling is not feasible.
- [ ] Live sessions are closed before 3 hours; pre-recorded jobs are split if over 135 minutes.
- [ ] Response includes expected fields: `transcription.utterances[]`, `metadata`, and any requested audio intelligence results.
- [ ] Confidence scores and timing (`start`, `end`) are present for quality validation.
- [ ] Multi-channel audio is correctly interleaved if merging multiple sources.

## Resources

- **Comprehensive page listing**: https://docs.gladia.io/llms.txt
- **Getting started guide**: https://docs.gladia.io/chapters/introduction/getting-started
- **Pre-recorded quickstart**: https://docs.gladia.io/chapters/pre-recorded-stt/quickstart
- **Live transcription quickstart**: https://docs.gladia.io/chapters/live-stt/quickstart
- **API reference**: https://docs.gladia.io/api-reference
- **Recommended parameters by use case**: https://docs.gladia.io/chapters/pre-recorded-stt/recommended-parameters
- **Audio intelligence features**: https://docs.gladia.io/chapters/audio-intelligence
- **Supported formats and limits**: https://docs.gladia.io/chapters/limits-and-specifications/supported-formats

---

> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
---

> This file is auto-synced from https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
> Do not edit manually — changes will be overwritten by CI.
> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
