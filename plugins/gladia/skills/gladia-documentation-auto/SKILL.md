---
name: gladia-documentation-auto
description: Comprehensive Gladia speech-to-text reference auto-synced from docs.gladia.io. Use as a general-purpose fallback when other specialized skills don't match, or when the user needs a broad overview of Gladia capabilities, endpoints, decision guidance, or workflows. Always prefer the official SDK; fall back to raw REST/WebSocket only when SDK cannot satisfy the requirement.
license: MIT
metadata:
  source: https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
  digest: sha256:1d7371c8d1abec687444f241e8bdf70df72389225c2665e17c56ab0a730d8a4d
  synced: "2026-08-24"
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
description: Use when building speech-to-text transcription features, processing audio or video files, implementing real-time transcription, extracting speaker information, translating transcripts, or adding audio intelligence features like summarization, sentiment analysis, and entity recognition.
metadata:
    mintlify-proj: gladia
    version: "1.0"
---

# Gladia Skill

## Product summary

Gladia is a speech-to-text (STT) API that transcribes audio and video files asynchronously (pre-recorded) or in real-time (live). It supports 100+ languages, speaker diarization, translation, and audio intelligence features like summarization, sentiment analysis, and entity recognition. Use the JavaScript/Python SDKs (`@gladiaio/sdk` or `gladiaio-sdk`), REST API endpoints, or the CLI (`gladia` command) to transcribe. Authentication uses the `x-gladia-key` header. Primary docs: https://docs.gladia.io

**Key endpoints:**
- Pre-recorded: `POST /v2/pre-recorded` (create job), `GET /v2/pre-recorded/{id}` (get result), `POST /v2/upload` (upload file)
- Live: `POST /v2/live` (init session), WebSocket connection for streaming audio
- Authentication: `x-gladia-key` header on all requests

## When to use

Reach for this skill when:
- A user asks to transcribe audio or video files (meetings, calls, podcasts, interviews)
- Building real-time transcription for live calls, meetings, or voice agents
- Extracting speaker information from multi-speaker audio (diarization)
- Translating transcripts to multiple languages
- Generating summaries, detecting sentiment, or extracting entities from audio
- Processing audio from URLs or local files
- Implementing webhooks or callbacks for async job completion
- Migrating from Deepgram or AssemblyAI to Gladia
- Using the CLI for quick terminal-based transcription

## Quick reference

### Models

| Model | Use case | Languages | Code switching | Live support |
|-------|----------|-----------|-----------------|--------------|
| `solaria-3` | Highest accuracy on European audio | en, fr, de, es, it | No | No (pre-recorded only) |
| `solaria-1` | Generalist, maximum coverage | 100+ languages | Yes | Yes |

### Pre-recorded workflow

1. **Upload** (optional): `POST /v2/upload` with multipart form-data → get `audio_url`
2. **Create job**: `POST /v2/pre-recorded` with `audio_url` and options → get `id`
3. **Get result**: Poll `GET /v2/pre-recorded/{id}` until `status: "done"` or use webhooks/callbacks

### Live workflow

1. **Init session**: `POST /v2/live` with audio config → get WebSocket `url` and `id`
2. **Connect**: Open WebSocket, send audio chunks as binary or base64
3. **Read messages**: Receive transcript, translation, sentiment, etc. via WebSocket
4. **Stop**: Send `stop_recording` message or close with code 1000

### Audio Intelligence features (pre-recorded & live)

| Feature | Parameter | Use case |
|---------|-----------|----------|
| Speaker diarization | `diarization: true` | Identify who spoke when |
| Translation | `translation: true` | Translate to multiple languages |
| Summarization | `summarization: true` | Generate summaries or bullet points |
| Sentiment analysis | `sentiment_analysis: true` | Extract emotions and tone |
| Named entity recognition | `named_entity_recognition: true` | Detect people, organizations, dates |
| PII redaction | `pii_redaction: true` | Anonymize sensitive data |
| Custom vocabulary | `custom_vocabulary: true` | Boost accuracy for domain terms |
| Subtitles | `subtitles: true` | Generate SRT or VTT files |

### SDK installation

```bash
# JavaScript
npm install @gladiaio/sdk

# Python
pip install gladiaio-sdk
# or
uv add gladiaio-sdk
```

### CLI installation

```bash
# macOS & Linux
curl -fsSL https://github.com/gladiaio/gladia-cli/releases/latest/download/install.sh | sh

# Windows (PowerShell)
powershell -c "irm https://github.com/gladiaio/gladia-cli/releases/latest/download/install.ps1 | iex"
```

### CLI quick commands

```bash
gladia auth set YOUR_API_KEY
gladia transcribe audio.wav                    # plain text
gladia transcribe audio.wav -o json            # JSON output
gladia transcribe audio.wav --diarize          # with speaker labels
gladia transcribe audio.wav --language en,fr   # constrain languages
gladia transcribe audio.wav --model solaria-3  # use specific model
```

## Decision guidance

### When to use pre-recorded vs. live

| Scenario | Use |
|----------|-----|
| User uploads a file, you transcribe later | Pre-recorded (`/v2/pre-recorded`) |
| Real-time call, meeting, or voice agent | Live (`/v2/live` + WebSocket) |
| Batch processing many files | Pre-recorded with polling or webhooks |
| Need results immediately as user speaks | Live with `receive_partial_transcripts: true` |

### When to use polling vs. webhooks vs. callbacks

| Approach | Best for | Trade-off |
|----------|----------|-----------|
| Polling | Small jobs, quick feedback | Keeps connection open, higher latency |
| Webhooks | Production, many jobs | Configure once at https://app.gladia.io/webhooks |
| Callbacks | Per-job control | Set `callback_config` in request body |

### When to use solaria-1 vs. solaria-3

| Condition | Choose |
|-----------|--------|
| European audio, high accuracy needed | `solaria-3` |
| Multiple languages or code switching | `solaria-1` |
| Live transcription required | `solaria-1` (only option) |
| Unknown language or 100+ language support | `solaria-1` |

### When to enable diarization

| Scenario | Enable |
|----------|--------|
| Multi-speaker call or meeting | Yes |
| Single speaker or known channels | No (use `channel` field instead) |
| Want speaker labels in output | Yes, set `diarization: true` |
| Know exact speaker count | Set `diarization_config.number_of_speakers` |
| Speaker count varies | Set `diarization_config.min_speakers` and `max_speakers` |

## Workflow

### Pre-recorded transcription (SDK)

1. **Initialize client**: Create `GladiaClient` with API key
2. **Transcribe in one call**: Use `.transcribe(audioPath, options)` for end-to-end flow
   - Pass local file path, URL, or binary data
   - SDK handles upload, job creation, polling automatically
3. **Or use individual steps** for fine control:
   - Upload: `.uploadFile(path)` → get `audio_url`
   - Create: `.createUntyped({audio_url, ...options})` → get `id`
   - Poll: `.get(id)` until `status: "done"`
4. **Access results**: Extract `result.transcription.full_transcript`, utterances, translations, etc.

### Pre-recorded transcription (API)

1. **Upload audio** (if local file): `POST /v2/upload` with multipart form-data
2. **Create job**: `POST /v2/pre-recorded` with JSON body containing `audio_url` and options
3. **Poll for result**: `GET /v2/pre-recorded/{id}` every 2–5 seconds until `status: "done"`
   - Or configure webhook at https://app.gladia.io/webhooks
   - Or set `callback_config` in request to receive POST when done
4. **Parse result**: Check `result.transcription`, `result.translation`, `result.summarization`, etc.

### Live transcription (SDK)

1. **Initialize session**: Call `.startSession(config)` with audio format (encoding, sample_rate, bit_depth, channels)
2. **Listen for events**: Register handlers for `message`, `started`, `ended`, `error`
3. **Send audio chunks**: Call `.sendAudio(chunk)` as audio arrives
4. **Read transcripts**: On `message` event, check `message.type === 'transcript'` and `is_final` flag
5. **Stop recording**: Call `.stopRecording()` to finalize and trigger post-processing
6. **Get final result**: Call `GET /v2/live/{id}` after session ends

### Live transcription (API)

1. **Init session**: `POST /v2/live` with audio config → get WebSocket `url` and `id`
2. **Connect WebSocket**: Open connection to returned `url`
3. **Send audio**: Send binary chunks or JSON with base64-encoded audio
4. **Read messages**: Parse incoming JSON messages; check `type` and `is_final` fields
5. **Stop**: Send `{"type": "stop_recording"}` or close with code 1000
6. **Retrieve final result**: `GET /v2/live/{id}` after WebSocket closes

### CLI transcription

1. **Set API key**: `gladia auth set YOUR_KEY` (or export `GLADIA_API_KEY`)
2. **Transcribe**: `gladia transcribe <file-or-url> [options]`
3. **Choose output**: Use `-o text|json|json-full|srt|vtt`
4. **Add features**: Use `--diarize`, `--language en,fr`, `--code-switching`, `--model solaria-3`
5. **Pipe output**: Use in scripts: `gladia transcribe audio.wav -o json | jq '.transcription'`

## Common gotchas

- **solaria-3 with multiple languages**: solaria-3 accepts only ONE language in `language_config.languages` — do not pass multiple or enable code switching. Use solaria-1 for multi-language.
- **Live transcription model**: Live only supports `solaria-1`. Attempting `solaria-3` will fail.
- **Audio format mismatch**: Specify `encoding`, `sample_rate`, `bit_depth`, and `channels` correctly in live init — mismatches cause garbled output.
- **3-hour session limit**: Live sessions terminate after 3 hours. Start a new session before hitting the limit.
- **Pre-recorded duration limit**: Max 135 minutes per request (4h15 for enterprise). Split longer files into ~60-minute chunks.
- **File size limit**: Max 1000 MB. Larger files are rejected.
- **Polling without timeout**: Always set a max retry count or timeout when polling — don't loop indefinitely.
- **API key in client code**: Never expose `x-gladia-key` in frontend code. Generate WebSocket URL on backend and pass only the URL to clients.
- **Diarization hints are not hard constraints**: `number_of_speakers`, `min_speakers`, `max_speakers` are hints; actual detection may differ.
- **Custom vocabulary not a spell-checker**: Custom vocabulary boosts recognition but doesn't force exact spelling — use `custom_spelling` for strict spelling control.
- **Callback URL must be public**: Webhook and callback endpoints must be reachable from Gladia's servers; localhost won't work.
- **Multi-channel billing**: Transcribing multi-channel audio is billed per channel — a 2-channel stream costs 2x the duration.
- **Deprecated endpoints**: Avoid `/v2/transcription/*` endpoints (deprecated); use `/v2/pre-recorded/*` instead.

## Verification checklist

Before submitting transcription work:

- [ ] API key is set and valid (test with a simple request)
- [ ] Audio file or URL is accessible and in a supported format (mp3, wav, m4a, ogg, flac, etc.)
- [ ] For pre-recorded: audio duration ≤ 135 minutes (or split into chunks)
- [ ] For live: session will not exceed 3 hours
- [ ] Model choice matches use case (solaria-3 for European audio + single language; solaria-1 for multi-language or live)
- [ ] Language config is correct (solaria-3: exactly one language; solaria-1: one or more, or auto-detect)
- [ ] Audio Intelligence features are enabled only if needed (diarization, translation, etc.)
- [ ] Callback/webhook URL is public and reachable (if using async notifications)
- [ ] Result parsing handles all expected fields (`transcription`, `translation`, `summarization`, etc.)
- [ ] Error handling covers job failures, network timeouts, and rate limits
- [ ] For live: WebSocket reconnection logic handles network drops gracefully

## Resources

**Comprehensive page listing**: https://docs.gladia.io/llms.txt

**Critical documentation pages**:
1. [Pre-recorded STT Quickstart](https://docs.gladia.io/chapters/pre-recorded-stt/quickstart) — Upload, create job, get result
2. [Live STT Quickstart](https://docs.gladia.io/chapters/live-stt/quickstart) — WebSocket streaming, real-time transcription
3. [API Reference](https://docs.gladia.io/api-reference) — Full endpoint documentation, request/response schemas

---

> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
---

> This file is auto-synced from https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
> Do not edit manually — changes will be overwritten by CI.
> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
