---
name: change-voice
description: Replace the voice in an audio recording or video with another voice while keeping intonation and timing, using the VocaLingo MCP server and ElevenLabs stock voices. Use when the user wants to anonymize a voice, make a recording sound like a different narrator, or re-voice a clip.
---

# Change the voice in a recording or video

Works through the VocaLingo MCP server (`https://nodejs.vocalingo.app/mcp`). Tools: `change_voice`, `list_voices`, `download_media`, `check_job`, `get_upload_url`.

## Steps

1. Get the file: a `fileId` after upload or a direct file link (see "Files" below), audio or video up to 5 minutes and 100 MB. For a YouTube, TikTok or Instagram link call `download_media { source: "<link>" }` first and use the `source` it returns.
2. Pick a voice: `list_voices` and take one from `stockVoices.elevenlabs` (a name like `George` or its `voiceId`). The user's own cloned voices do not work here.
3. Call `change_voice { source, voice }`. The reply has `resultUrl`, or a `jobId` for `check_job`.
4. Video keeps its picture. Videos of users without VocaLingo Premium get a small watermark.

## Cost

Charged by duration: about 14 VocaLingo tokens for a 1-minute recording.

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
