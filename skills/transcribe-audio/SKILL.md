---
name: transcribe-audio
description: Turn a voice message, audio recording, call, interview or video soundtrack into text with the VocaLingo MCP server. Detects the language. Use when the user shares a voice note (Telegram, WhatsApp), an mp3/m4a/ogg/wav file or a recording and asks what was said, wants it written down or needs a transcript.
---

# Transcribe a voice message or recording

Works through the VocaLingo MCP server (`https://nodejs.vocalingo.app/mcp`). Tools: `speech_to_text`, `summarize_media`, `get_upload_url`, `check_job`.

## Steps

1. Get the source: a direct file link (a Telegram Bot API file link works) or a `fileId` after upload (see "Files" below).
2. Call `speech_to_text { source }`. It returns `text`, `language` and `durationSeconds`.
3. Give the user the text. If they asked for the meaning in another language, translate the text yourself: no extra VocaLingo call is needed.

Pick `summarize_media` instead when the recording is long (a meeting, lecture or podcast), bigger than 50 MB, comes as a YouTube/TikTok/Instagram link, or the user wants a summary or timecodes. Its transcript is available through `check_job { jobId, include: "transcript" }`.

## Cost

About 1 VocaLingo token per 2 minutes of audio. Files up to 50 MB.

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
