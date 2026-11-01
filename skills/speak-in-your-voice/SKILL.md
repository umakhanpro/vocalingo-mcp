---
name: speak-in-your-voice
description: Read a text aloud in the user's own cloned voice, or in an OpenAI or ElevenLabs voice, with the VocaLingo MCP server. Can save a new voice from a 10 to 60 second sample. Use when the user wants a voice-over, an audio version of an article or post, a voice reply, narration for a video, or asks to "say this in my voice".
---

# Voice a text in the user's own voice

Works through the VocaLingo MCP server (`https://nodejs.vocalingo.app/mcp`). Tools: `list_voices`, `save_voice`, `text_to_speech`, `delete_voice`, `check_job`, `get_upload_url`.

## Steps

1. Call `list_voices`. `ownVoices` are the voices the user saved in VocaLingo; `stockVoices` are OpenAI and ElevenLabs voices.
2. No own voice and the user wants theirs: ask for a 10 to 60 second recording of clean speech, then `save_voice { name, source }` (see "Files" below). Only clone a voice the user owns or has explicit consent to use. The voice also appears in the VocaLingo app.
3. Call `text_to_speech { text, voice: "<voiceId>" }`.
   - Own voices speak en, ru, zh, de, fr, es, it, ja, ko, pt, up to 10,000 characters. Pass `language` for the text's language and optionally `emotion` (`happy`, `sad`, `angry`, `neutral`…) or `speed` (0.5 to 2.0).
   - OpenAI voices (`cedar`, `marin`, `alloy`, `nova`, `onyx`…): up to 4,000 characters, `instructions` sets the style ("calm and slow").
   - ElevenLabs voices: a name like `George` or an id from `list_voices`.
4. The reply has `audioUrl`. A long text can return `status: "processing"` with a `jobId`: call `check_job { jobId }` until it finishes.

## Cost

Own voice: about 11 to 12 VocaLingo tokens per 1,000 characters. Saving a voice: about 1 token. `delete_voice` is free and irreversible: ask before calling it.

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
