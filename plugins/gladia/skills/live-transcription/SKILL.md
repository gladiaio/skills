---
name: live-transcription
description: Real-time speech-to-text streaming with Gladia via WebSocket. Use when the user needs live transcription, builds a voice agent, meeting recorder, call center integration, live subtitles, or any application streaming audio for low-latency partial and final transcripts. Always prefer the official SDK; fall back to raw WebSocket/REST only when SDK cannot satisfy the requirement.
license: MIT
---

# Live Transcription

Gladia's live API transcribes audio in real-time over WebSocket.

> **SDK-first**: always use the official SDK — see [sdk-integration](../sdk-integration/SKILL.md) for policy, setup, and fallback criteria.

## When to Use

- User needs real-time / live transcription of an audio stream (microphone, telephony, broadcast)
- Building a voice agent, meeting recorder, call center integration, or live subtitle system
- Application requires low-latency partial and final transcripts as audio is captured
- WebSocket-based streaming with real-time intelligence (translation, sentiment, NER)

**When NOT to use:** If the user has a pre-existing audio/video file or URL to transcribe after the fact, use the [pre-recorded-transcription skill](../pre-recorded-transcription/SKILL.md) instead. Pre-recorded supports additional features like speaker diarization and PII redaction that are unavailable in live mode.

## References

Consult these resources as needed:

- ./references/recommended-params.md -- Best parameter configurations by use case
- ./references/websocket-events.md -- Complete WebSocket message type reference
- ../audio-intelligence/SKILL.md -- Audio intelligence feature availability and configs for live and pre-recorded
- ../audio-intelligence/references/live-audio-intelligence.md -- Detailed live-mode feature configs and WebSocket response structures
- ../sdk-integration/SKILL.md -- SDK setup, client initialization, error handling, retry/timeout config, and SDK vs raw API decision guide
- ../sdk-integration/references/sdk-versions.md -- Current SDK versions (auto-synced by CI)
- ../troubleshooting/SKILL.md -- Common errors, gotchas, and verification checklist

## Session Lifecycle (handled automatically by the SDK)

The SDK's `startSession()` manages this entire lifecycle — init, WebSocket connect, reconnection, and cleanup. The diagram below is for reference; you do not need to implement these steps manually.

```
POST /v2/live (init)           ← SDK: startSession()
       │
       ▼
  Get session id + WebSocket URL
       │
       ▼
  Connect WebSocket            ← SDK: automatic
       │
       ▼
  Stream audio chunks (binary) ← SDK: sendAudio()
       │
       ▼
  Receive transcript events    ← SDK: session.on("message")
       │
       ▼
  Send stop_recording          ← SDK: stopRecording()
       │
       ▼
  Receive post-processing results
       │
       ▼
  Session ends (WebSocket closes)
       │
       ▼
  GET /v2/live/:id             ← SDK: client.liveV2().get(id)
```

## Quick Start

For SDK installation and client initialization, see the [sdk-integration skill](../sdk-integration/SKILL.md).

### JavaScript/TypeScript

```typescript
const session = client.liveV2().startSession({
  model: "solaria-1",
  encoding: "wav/pcm",
  sample_rate: 16000,
  bit_depth: 16,
  channels: 1,
  language_config: { languages: ["en"] },
  messages_config: { receive_partial_transcripts: true },
});

session.on("message", (msg) => {
  if (msg.type === "transcript") {
    const prefix = msg.data.is_final ? "FINAL" : "PARTIAL";
    console.log(`${prefix} | ${msg.data.utterance.text}`);
  }
});

session.once("started", () => {
  console.log(`Session ${session.sessionId} started`);
});

session.on("error", (err) => console.error("Error:", err));

// Stream audio
session.sendAudio(audioBuffer);

// When done
session.stopRecording();

await new Promise((resolve) => session.once("ended", resolve));
```

### Python (sync)

```python
from gladiaio_sdk import (
    LiveV2InitRequest,
    LiveV2LanguageConfig,
    LiveV2MessagesConfig,
    LiveV2WebSocketMessage,
)

live_client = client.live()

session = live_client.start_session(
    LiveV2InitRequest(
        model="solaria-1",
        encoding="wav/pcm",
        sample_rate=16000,
        bit_depth=16,
        channels=1,
        language_config=LiveV2LanguageConfig(languages=["en"]),
        messages_config=LiveV2MessagesConfig(receive_partial_transcripts=True),
    )
)

@session.on("message")
def on_message(message: LiveV2WebSocketMessage):
    if message.type == "transcript":
        prefix = "F" if message.data.is_final else "P"
        print(f"{prefix} | {message.data.utterance.text.strip()}")

@session.on("error")
def on_error(error: Exception):
    print(f"Error: {error}")

session.send_audio(audio_bytes)
session.stop_recording()
```

## Session Configuration

These options are passed to `startSession()` (JS) / `start_session()` (Python). For client-level config (retry, timeouts, WebSocket settings), see the [sdk-integration skill](../sdk-integration/SKILL.md#configuration-options).

```typescript
{
  // Model
  model: 'solaria-1',

  // Audio format (MUST match your audio stream exactly)
  encoding: 'wav/pcm',        // wav/pcm | wav/alaw | wav/ulaw
  sample_rate: 16000,          // Hz (8000, 16000, 44100, 48000)
  bit_depth: 16,               // 8 or 16
  channels: 1,                 // 1 (mono) or 2 (stereo; billed as 2x duration)

  // Language
  language_config: {
    languages: ['en'],         // 1-5 expected languages
    code_switching: false,     // Detect language changes mid-speech
  },

  // Pre-processing
  pre_processing: {
    audio_enhancer: true,      // Noise reduction
    speech_threshold: 0.5,     // VAD threshold (0.0-1.0)
  },

  // Real-time features
  realtime_processing: {
    custom_vocabulary: true,
    custom_vocabulary_config: { vocabulary: [...] },
    translation: true,
    translation_config: { target_languages: ['fr'] },
    named_entity_recognition: true,
    sentiment_analysis: true,
  },

  // Post-processing (runs after stop_recording)
  post_processing: {
    summarization: true,
    summarization_config: { type: 'bullet_points' },
    chapterization: true,
  },

  // Message filtering
  messages_config: {
    receive_partial_transcripts: true,   // Low-latency partial results
    receive_speech_events: true,         // speech_start / speech_end
    receive_pre_processing_events: false,
  },

  // Callbacks
  callback_url: 'https://your-server.com/live-events',
}
```

Python uses typed request objects for the same configuration — see the [Python SDK reference](../sdk-integration/references/python.md) for all available types:

```python
from gladiaio_sdk import (
    LiveV2InitRequest, LiveV2LanguageConfig,
    LiveV2MessagesConfig, LiveV2PreProcessing,
    LiveV2RealtimeProcessing, LiveV2PostProcessing,
)

session = live_client.start_session(
    LiveV2InitRequest(
        model="solaria-1",
        encoding="wav/pcm",
        sample_rate=16000,
        bit_depth=16,
        channels=1,
        language_config=LiveV2LanguageConfig(languages=["en"], code_switching=False),
        pre_processing=LiveV2PreProcessing(audio_enhancer=True, speech_threshold=0.5),
        realtime_processing=LiveV2RealtimeProcessing(
            custom_vocabulary=True,
            translation=True,
            sentiment_analysis=True,
        ),
        post_processing=LiveV2PostProcessing(summarization=True, chapterization=True),
        messages_config=LiveV2MessagesConfig(
            receive_partial_transcripts=True,
            receive_speech_events=True,
        ),
    )
)
```

## Key Tuning Parameters

### `endpointing` (seconds)

How long a silence must last before Gladia closes the current utterance and emits a final transcript.

- **Default:** `0.05` — **Range:** `0.01` to `10`

This is the core **latency vs. accuracy tradeoff** in live transcription. Lowering it produces faster final results but risks splitting sentences mid-hesitation. Raising it produces cleaner, more complete segments at the cost of higher latency. There is no universal right value — it depends on your audio environment, speaker patterns, and downstream UX requirements.

| Use case         | Recommended value | Why                                    |
| ---------------- | ----------------- | -------------------------------------- |
| Voice agent      | `0.05` - `0.1`    | Fast turn detection                    |
| Call center      | `0.1` - `0.3`     | Snappy but tolerates telephony gaps    |
| Live subtitles   | `0.2` - `0.4`     | Readable chunks without too much delay |
| Meeting recorder | `0.3` - `0.5`     | Complete sentences before closing      |

> **Unsure what value to use?** Endpointing tuning is highly dependent on your specific audio conditions and use case. Reach out to the [Gladia team](https://gladia.io/contact) — they can help you find the optimal configuration.

### `maximum_duration_without_endpointing` (seconds)

A hard cap on how long Gladia keeps an utterance open if no silence is ever detected. When this limit is reached, the utterance is force-closed regardless.

- **Default:** `5` — **Range:** `5` to `60`

This is a safety valve against unbounded utterances caused by constant background noise, a speaker who never pauses, or long monologues. Without it, a noisy environment could produce a single never-ending utterance that blocks downstream processing.

- Voice agents: keep low (~`5`) for fast turn-taking
- Meeting recorders: raise to `15`-`30` to handle long speaking turns without premature cuts

### `speech_threshold` (0.0 - 1.0)

Located in `pre_processing.speech_threshold`. Sets the VAD (Voice Activity Detection) confidence threshold — how certain the system must be that incoming audio contains speech before it opens a new utterance.

- **Default:** `0.5` — **Range:** `0.0` to `1.0`
- Low value (e.g. `0.3`) → more sensitive, may trigger on background noise or breathing
- High value (e.g. `0.7`) → more conservative, ignores faint or distant speech

Raise this when you're in a noisy environment and getting false transcript segments from background sounds. Lower it only if the speaker is faint or far from the microphone.

> **Note:** `speech_threshold` does not have a dedicated page in the official Gladia docs. Verify its current behavior and supported range with the Gladia team before relying on it in production.

## Audio Streaming

Use `session.sendAudio(chunk)` (JS) / `session.send_audio(chunk)` (Python) to stream audio data. The SDK sends each chunk as a binary WebSocket frame.

- Chunk size: 100ms of audio per frame (recommended)
- Send continuously — do not batch large chunks
- Audio format MUST match the `encoding`, `sample_rate`, `bit_depth`, and `channels` in session config

## Stopping and Reconnection

### Normal stop

```typescript
session.stopRecording(); // Triggers post-processing, then session ends
```

```python
session.stop_recording()  # Triggers post-processing, then session ends
```

### Force end (skip post-processing)

```typescript
session.endSession(); // Immediately closes, no post-processing
```

```python
session.end_session()  # Immediately closes, no post-processing
```

### Reconnection

The SDK handles reconnection automatically with configurable `wsRetry` — no manual intervention needed. If using raw WebSocket, reconnect to the **same URL** to resume. Do NOT create a new session.

## Limits

| Constraint           | Value                        |
| -------------------- | ---------------------------- |
| Max session duration | 3 hours                      |
| Supported encodings  | wav/pcm, wav/alaw, wav/ulaw  |
| Concurrency (paid)   | 30 concurrent sessions       |
| Concurrency (free)   | 1 concurrent session         |
| Billing              | Per-second of streamed audio |
| Multi-channel        | Billed as N x duration       |

## Retrieving Final Results

After a session ends, you can retrieve the complete results via the SDK:

```typescript
const results = await client.liveV2().get(sessionId);
```

```python
results = client.live().get(session_id)
```

This returns the same structure as polling a pre-recorded job, including all post-processing results.

## Common Mistakes

- **Audio format mismatch**: the `encoding`, `sample_rate`, `bit_depth`, and `channels` in session config MUST match the actual audio stream exactly.
- **Forgetting to stop recording**: leaving a session open without `stopRecording()` keeps it hanging.
- **Expecting diarization**: speaker diarization is pre-recorded only. For live multi-speaker, use multi-channel audio with one speaker per channel.
- **PII redaction in live mode**: `pii_redaction: true` is silently ignored in live sessions. Implement client-side redaction if needed.
- **Partial transcripts disabled by default**: set `messages_config.receive_partial_transcripts: true` to get low-latency partial results.

For the full list of gotchas and diagnostics, see the [troubleshooting skill](../troubleshooting/SKILL.md).

## Further Reading

- [Live quickstart](https://docs.gladia.io/chapters/live-stt/quickstart)
- [Partial transcripts](https://docs.gladia.io/chapters/live-stt/features/partial-transcripts)
- [Endpointing](https://docs.gladia.io/chapters/live-stt/features/endpointing)
- [Live audio intelligence](https://docs.gladia.io/chapters/live-stt/audio-intelligence)
- [API reference: init](https://docs.gladia.io/api-reference/v2/live/init)
- [API reference: WebSocket](https://docs.gladia.io/api-reference/v2/live/websocket)
