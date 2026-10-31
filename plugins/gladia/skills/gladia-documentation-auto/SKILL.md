---
name: gladia-documentation-auto
description: Comprehensive Gladia speech-to-text reference auto-synced from docs.gladia.io. Use as a general-purpose fallback when other specialized skills don't match, or when the user needs a broad overview of Gladia capabilities, endpoints, decision guidance, or workflows. Always prefer the official SDK; fall back to raw REST/WebSocket only when SDK cannot satisfy the requirement.
license: MIT
metadata:
  source: https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
  digest: sha256:bd394d9233bb03edd70ca34091551eb3bd0e90c1c82cfa93b0b0e8628ca62419
  synced: "2026-10-31"
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
  audio or video files asynchronously, streaming live audio for real-time
  transcription, extracting insights from audio (speaker identification,
  translation, sentiment analysis), or integrating transcription into voice
  agents, meeting recorders, or multilingual applications.
metadata:
  mintlify-proj: gladia
  version: "1.0"
---

# Gladia Speech-to-Text API

## Product summary

Gladia is a speech-to-text API that transcribes audio and video in real-time (live) or asynchronously (pre-recorded). It supports 100+ languages, offers two models (Solaria-3 for European accuracy, Solaria-1 for global coverage), and includes audio intelligence features like speaker diarization, translation, sentiment analysis, and entity recognition. Agents use Gladia to build transcription features, extract structured data from audio, and power voice applications.

**Key files and commands:**
- SDK: `npm install @gladiaio/sdk` (JavaScript) or `pip install gladiaio-sdk` (Python)
- CLI: `gladia transcribe <file>` for terminal-based transcription
- API authentication: Pass `x-gladia-key` header with your API key
- Primary docs: https://docs.gladia.io

## When to use

Reach for this skill when:
- Building pre-recorded transcription (upload audio, get transcript asynchronously)
- Implementing live/real-time transcription (WebSocket streaming)
- Extracting speaker information, translations, or sentiment from audio
- Building voice agents, meeting recorders, or call analysis tools
- Choosing between models based on language coverage or audio quality
- Handling multilingual audio with code switching
- Integrating with platforms like Pipecat, Livekit, Vapi, Twilio, or n8n
- Testing transcription quickly in the browser (playground)
- Transcribing from the terminal without writing code (CLI)

## Quick reference

### Authentication
```bash
# Header-based (only method)
x-gladia-key: YOUR_GLADIA_API_KEY
```

### SDK installation
```bash
npm install @gladiaio/sdk          # JavaScript/TypeScript
pip install gladiaio-sdk            # Python
```

### Pre-recorded transcription (one call)
```javascript
const gladia = new GladiaClient({ apiKey: "YOUR_KEY" });
const result = await gladia.preRecorded().transcribe("audio.mp3");
```

```python
gladia = GladiaClient(api_key="YOUR_KEY").prerecorded()
result = gladia.transcribe("audio.mp3")
```

### Live transcription (WebSocket)
```javascript
const session = gladia.liveV2().startSession({
  encoding: "wav/pcm",
  sample_rate: 16000,
  bit_depth: 16,
  channels: 1,
});
session.on("message", (msg) => { /* handle transcript */ });
session.sendAudio(audioChunk);
session.stopRecording();
```

### CLI transcription
```bash
gladia auth set YOUR_KEY
gladia transcribe meeting.wav                    # text output
gladia transcribe meeting.wav -o json            # JSON output
gladia transcribe meeting.wav --diarize          # speaker labels
gladia transcribe meeting.wav --language en,fr   # constrain detection
```

### API endpoints
| Endpoint | Purpose |
|----------|---------|
| `POST /v2/pre-recorded` | Create async transcription job |
| `GET /v2/pre-recorded/:id` | Poll job status and results |
| `POST /v2/live` | Initialize live session (returns WebSocket URL) |
| `GET /v2/live/:id` | Get final live session results |
| `POST /v2/upload` | Upload audio file |

### Audio intelligence features
| Feature | Pre-recorded | Live | Config key |
|---------|--------------|------|-----------|
| Speaker diarization | ✓ | ✓ | `diarization` |
| Translation | ✓ | ✓ | `translation` |
| Sentiment analysis | ✓ | ✓ | `sentiment_analysis` |
| Named entity recognition | ✓ | ✓ | `named_entity_recognition` |
| Summarization | ✓ | ✓ | `summarization` |
| Custom vocabulary | ✓ | ✓ | `custom_vocabulary` |
| Custom spelling | ✓ | ✓ | `custom_spelling` |
| Subtitles (SRT/VTT) | ✓ | — | `subtitles` |
| PII redaction | ✓ | — | `pii_redaction` |

## Decision guidance

### When to use Solaria-3 vs Solaria-1

| Criterion | Solaria-3 | Solaria-1 |
|-----------|-----------|-----------|
| **Best for** | European real-world/business audio | Global coverage, any domain |
| **Modes** | Pre-recorded only | Pre-recorded + live |
| **Languages** | EN, FR, DE, ES, IT (5 only) | 100+ languages |
| **Code switching** | No | Yes |
| **Accuracy focus** | Noisy, conversational audio | Clean speech, formal |
| **Use case** | Calls, meetings, field recordings | Voice agents, live captions, multilingual |

**Decision:** Use Solaria-3 if audio is pre-recorded, language is one of the five covered, and accuracy on real production audio matters. Otherwise use Solaria-1 (default).

### When to use pre-recorded vs live

| Aspect | Pre-recorded | Live |
|--------|--------------|------|
| **Latency** | Asynchronous (seconds to minutes) | Real-time (100-500ms) |
| **Use case** | Batch processing, files, URLs | Streaming, voice calls, agents |
| **Result retrieval** | Poll, webhook, or callback | WebSocket messages |
| **Session limit** | No limit | 3 hours max |
| **Model** | Solaria-1 or Solaria-3 | Solaria-1 only |

**Decision:** Use pre-recorded for files and batch jobs. Use live for streaming, voice agents, and real-time applications.

### When to use polling vs webhooks vs callbacks

| Method | Best for | Implementation |
|--------|----------|-----------------|
| **Polling** | Testing, short jobs | `gladia.preRecorded().poll(jobId)` |
| **Webhooks** | Production, event-driven | Configure at https://app.gladia.io/webhooks |
| **Callbacks** | Per-job notification | Set `callback` and `callback_config` in request |

**Decision:** Use polling for quick tests. Use webhooks for production (configured once). Use callbacks for per-job control.

## Workflow

### Pre-recorded transcription (typical flow)

1. **Prepare audio:** Have a local file or remote URL ready.
2. **Choose model:** Decide between Solaria-3 (European accuracy) or Solaria-1 (global, default).
3. **Set language:** Specify `language_config.languages` (e.g., `["en"]` for Solaria-3, or `["en", "fr"]` for Solaria-1).
4. **Add features:** Enable diarization, translation, sentiment, or other audio intelligence if needed.
5. **Create job:** Call `POST /v2/pre-recorded` with audio URL and config, or use SDK's `transcribe()` method.
6. **Retrieve result:** Poll `GET /v2/pre-recorded/:id`, use webhooks, or set a callback URL.
7. **Parse output:** Extract `transcription.full_transcript`, utterances, or audio intelligence results.
8. **Handle errors:** Check job status; if error, inspect error details before retrying (fix input first, not just resubmit).

### Live transcription (typical flow)

1. **Initialize session:** Call `POST /v2/live` with audio format (encoding, sample_rate, bit_depth, channels).
2. **Get WebSocket URL:** Response contains `url` and `id`; generate on backend, pass to client.
3. **Connect WebSocket:** Client connects to the URL (no API key needed in URL).
4. **Send audio chunks:** Stream audio in the format specified during init.
5. **Handle messages:** Listen for `transcript` messages; check `is_final` to distinguish partial vs final.
6. **Stop recording:** Send `stop_recording` message or close WebSocket with code 1000.
7. **Get final results:** Retrieve via `GET /v2/live/:id` or callback URL.

### CLI transcription (typical flow)

1. **Set API key:** `gladia auth set YOUR_KEY` or `export GLADIA_API_KEY=YOUR_KEY`.
2. **Run transcribe:** `gladia transcribe <file-or-url>`.
3. **Choose output:** Add `-o json`, `-o srt`, `-o vtt` as needed.
4. **Add features:** Use `--diarize`, `--language en,fr`, `--code-switching`, `--model solaria-3`.
5. **Pipe or save:** Redirect stdout to file or pipe to `jq` for JSON processing.

## Common gotchas

- **Solaria-3 language constraint:** With Solaria-3, pass exactly one language in `language_config.languages` (e.g., `["fr"]`). Do not pass multiple languages or enable code switching; the model will reject it.
- **Live model limitation:** Live transcription only supports Solaria-1. Solaria-3 is pre-recorded only.
- **Audio format mismatch:** Encoding, sample_rate, bit_depth, and channels in the init request must match the actual audio being sent. Mismatches cause silent failures or garbled transcripts.
- **Don't resubmit after 200:** If `POST /v2/pre-recorded` returns 200 or you receive a `transcription.created` webhook, the job is accepted. Do not resubmit the same audio while it's queued or processing; wait on that job ID.
- **429 errors:** HTTP 429 means either concurrency exhausted or rate limiting. Reduce parallel jobs for concurrency; slow request rate for rate limiting. Use bounded retries with backoff.
- **Transient vs permanent errors:** Timeouts and temporary 5xx errors can be retried with backoff. Invalid URLs, corrupt files, or bad parameters should be fixed first; resubmitting the same payload will fail again and create a new billable job.
- **Partial transcripts accuracy:** Partial transcripts use a faster, smaller model than finals, so accuracy is lower. Disable `receive_partial_transcripts` if you only need final results.
- **Session timeout:** Live sessions cannot exceed 3 hours. For longer events, start a new session before the limit.
- **Multi-channel billing:** Transcribing multi-channel audio (e.g., 2 channels) is billed as 2× the duration. Plan accordingly.
- **Callback URL must be reachable:** If using callbacks, ensure your endpoint is publicly accessible and returns 2xx within a reasonable time.

## Verification checklist

Before submitting work with Gladia:

- [ ] API key is set (environment variable, `~/.gladia`, or per-command flag).
- [ ] Audio format (encoding, sample_rate, bit_depth, channels) matches the actual audio being sent.
- [ ] Model choice is correct: Solaria-3 for European accuracy (pre-recorded only), Solaria-1 for live or global coverage.
- [ ] Language configuration is valid: Solaria-3 accepts one language; Solaria-1 accepts multiple or code switching.
- [ ] Audio file or URL is accessible and in a supported format (MP3, WAV, FLAC, OGG, M4A, etc.).
- [ ] Job ID is captured from the response and used for polling or callbacks (do not resubmit).
- [ ] Webhook or callback URL is configured and publicly reachable (if using async notification).
- [ ] Audio intelligence features (diarization, translation, etc.) are enabled only if needed (they add latency and cost).
- [ ] Error handling is in place: check job status, inspect error details, and fix input before retrying.
- [ ] For live sessions: WebSocket URL is generated on backend and passed securely to client; API key is not exposed in the URL.

## Resources

- **Comprehensive page navigation:** https://docs.gladia.io/llms.txt
- **Pre-recorded transcription quickstart:** https://docs.gladia.io/chapters/pre-recorded-stt/quickstart
- **Live transcription quickstart:** https://docs.gladia.io/chapters/live-stt/quickstart
- **Model comparison and selection:** https://docs.gladia.io/chapters/introduction/models
- **Audio intelligence features:** https://docs.gladia.io/chapters/audio-intelligence/index
- **CLI reference:** https://docs.gladia.io/chapters/developer-tools/gladia-cli
- **API reference:** https://docs.gladia.io/api-reference/
- **SDK samples:** https://github.com/gladiaio/gladia-samples

---

> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
---

> This file is auto-synced from https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
> Do not edit manually — changes will be overwritten by CI.
> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
