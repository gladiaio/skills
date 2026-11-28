---
name: gladia-documentation-auto
description: Comprehensive Gladia speech-to-text reference auto-synced from docs.gladia.io. Use as a general-purpose fallback when other specialized skills don't match, or when the user needs a broad overview of Gladia capabilities, endpoints, decision guidance, or workflows. Always prefer the official SDK; fall back to raw REST/WebSocket only when SDK cannot satisfy the requirement.
license: MIT
metadata:
  source: https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
  digest: sha256:c67b4859837170382b052da671cd7dc5b63556a85be1b1b31006aa947ed52f0d
  synced: "2026-11-28"
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
description: Use when transcribing audio or video to text in real-time or batch
  mode, extracting insights from speech (diarization, sentiment, translation,
  PII redaction), building voice AI agents, or integrating speech-to-text into
  applications. Reach for Gladia when you need accurate transcription with audio
  intelligence features across 100+ languages.
metadata:
  mintlify-proj: gladia
  version: "1.0"
---

# Gladia Skill

## Product summary

Gladia is a speech-to-text (STT) API that transcribes audio and video to text in real-time (live) or batch mode (pre-recorded). It offers two models—**Solaria-3** (highest accuracy on European business audio, pre-recorded only) and **Solaria-1** (100+ languages, live and pre-recorded, code switching)—plus audio intelligence features like speaker diarization, translation, sentiment analysis, PII redaction, and custom vocabulary. Authenticate with `x-gladia-key` header. Official SDKs available for Python (`pip install gladiaio-sdk`) and JavaScript/TypeScript (`npm install @gladiaio/sdk`). Primary docs: https://docs.gladia.io

## When to use

Reach for Gladia when:
- Transcribing pre-recorded audio or video files (batch/async mode)
- Streaming live audio in real-time (WebSocket-based)
- Building voice AI agents or conversational interfaces
- Extracting structured data from speech (who spoke, sentiment, entities, translations)
- Handling multilingual audio with automatic language detection or code switching
- Integrating with platforms like Pipecat, LiveKit, Vapi, Twilio, or meeting recorders
- Using the CLI for quick terminal-based transcription without app code
- Needing speaker diarization, PII redaction, or subtitle generation

## Quick reference

### Authentication
Pass API key in header: `x-gladia-key: YOUR_GLADIA_API_KEY`

### Models
| Model | Best for | Modes | Languages | Code switching | Accuracy focus |
|-------|----------|-------|-----------|-----------------|-----------------|
| **Solaria-3** | European real-world/business audio | Pre-recorded only | EN, FR, DE, ES, IT | No | Highest on noisy, conversational audio |
| **Solaria-1** | Global coverage, any domain | Pre-recorded + live | 100+ | Yes | Clean speech, live streaming, multilingual |

### Core endpoints
- `POST /v2/pre-recorded` — Create async transcription job
- `GET /v2/pre-recorded/{id}` — Poll job status
- `POST /v2/live` — Initiate live WebSocket session
- `GET /v2/live/{id}` — Get live session results
- `POST /v2/upload` — Upload audio file

### SDK quick start

**Pre-recorded (one call):**
```javascript
const gladia = new GladiaClient({ apiKey: "YOUR_KEY" });
const result = await gladia.preRecorded().transcribe("audio.mp3");
```

```python
gladia = GladiaClient(api_key="YOUR_KEY").prerecorded()
result = gladia.transcribe("audio.mp3")
```

**Live (stream audio):**
```javascript
const session = gladia.liveV2().startSession({
  encoding: "wav/pcm", sample_rate: 16000, bit_depth: 16, channels: 1
});
session.on("message", (msg) => { if (msg.type === "transcript") console.log(msg); });
session.sendAudio(chunk);
session.stopRecording();
```

### CLI commands
```bash
gladia auth set YOUR_KEY
gladia transcribe audio.mp3                    # text output
gladia transcribe audio.mp3 -o srt             # subtitles
gladia transcribe audio.mp3 --diarize          # speaker labels
gladia transcribe audio.mp3 --language en,fr   # constrain detection
gladia languages                               # list supported codes
```

### Audio intelligence features
- **Diarization** — Identify speakers (`diarization: true`)
- **Translation** — Translate to 100+ languages (`translation: true`, set `target_languages`)
- **PII Redaction** — Mask sensitive data (`pii_redaction: true`, set `entity_types`)
- **Sentiment Analysis** — Extract emotion (`sentiment_analysis: true`)
- **Summarization** — Generate summaries (`summarization: true`, set `type: "general" | "bullet_points" | "concise"`)
- **Named Entity Recognition** — Extract entities (`named_entity_recognition: true`)
- **Custom Vocabulary** — Boost accuracy for domain terms (`custom_vocabulary: true`, provide `vocabulary` list)
- **Subtitles** — Generate SRT/VTT (`subtitles: true`, set `formats`)

### Limits & concurrency
- **Live sessions:** 30 concurrent (default), max 3 hours per session
- **Pre-recorded jobs:** 25 parallel + 300 queued (default)
- **File size:** ~2 GB; split files >60 min for better results
- **Supported formats:** MP3, WAV, OGG, FLAC, M4A, WebM, and others
- **Channels:** Pre-recorded up to 2; live up to 8

## Decision guidance

### When to use Solaria-3 vs Solaria-1

| Scenario | Use Solaria-3 | Use Solaria-1 |
|----------|---------------|---------------|
| Pre-recorded audio only | ✓ | ✓ |
| Live/real-time streaming | ✗ | ✓ |
| European business/noisy audio | ✓ | — |
| 100+ languages needed | ✗ | ✓ |
| Code switching (mixed languages) | ✗ | ✓ |
| Clean, formal speech | — | ✓ |
| Single language constraint | ✓ | ✓ |

### When to use pre-recorded vs live

| Scenario | Pre-recorded | Live |
|----------|--------------|------|
| Batch processing files | ✓ | ✗ |
| Real-time streaming | ✗ | ✓ |
| Polling or webhooks acceptable | ✓ | — |
| Immediate partial results needed | — | ✓ |
| Audio intelligence (all features) | ✓ | ✓ |
| Solaria-3 model | ✓ | ✗ |

### When to use SDK vs API vs CLI

| Tool | Best for | Complexity |
|------|----------|-----------|
| **SDK** | App integration, error handling, retries | Low |
| **API** | Custom workflows, fine-grained control | Medium |
| **CLI** | Terminal scripts, CI/CD, one-off transcriptions | Low |

## Workflow

### Pre-recorded transcription (SDK)
1. **Initialize client:** Create `GladiaClient` with API key
2. **Choose model:** Decide Solaria-3 (European, pre-recorded) or Solaria-1 (global, live+pre-recorded)
3. **Prepare audio:** Local file path or remote URL
4. **Configure options:** Set language, diarization, translation, custom vocabulary, etc.
5. **Transcribe:** Call `transcribe()` (one-shot) or `create()` + `poll()` (manual control)
6. **Handle result:** Access `result.transcription.full_transcript` and audio intelligence fields
7. **Verify:** Check `status` is `"done"`, inspect error if `status` is `"error"`

### Live transcription (SDK)
1. **Initialize session:** Call `startSession()` with encoding, sample_rate, bit_depth, channels
2. **Configure messages:** Set `messages_config` to receive partials, finals, or post-processing events
3. **Register handlers:** Attach listeners for `"message"`, `"error"`, `"started"`, `"ended"`
4. **Send audio:** Stream chunks via `sendAudio()`; match configured encoding exactly
5. **Handle transcripts:** Check `message.type === "transcript"` and `is_final` flag
6. **Stop session:** Call `stopRecording()` to finalize and trigger post-processing
7. **Retrieve results:** Call `GET /v2/live/{id}` or listen for final callback/webhook

### CLI transcription
1. **Set API key:** `gladia auth set YOUR_KEY` (saved to `~/.gladia`)
2. **Choose output format:** `-o text` (default), `json`, `srt`, `vtt`
3. **Add options:** `--language en,fr`, `--diarize`, `--model solaria-3`
4. **Run:** `gladia transcribe file.mp3 -o srt`
5. **Pipe output:** Use with `jq`, shell scripts, or CI pipelines

## Common gotchas

- **Solaria-3 language constraint:** Set exactly one language in `language_config.languages` (e.g., `["fr"]`); do not pass multiple languages or enable code switching. Solaria-3 will fail if misconfigured.
- **Audio format mismatch:** Live sessions require `encoding`, `sample_rate`, `bit_depth`, and `channels` to match the audio you send. Mismatches cause silent failures or garbled transcripts.
- **Live session timeout:** Sessions cannot exceed 3 hours. For longer events, start a new session before the limit.
- **Polling vs webhooks:** Do not resubmit the same audio if a job already returned 200 or you received a `transcription.created` webhook. Wait on that job ID; resubmitting creates duplicate billable jobs.
- **Partial vs final transcripts:** Partial transcripts are provisional. Use `is_final: true` to identify complete utterances. Partials and finals for the same utterance share `data.id`; newer messages replace older ones.
- **Custom vocabulary is post-processing:** It does not retrain the model. Use for fixing mis-transcribed terms, not for improving base accuracy.
- **Multi-channel billing:** Transcribing stereo or multi-channel audio is billed per channel. A 10-minute stereo file costs 20 minutes of usage.
- **Concurrency limits:** Free/paid tiers have different limits (30 live, 25 pre-recorded parallel). Hitting limits returns HTTP 429; implement backoff and retry.
- **Endpointing tuning:** Small endpointing values (e.g., 0.05s) close utterances fast but may split sentences on hesitations. Larger values (e.g., 0.8s) wait longer, increasing latency but improving completeness.
- **Language detection latency:** Omitting `language_config.languages` triggers auto-detection, which adds latency. If you know the language, set it to skip this phase.

## Verification checklist

Before submitting transcription work:
- [ ] API key is set in header (`x-gladia-key`) or environment (`GLADIA_API_KEY`)
- [ ] Model choice matches use case (Solaria-3 for European pre-recorded, Solaria-1 for live or multilingual)
- [ ] Audio format, sample rate, bit depth, and channels are correctly specified and match the audio being sent
- [ ] Language configuration is valid (Solaria-3: single language; Solaria-1: multiple or auto-detect)
- [ ] For pre-recorded: job status is `"done"` before reading results; check for `"error"` status
- [ ] For live: `is_final: true` is checked before processing transcript text
- [ ] Callbacks or webhooks are configured if polling is not acceptable
- [ ] Audio file size is under ~2 GB; files >60 min are split for better results
- [ ] Concurrency limits are respected; implement backoff on HTTP 429
- [ ] Custom vocabulary or spelling is tested on sample audio before production use
- [ ] PII redaction entity types are appropriate for the use case (GDPR, HIPAA, etc.)

## Resources

**Comprehensive navigation:** https://docs.gladia.io/llms.txt

**Critical documentation pages:**
1. [Pre-recorded STT quickstart](https://docs.gladia.io/chapters/pre-recorded-stt/quickstart) — Upload, create jobs, poll results, webhooks
2. [Live STT quickstart](https://docs.gladia.io/chapters/live-stt/quickstart) — WebSocket sessions, audio streaming, partial transcripts
3. [Models](https://docs.gladia.io/chapters/introduction/models) — Solaria-3 vs Solaria-1 comparison and selection guide
4. [Audio Intelligence](https://docs.gladia.io/chapters/audio-intelligence/) — Diarization, translation, sentiment, PII redaction, custom vocabulary
5. [Concurrency & rate limits](https://docs.gladia.io/chapters/limits-and-specifications/concurrency) — Plan limits, 429 handling, capacity
6. [CLI reference](https://docs.gladia.io/chapters/developer-tools/gladia-cli) — Terminal transcription without app code
7. [Integrations](https://docs.gladia.io/chapters/integrations/) — Pipecat, LiveKit, Vapi, Twilio, meeting recorders

---

> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
---

> This file is auto-synced from https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
> Do not edit manually — changes will be overwritten by CI.
> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
