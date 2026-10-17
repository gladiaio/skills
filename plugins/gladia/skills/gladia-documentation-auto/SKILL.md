---
name: gladia-documentation-auto
description: Comprehensive Gladia speech-to-text reference auto-synced from docs.gladia.io. Use as a general-purpose fallback when other specialized skills don't match, or when the user needs a broad overview of Gladia capabilities, endpoints, decision guidance, or workflows. Always prefer the official SDK; fall back to raw REST/WebSocket only when SDK cannot satisfy the requirement.
license: MIT
metadata:
  source: https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
  digest: sha256:b43e3051df7c5bb8d90571bcc4e56e519c83a3adffd81724a22371ee34863380
  synced: "2026-10-17"
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
  audio or video for real-time or batch transcription, extracting insights from
  audio (diarization, translation, sentiment), or integrating transcription into
  voice agents, meeting recorders, or multilingual applications.
metadata:
  mintlify-proj: gladia
  version: "1.0"
---

# Gladia Speech-to-Text API

## Product summary

Gladia is a speech-to-text API that converts audio and video into text with optional audio intelligence features (diarization, translation, sentiment analysis, PII redaction, summarization, and more). Use the SDK (JavaScript/Python), REST API, or CLI to transcribe pre-recorded audio asynchronously or stream live audio in real time. Authentication uses a single API key passed in the `x-gladia-key` header. Primary docs: https://docs.gladia.io

**Key files and commands:**
- SDK: `npm install @gladiaio/sdk` (JavaScript) or `pip install gladiaio-sdk` (Python)
- CLI: `gladia transcribe audio.mp3` (install via https://github.com/gladiaio/gladia-cli)
- API endpoints: `POST /v2/pre-recorded` (async), `POST /v2/live` (streaming), `GET /v2/pre-recorded/:id` (poll results)
- Authentication: Pass `x-gladia-key: YOUR_API_KEY` in request headers

## When to use

Reach for Gladia when:
- **Transcribing pre-recorded audio/video** — batch jobs that can wait for results (use SDK's `transcribe()` or API polling)
- **Streaming live audio** — real-time transcription from microphones, WebRTC, or telephony (use SDK's `liveV2()` or WebSocket API)
- **Extracting structured data from audio** — speaker identification, translation, sentiment, entities, summaries, or PII redaction
- **Building multilingual products** — 100+ language support with auto-detection or code switching (Solaria-1 only)
- **Integrating into voice agents or meeting bots** — use Pipecat, Livekit, Vapi, Recall, or Twilio integrations
- **Testing transcription quality** — use the playground at https://app.gladia.io/playground or CLI for quick validation

Do not use Gladia for:
- Synchronous transcription with sub-second latency (live sessions have inherent streaming latency)
- Audio longer than 135 minutes in a single pre-recorded request (enterprise plans support up to 4h15)
- Live sessions exceeding 3 hours (start a new session before the limit)

## Quick reference

### Authentication
Always include the API key header:
```bash
-H "x-gladia-key: YOUR_GLADIA_API_KEY"
```

### Models
| Model | Best for | Modes | Languages | Code switching |
|-------|----------|-------|-----------|---|
| **Solaria-3** | European real-world audio, highest accuracy | Pre-recorded only | EN, FR, DE, ES, IT | No |
| **Solaria-1** (default) | Global coverage, live streaming, clean speech | Pre-recorded + live | 100+ | Yes |

### Pre-recorded workflow (SDK)
```javascript
const gladia = new GladiaClient({ apiKey: "YOUR_KEY" });
const result = await gladia.preRecorded().transcribe("audio.mp3", {
  model: "solaria-3",
  language_config: { languages: ["en"] },
  diarization: true,
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
session.on("message", (msg) => {
  if (msg.type === "transcript" && msg.data.is_final) {
    console.log(msg.data.utterance.text);
  }
});
session.sendAudio(audioChunk);
session.stopRecording();
```

### CLI quick commands
```bash
gladia transcribe meeting.wav                    # text to stdout
gladia transcribe podcast.mp3 -o json            # JSON output
gladia transcribe call.wav --diarize -o srt      # subtitles with speakers
gladia transcribe mixed.mp3 --code-switching     # mixed languages
gladia transcribe audio.mp3 --model solaria-3 --language en
```

### Audio Intelligence features
| Feature | Pre-recorded | Live | Use case |
|---------|---|---|---|
| Speaker diarization | ✓ | ✓ | Who spoke when |
| Translation | ✓ | ✓ | Multilingual output (100+ languages) |
| Sentiment analysis | ✓ | ✓ | Tone and emotion detection |
| PII redaction | ✓ | ✗ | GDPR/compliance masking |
| Summarization | ✓ | ✓ | Bullet points or concise summary |
| Named entity recognition | ✓ | ✓ | Extract people, orgs, dates |
| Custom vocabulary | ✓ | ✓ | Boost accuracy for domain terms |
| Subtitles (SRT/VTT) | ✓ | ✗ | Video captions |

### Result retrieval methods
| Method | Pre-recorded | Live | Best for |
|--------|---|---|---|
| **Polling** | `GET /v2/pre-recorded/:id` | `GET /v2/live/:id` | Simple integrations, low volume |
| **Webhooks** | `transcription.created`, `.success`, `.error` | `start-session`, `end-session` | Production, event-driven |
| **Callbacks** | `callback_config.url` (POST/PUT) | `callback_config.url` | Async notification to your endpoint |
| **SDK methods** | `poll()`, `createAndPoll()` | Event listeners (`on("message")`) | Recommended for most cases |

## Decision guidance

### When to use Solaria-3 vs Solaria-1

| Condition | Use Solaria-3 | Use Solaria-1 |
|-----------|---|---|
| Need live/real-time transcription | ✗ | ✓ |
| Audio is pre-recorded | ✓ | ✓ |
| Language is EN, FR, DE, ES, or IT | ✓ | ✓ |
| Need 100+ languages or code switching | ✗ | ✓ |
| Prioritize accuracy on noisy/conversational audio | ✓ | ✗ |
| Audio is clean, formal, or read speech | ✓ | ✓ |

### When to poll vs webhook vs callback

| Scenario | Approach | Reason |
|----------|----------|--------|
| Testing or low-volume integration | Polling | Simple, no server setup |
| Production, high volume, event-driven | Webhooks | Gladia notifies you; configure at https://app.gladia.io/webhooks |
| Need custom notification logic | Callbacks | Pass `callback_config` in job request; Gladia POSTs to your URL |
| Real-time user feedback (live) | Event listeners (SDK) | Immediate message handling via WebSocket |

### When to use SDK vs raw API

| Factor | SDK | Raw API |
|--------|-----|---------|
| Language support | JavaScript, Python | Any (cURL, Go, Rust, etc.) |
| Complexity | Handles upload, polling, retries, WebSocket lifecycle | Manual orchestration |
| Recommended for | Most integrations | Custom workflows, CI/CD, non-standard environments |

## Workflow

### Pre-recorded transcription (typical flow)

1. **Get API key** — Create one at https://app.gladia.io/apikeys
2. **Prepare audio** — Local file, binary data, or remote URL (must be publicly accessible for URLs)
3. **Choose model** — Solaria-3 for European audio (EN/FR/DE/ES/IT), Solaria-1 for global/live
4. **Configure features** — Add diarization, translation, sentiment, etc. as needed
5. **Submit job** — Call `POST /v2/pre-recorded` or SDK's `transcribe()`
6. **Retrieve results** — Poll `GET /v2/pre-recorded/:id`, use webhooks, or SDK's `poll()`
7. **Parse output** — Extract `result.transcription.full_transcript`, `result.translation`, `result.sentiment_analysis`, etc.

### Live transcription (typical flow)

1. **Initialize session** — Call `POST /v2/live` with audio format (encoding, sample_rate, bit_depth, channels)
2. **Get WebSocket URL** — Response includes `url` and `id`
3. **Connect WebSocket** — Open connection to the URL (SDK handles this)
4. **Send audio chunks** — Stream audio data as binary or base64-encoded JSON
5. **Handle messages** — Listen for `transcript` messages; check `is_final` to distinguish partial vs final
6. **Stop recording** — Send `stop_recording` message or close WebSocket with code 1000
7. **Retrieve final results** — Poll `GET /v2/live/:id` or receive via callback/webhook

### CLI workflow (quick transcription)

1. **Install CLI** — `curl -fsSL https://github.com/gladiaio/gladia-cli/releases/latest/download/install.sh | sh`
2. **Set API key** — `export GLADIA_API_KEY=your_key` or `gladia auth set your_key`
3. **Transcribe** — `gladia transcribe audio.mp3 -o json` (or `srt`, `vtt`, `text`)
4. **Pipe output** — `gladia transcribe audio.mp3 -o json | jq '.transcription'`

## Common gotchas

- **Solaria-3 language constraint** — With Solaria-3, pass exactly one language in `language_config.languages` (e.g., `["fr"]`). Do not enable code switching or pass multiple languages; the model will fail or produce poor results.
- **Live sessions limited to 3 hours** — A single WebSocket session cannot exceed 3 hours. For longer events, close the session and start a new one before the limit.
- **Pre-recorded audio limited to 135 minutes** — Single requests cannot exceed 135 minutes (enterprise plans support up to 4h15). Split longer files into chunks.
- **Do not resubmit jobs** — If `POST /v2/pre-recorded` returns 200 or you receive a `transcription.created` webhook, the job is accepted. Do not resubmit the same audio; wait on that job ID. Resubmitting causes duplicate charges.
- **Audio format must match config** — Encoding, sample_rate, bit_depth, and channels in the init request must exactly match the audio you send. Mismatches cause transcription errors or garbled output.
- **Partial transcripts are provisional** — When `receive_partial_transcripts: true`, partial messages have `is_final: false`. Do not display them as final; replace them with the final transcript when `is_final: true` arrives for the same utterance ID.
- **429 errors mean concurrency or rate limits** — Reduce parallel jobs/sessions or slow the request rate. Check your plan limits at https://app.gladia.io/dashboard.
- **Webhooks require public HTTPS endpoint** — Callback URLs must be publicly accessible and use HTTPS. Test with a tool like ngrok if developing locally.
- **Custom vocabulary is phoneme-based** — Provide pronunciations or let Gladia infer them. Spelling-based matching does not work; use `custom_spelling` for text replacement instead.
- **Multi-channel audio is billed per channel** — A 2-channel live session is billed as 2× the audio duration. Set `channels` in the init request to match your audio.
- **Deprecated endpoints** — `/v2/transcription` is deprecated; use `/v2/pre-recorded` instead. `/v2/upload` is disabled in some plans; use external URLs or SDK's `uploadFile()`.

## Verification checklist

Before submitting transcription work:

- [ ] API key is set and valid (test with a simple request)
- [ ] Audio format (encoding, sample_rate, bit_depth, channels) matches the config in the init request
- [ ] Model choice is correct (Solaria-3 for European audio, Solaria-1 for live or 100+ languages)
- [ ] If using Solaria-3, language_config has exactly one language and code_switching is false
- [ ] Audio URL is publicly accessible (for pre-recorded) or audio chunks are being sent (for live)
- [ ] Job ID is captured and stored for polling or callback handling
- [ ] Callback/webhook URL is HTTPS and publicly accessible (if using callbacks)
- [ ] Result parsing handles both partial (`is_final: false`) and final (`is_final: true`) messages correctly
- [ ] Audio intelligence features (diarization, translation, etc.) are enabled only if needed (they add latency and cost)
- [ ] Concurrency limits are respected (check your plan; 429 errors indicate exhausted capacity)
- [ ] For live sessions, `stopRecording()` is called or WebSocket is closed with code 1000 to trigger post-processing

## Resources

**Comprehensive navigation:** https://docs.gladia.io/llms.txt

**Critical documentation pages:**
1. [Pre-recorded STT quickstart](https://docs.gladia.io/chapters/pre-recorded-stt/quickstart) — Upload, job creation, polling, webhooks, and audio intelligence
2. [Live STT quickstart](https://docs.gladia.io/chapters/live-stt/quickstart) — WebSocket setup, audio streaming, partial/final transcripts, multi-channel
3. [Models guide](https://docs.gladia.io/chapters/introduction/models) — Solaria-3 vs Solaria-1 comparison and selection criteria
4. [Audio Intelligence index](https://docs.gladia.io/chapters/audio-intelligence/) — Diarization, translation, sentiment, PII redaction, summarization, NER
5. [API Reference](https://docs.gladia.io/api-reference/) — Full endpoint documentation, request/response schemas, error codes
6. [Concurrency and rate limits](https://docs.gladia.io/chapters/limits-and-specifications/concurrency) — Plan limits, 429 handling, retry guidance
7. [CLI documentation](https://docs.gladia.io/chapters/how-to-use-gladia/cli) — Terminal transcription without code

---

> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
---

> This file is auto-synced from https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
> Do not edit manually — changes will be overwritten by CI.
> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
