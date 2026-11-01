---
name: translate-voice-message
description: Translate a voice message or audio recording into another language and voice it in the same speaker's voice, with the VocaLingo MCP server. Returns the transcript, the translation and an audio file. Use when the user wants to hear a voice note in their language, reply to someone in a language they don't speak, or translate speech while keeping the voice.
---

# Translate a voice message keeping the voice

Works through the VocaLingo MCP server (`https://nodejs.vocalingo.app/mcp`). Tools: `translate_audio`, `speech_to_text`, `list_voices`, `check_job`, `get_upload_url`.

## Steps

1. Get the source: a direct file link or a `fileId` after upload (see "Files" below). Files up to 150 MB.
2. Call `translate_audio { source, targetLanguage }`. Languages take an ISO code (`es`) or a name (`Spanish`, `Español`).
   - `voice` defaults to `"clone"`: a one-off copy of the speaker's voice from this recording. Nothing is saved.
   - `voice: "none"` returns text only.
   - A `voiceId` from `list_voices` voices the translation with that saved voice (for example, the user's own voice for a reply).
3. Usually the result comes back in the same call: `transcript`, `translatedText`, `summary`, `translatedAudioUrl`. Otherwise you get a `jobId`: call `check_job { jobId }` with the same id until it finishes.
4. Give the user the audio link and the translated text.

If the user only needs to understand what was said, `speech_to_text` plus your own translation is cheaper.

## Cost

About 2 to 3 VocaLingo tokens per minute for target languages en, ru, zh, de, fr, es, it, ja, ko, pt (needs 3+ seconds of speech). Other target languages use another voice engine: about 180 tokens extra and at least 10 seconds of speech, so the call returns `confirmation_required` first.

## Files

- Public link: pass it as `source`.
- Agent with a shell (Claude Code, Cursor, Codex, Gemini CLI): call `get_upload_url { filename }`, run the returned `curlCommand` with the local path, then pass the `fileId` as `source`. A `fileId` stays valid for 48 hours, so reuse it instead of uploading again.
- Chat app (claude.ai, Claude apps): call `get_upload_url` and the user gets an upload button in the chat (if there is no button, give them `browserUploadUrl`). Wait for their message with the `fileId`. Do not upload from a code sandbox: its network blocks the upload domain.
- ChatGPT: a file attached to the message reaches the tool through its `file` parameter, no upload step needed.

## Tokens and confirmation

Tools spend VocaLingo tokens, the currency of the user's VocaLingo account (unrelated to LLM tokens). A call that would cost 50 tokens or more does not start: it returns `status: "confirmation_required"` with the price and a `confirmToken`. Show the price and ask. Only after the user agrees, repeat the same call with `confirmToken` added. Never confirm on the user's behalf. `dryRun: true` returns the estimate without starting anything. On `insufficient_balance`, tell the user to top up in the VocaLingo app and stop.

## Errors

Every error has a `code` and a `nextStep`: follow `nextStep`. If the same call fails twice, show the error to the user and stop.

## Not connected

If no VocaLingo tools are available, the server is not connected yet. Send the user to https://github.com/umakhanpro/vocalingo-mcp#connect. They need a VocaLingo account (https://vocalingo.app) and sign in through the browser.
