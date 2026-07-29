---
name: Kalpa text-to-speech
description: Turn text into spoken audio (mono 16-bit PCM WAV at 24 kHz) with the Kalpa Speech API.
api: openapi/kalpa-labs-openapi-original.json
operations: [tts_v1_tts_post, tts_voice_v1_tts__voice_id__post, models_v1_models_get, voices_v1_voices_get]
---

# Kalpa text-to-speech

Use this skill to synthesize speech from text.

## Auth
Send the per-team API key as `Authorization: Bearer <key>` (or `X-API-Key`). Call from a
server, never a browser. Base URL: `https://api.kalpalabs.ai`.

## Steps
1. (Optional) Call `models_v1_models_get` (`GET /v1/models`) to pick a model. Omit `model`
   to use the default (`kalpa-beta-v0.3`); use `kalpa-atom-beta-v0.1` for fastest/cheapest.
2. (Optional) Call `voices_v1_voices_get` (`GET /v1/voices`) if you want a stored voice, then
   call `tts_voice_v1_tts__voice_id__post` (`POST /v1/tts/{voice_id}`) instead of the default.
3. Call `tts_v1_tts_post` (`POST /v1/tts`) with `{ "text": "...", "speaker": "0" }`.
   Keep text ≤ 8,000 characters per request.
4. Decode `audio.data_b64` (base64) — the result is mono 16-bit PCM WAV at 24 kHz.
5. Read `usage` (`input_chars`, `output_audio_seconds`) for per-call metering.

## Rules
- Errors use one envelope: `{ "error": { "type": "...", "message": "...", "request_id": "..." } }`.
  Match on `error.type`, not `message`.
- On `429` honor `Retry-After`; on `502 inference_error` retry with backoff — generation is
  retry-safe (nothing is committed on failure). There is no Idempotency-Key.
- Pass an `X-Request-ID` to correlate; include it when reporting problems.
