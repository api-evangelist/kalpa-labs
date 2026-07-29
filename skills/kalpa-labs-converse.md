---
name: Kalpa conversation completion
description: Complete the open turn of a multi-speaker conversation — authored speech or contextual TTS — with the Kalpa Speech API.
api: openapi/kalpa-labs-openapi-original.json
operations: [converse_v1_converse_post, converse_stream_v1_converse_stream_post, models_v1_models_get]
---

# Kalpa conversation completion

Use this skill to have a model complete the last turn of a conversation.

## Auth
`Authorization: Bearer <key>` (or `X-API-Key`), server-side only. Base URL `https://api.kalpalabs.ai`.

## Steps
1. Build a `conversation` array, oldest turn first. Each turn: `{ "speaker", "text"?, "audio_wav_b64"? }`.
   Every turn except the last must carry `text` and/or `audio_wav_b64` (grounded history).
2. Shape the **last (open) turn**:
   - Only a `speaker` → the model authors and voices the next turn.
   - `speaker` + `text` → contextual TTS: renders exactly that text in context.
   - A last turn with both text and audio has nothing to generate → `400`.
3. Use only the model's advertised `speaker` labels (current models: `"0"` and `"1"`, in turn
   order). Names or custom labels degrade output — check `models_v1_models_get` (`GET /v1/models`).
4. Call `converse_v1_converse_post` (`POST /v1/converse`). Read `reply.text`, decode
   `reply.audio.data_b64`, and read `model` (resolved id) + `usage`.
5. For live/streaming multi-speaker sessions, use the WebSocket
   (`converse_stream_v1_converse_stream_post`, `wss://api.kalpalabs.ai/v1/converse/stream`) —
   the server holds history and streams audio back as it is generated.

## Caps & rules
- Caps: 64 turns/conversation, 8,000 chars/turn, 25 MiB decoded WAV/turn (live at `GET /v1/info`).
- Same one-shape error envelope; match `error.type`. `429` → honor `Retry-After`; `502` → retry.
- Pass `X-Request-ID` to correlate.
