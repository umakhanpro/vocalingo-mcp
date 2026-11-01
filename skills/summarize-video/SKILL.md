---
name: summarize-video
description: Summarize a YouTube, TikTok, Instagram, X or Vimeo video, a podcast, lecture, webinar or meeting recording with the VocaLingo MCP server. Gives key moments with timecodes, quotes, chapters and the full transcript on request. Use when the user shares a video link or a recording and asks what it is about, wants notes, a summary, timestamps or a transcript.
---

# Summarize a video or recording

Works through the VocaLingo MCP server (`https://nodejs.vocalingo.app/mcp`). Tools: `summarize_media`, `check_job`, `get_upload_url`.

## Steps

1. Get the source. A link to YouTube, TikTok, Instagram, X, Vimeo or a direct media file goes into `source` as is. For a local file see "Files" below.
2. Call `summarize_media { source }`. Add `summaryLanguage` (ISO code or name) when the user asks for a language or writes in a language other than the video's.
3. If the reply has a `jobId` with `status: "processing"`, call `check_job { jobId }` with that same id until it finishes. Never call `summarize_media` again for the same video while it runs.
4. Present the result: the takeaway in one or two sentences, then key moments with timecodes, then quotes. Keep timecodes as returned. For YouTube you can turn them into `&t=<seconds>` links.
5. Full transcript on request: `check_job { jobId, include: "transcript" }`. It comes in pages: continue with `offset: nextOffset`. Timecoded segments: `include: "segments"`.

## Cost

About 0.8 VocaLingo tokens per minute of media: a 10-minute video costs about 8 tokens. A very long upload can return `confirmation_required` (see below).

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
