---
name: gladia-documentation-auto
description: Comprehensive Gladia speech-to-text reference auto-synced from docs.gladia.io. Use as a general-purpose fallback when other specialized skills don't match, or when the user needs a broad overview of Gladia capabilities, endpoints, decision guidance, or workflows. Always prefer the official SDK; fall back to raw REST/WebSocket only when SDK cannot satisfy the requirement.
license: MIT
metadata:
  source: https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
  digest: sha256:77515d1aa8f2573ed523e7e2fbbf82a91a57d761a959afad1345468b345032c6
  synced: "2026-10-10"
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
description: Use when transcribing audio or video files (pre-recorded),
  streaming live audio, extracting structured data from speech, or building
  voice AI applications. Reach for Gladia when you need speech-to-text with
  audio intelligence features like speaker diarization, translation, sentiment
  analysis, or PII redaction.
metadata:
  mintlify-proj: gladia
  version: "1.0"
---

# Gladia Skill

## Product summary

Gladia is a speech-to-text (STT) API that transcribes audio and video files asynchronously (pre-recorded) or in real-time (live streaming). It supports 100+ languages, speaker diarization, translation, sentiment analysis, PII redaction, and other audio intelligence features. Use the official SDKs (JavaScript/TypeScript, Python) for quick integration, or call the REST API directly. Authentication uses the `x-gladia-key` header. Primary docs: https://docs.gladia.io

**Key files and endpoints:**
- Pre-recorded: `POST /v2/pre-recorded` (create job), `GET /v2/pre-recorded/:id` (poll result)
- Live: `POST /v2/live` (init session), WebSocket connection for streaming
- Upload: `POST /v2/upload` (for local files)
- SDK: `npm install @gladiaio/sdk` or `pip install gladiaio-sdk`
- CLI: `gladia transcribe <file>` (terminal tool)

## When to use

Reach for Gladia when:
- **Pre-recorded transcription**: You have audio/video files (MP3, WAV, MP4, etc.) and need a transcript with optional speaker labels, translation, or sentiment analysis
- **Live streaming**: You're streaming audio in real-time (WebSocket) and need partial and final transcripts as they arrive
- **Audio intelligence**: You need to extract structured data — who spoke when (diarization), translate to multiple languages, detect sentiment, redact PII, or summarize
- **Multilingual**: Audio may contain multiple languages or code-switching; Solaria-1 handles 100+ languages
- **High-accuracy European audio**: Pre-recorded only; use Solaria-3 for English, French, German, Spanish, or Italian business/call center audio
- **Terminal workflows**: Quick one-off transcriptions from the CLI without writing code

Do not use Gladia for:
- Real-time transcription with Solaria-3 (pre-recorded only)
- Audio longer than 135 minutes in a single request (split into ~60-minute chunks)
- Live sessions exceeding 3 hours (start a new session)

## Quick reference

### Authentication
```bash
# Header-based (only method)
x-gladia-key: YOUR_API_KEY

# SDK initialization
const gladia = new GladiaClient({ apiKey: "YOUR_API_KEY" });
gladia_client = GladiaClient(api_key="YOUR_API_KEY")
```

### Pre-recorded transcription (SDK)
```javascript
// One-call transcription
const result = await gladia.preRecorded().transcribe("audio.mp3");

// With options
const result = await gladia.preRecorded().transcribe("audio.mp3", {
  model: "solaria-3",
  language_config: { languages: ["en"] },
  diarization: true,
  translation: true,
  translation_config: { target_languages: ["fr", "es"] }
});
```

```python
# One-call transcription
result = gladia_client.prerecorded().transcribe("audio.mp3")

# With options
result = gladia_client.prerecorded().transcribe("audio.mp3", {
    "model": "solaria-3",
    "language_config": {"languages": ["en"]},
    "diarization": True,
    "translation": True,
    "translation_config": {"target_languages": ["fr", "es"]}
})
```

### Live transcription (SDK)
```javascript
const session = gladia.liveV2().startSession({
  encoding: "wav/pcm",
  sample_rate: 16000,
  bit_depth: 16,
  channels: 1,
  language_config: { languages: ["en"] }
});

session.on("message", (msg) => {
  if (msg.type === "transcript" && msg.data.is_final) {
    console.log(msg.data.utterance.text);
  }
});

session.sendAudio(audioChunk);
session.stopRecording();
```

```python
session = gladia_client.live_v2().start_session(
    LiveV2InitRequest(
        encoding="wav/pcm",
        sample_rate=16000,
        bit_depth=16,
        channels=1,
        language_config=LiveV2LanguageConfig(languages=["en"])
    )
)

@session.on("message")
def on_message(msg):
    if getattr(msg, "type", None) == "transcript":
        if msg.data.is_final:
            print(msg.data.utterance.text)

session.send_audio(audio_chunk)
session.stop_recording()
```

### CLI
```bash
gladia auth set YOUR_API_KEY
gladia transcribe meeting.wav                    # text output
gladia transcribe podcast.mp3 -o json            # JSON output
gladia transcribe call.wav --diarize -o srt      # subtitles with speakers
gladia transcribe mixed.mp3 --code-switching --language en,fr
gladia transcribe audio.mp3 --model solaria-3 --language en
```

### Supported audio formats
MP3, WAV, FLAC, OGG, Opus, AAC, M4A, AC3, EAC3, MP2

### Supported video formats
MP4, MOV, AVI, FLV, MKV, 3GP, WMV, WebM

### Models
| Model | Best for | Modes | Languages | Code switching |
|-------|----------|-------|-----------|-----------------|
| **solaria-3** | European real-world audio (calls, meetings) | Pre-recorded only | EN, FR, DE, ES, IT | No |
| **solaria-1** (default) | Global coverage, any domain | Pre-recorded + live | 100+ | Yes |

### Audio Intelligence features
- **Speaker diarization**: Identify who spoke when
- **Translation**: Translate to 100+ target languages
- **Sentiment analysis**: Extract sentiment and up to 25 emotions
- **PII redaction**: Detect and mask sensitive data (GDPR, HIPAA, etc.)
- **Named entity recognition**: Extract people, organizations, dates
- **Summarization**: Generate general, bullet-point, or concise summaries
- **Custom vocabulary**: Boost accuracy for domain-specific terms
- **Custom spelling**: Fix misspellings in output
- **Subtitles**: Generate SRT or VTT files
- **Sentences**: Group words into sentences with timing

## Decision guidance

### When to use Solaria-3 vs Solaria-1

| Condition | Use Solaria-3 | Use Solaria-1 |
|-----------|---------------|---------------|
| Pre-recorded audio | ✓ | ✓ |
| Live/streaming audio | ✗ | ✓ |
| Language: EN, FR, DE, ES, IT | ✓ (better accuracy) | ✓ |
| Language: other (100+ languages) | ✗ | ✓ |
| Code switching (mixed languages) | ✗ | ✓ |
| Noisy, conversational audio | ✓ | ✓ (good) |
| Clean, formal speech | ✓ | ✓ |

### When to poll vs webhooks vs callbacks

| Approach | Use when | Pros | Cons |
|----------|----------|------|------|
| **Polling** (`GET /v2/pre-recorded/:id`) | Testing, small batch, low volume | Simple, no infrastructure | Wastes requests, blocks threads |
| **Webhooks** (dashboard config) | Production, high volume, event-driven | Scalable, no polling | Requires public endpoint, retry logic |
| **Callbacks** (per-job config) | Per-job flexibility, mixed patterns | Flexible, no dashboard setup | Same as webhooks |

### When to use custom vocabulary vs custom spelling

| Issue | Use custom vocabulary | Use custom spelling |
|-------|----------------------|---------------------|
| Word is garbled or phonetically wrong | ✓ | ✗ |
| Word is recognized but misspelled | ✗ | ✓ |
| Multiple phonetic variants | ✓ | ✗ |
| Brand names, product names | ✓ | ✓ (if misspelled) |
| Acronyms | ✓ | ✓ |

## Workflow

### Pre-recorded transcription (typical task)

1. **Prepare audio**: Ensure file is under 135 minutes and 1000 MB. Supported formats: MP3, WAV, FLAC, OGG, Opus, AAC, M4A, AC3, EAC3, MP2, or video (MP4, MOV, AVI, FLV, MKV, 3GP, WMV).

2. **Choose model and language**:
   - If audio is EN/FR/DE/ES/IT and pre-recorded: consider `solaria-3` for higher accuracy
   - If audio is other language or live: use `solaria-1` (default)
   - If language is known: set `language_config.languages: ["en"]` to skip detection
   - If audio is multilingual: enable `code_switching: true` (Solaria-1 only)

3. **Decide on audio intelligence features**:
   - Speaker diarization: `diarization: true` (set `min_speakers`/`max_speakers` for hints)
   - Translation: `translation: true` + `translation_config.target_languages: ["fr", "es"]`
   - Sentiment: `sentiment_analysis: true`
   - PII redaction: `pii_redaction: true` + specify entity types
   - Summarization: `summarization: true` + choose type (general, bullet_points, concise)

4. **Upload and create job** (SDK handles this in one call):
   ```javascript
   const result = await gladia.preRecorded().transcribe("audio.mp3", {
     model: "solaria-3",
     language_config: { languages: ["en"] },
     diarization: true
   });
   ```

5. **Retrieve result**: SDK polls automatically. For raw API, poll `GET /v2/pre-recorded/:id` until `status: "done"`, or configure webhooks/callbacks.

6. **Parse output**: Access `result.transcription.full_transcript` for text, `result.transcription.utterances` for timing/speaker, `result.diarization` for speaker labels, `result.translation` for translations.

### Live transcription (typical task)

1. **Prepare audio source**: Ensure encoding, sample rate, bit depth, and channels match what you'll send. Common: `wav/pcm`, 16000 Hz, 16-bit, 1 channel.

2. **Initialize session** (backend):
   ```javascript
   const session = gladia.liveV2().startSession({
     encoding: "wav/pcm",
     sample_rate: 16000,
     bit_depth: 16,
     channels: 1,
     language_config: { languages: ["en"] },
     messages_config: { receive_partial_transcripts: true }
   });
   ```

3. **Connect client to WebSocket**: Pass the session URL to frontend/client (keep API key on backend).

4. **Send audio chunks**: Stream audio in real-time.

5. **Handle messages**: Listen for `transcript` messages; use `is_final` to distinguish partial vs final.

6. **Stop session**: Call `session.stopRecording()` when done. Post-processing runs automatically.

7. **Retrieve final results**: Call `GET /v2/live/:id` or listen for final callback/webhook.

### CLI one-off transcription

```bash
gladia transcribe meeting.wav --diarize -o json | jq '.transcription.full_transcript'
```

## Common gotchas

- **Solaria-3 with multiple languages**: Solaria-3 does not support code switching. Set exactly one language in `language_config.languages` (e.g., `["fr"]`). Passing multiple languages or enabling code switching will fail.

- **Resubmitting after 200 response**: If `POST /v2/pre-recorded` returns HTTP 200 or you receive `transcription.created` webhook, the job is accepted. Do not resubmit the same audio — it creates a duplicate billable job. Store the job ID and wait for completion.

- **Language detection overhead**: If you don't set a language, Gladia auto-detects on the first utterance. If your audio starts with silence, music, or a different language, detection may fail for the whole file. Always set language if known.

- **Audio longer than 135 minutes**: Split into ~60-minute chunks. Gladia will reject files over 135 minutes.

- **Live session exceeding 3 hours**: A single WebSocket session cannot exceed 3 hours. Start a new session before the limit.

- **Confusing partial vs final transcripts**: Partial transcripts are provisional and may change. Use `is_final: true` to identify final transcripts. Events for the same utterance share `data.id` — a newer final replaces the displayed partial.

- **Custom vocabulary not working**: Transcribe without custom vocabulary first, note the mis-transcribed terms, then add them. Test again and refine intensity (0.4–0.6 is typical). Collect variants from real transcripts, not guesses.

- **Diarization vs multi-channel confusion**: If each speaker is on a separate audio channel (e.g., stereo with left=speaker1, right=speaker2), use the `channel` field in utterances — diarization is not needed. If all speakers share one channel, enable diarization.

- **Callback URL not receiving events**: Ensure your callback URL is publicly accessible, returns 2xx status, and is correctly configured in `callback_config.url`. Gladia retries failed callbacks; check logs for delivery attempts.

- **429 (rate limit) errors**: You've hit concurrency limits. Back off and retry with exponential jitter. Check your plan's concurrency limit.

- **Deprecated transcription endpoint**: The old `POST /v2/transcription` is deprecated. Use `POST /v2/pre-recorded` instead.

## Verification checklist

Before submitting transcription work:

- [ ] Audio file is under 135 minutes and 1000 MB (or split into chunks)
- [ ] Audio format is supported (MP3, WAV, FLAC, OGG, Opus, AAC, M4A, AC3, EAC3, MP2, or video)
- [ ] If using Solaria-3: language is one of EN, FR, DE, ES, IT; code switching is disabled
- [ ] If using Solaria-1: language is set if known, or auto-detection is acceptable
- [ ] Audio intelligence features are enabled only if needed (diarization, translation, sentiment, etc.)
- [ ] Callback/webhook URL is publicly accessible and returns 2xx (if using callbacks)
- [ ] Job ID is stored before polling or waiting for webhooks
- [ ] Not resubmitting the same audio after receiving HTTP 200 or `transcription.created`
- [ ] Custom vocabulary entries are collected from real transcripts, not guesses
- [ ] For live: encoding, sample rate, bit depth, and channels match the audio being sent
- [ ] For live: session will not exceed 3 hours; plan for new session if needed

## Resources

**Comprehensive page listing**: https://docs.gladia.io/llms.txt

**Critical documentation pages**:
1. [Pre-recorded quickstart](https://docs.gladia.io/chapters/pre-recorded-stt/quickstart) — Upload, create jobs, poll results, webhooks, callbacks
2. [Live quickstart](https://docs.gladia.io/chapters/live-stt/quickstart) — WebSocket streaming, partial transcripts, session lifecycle
3. [Models](https://docs.gladia.io/chapters/introduction/models) — Solaria-3 vs Solaria-1 comparison and when to use each

---

> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
---

> This file is auto-synced from https://docs.gladia.io/.well-known/agent-skills/gladia/skill.md
> Do not edit manually — changes will be overwritten by CI.
> For additional documentation and navigation, see: https://docs.gladia.io/llms.txt
