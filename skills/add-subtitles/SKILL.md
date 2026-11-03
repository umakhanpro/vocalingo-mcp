---
name: add-subtitles
description: Burn subtitles into a video, in the original language or translated, with the VocaLingo MCP server. Ten caption styles, including TikTok-style bold captions. Works with an uploaded file or a YouTube, TikTok or Instagram link. Use when the user wants captions, subtitles or translated subtitles on a video or a short.
---

# Add subtitles to a video

Works through the VocaLingo MCP server (`https://nodejs.vocalingo.app/mcp`). Tools: `caption_video`, `download_media`, `check_job`, `get_upload_url`.

## Steps

1. Get the video file.
   - Uploaded file or direct file link: use it as `source` (see "Files" below). Max 100 MB.
   - YouTube, TikTok, Instagram or another platform link: call `download_media { source: "<link>" }` first and use the `source` it returns. `caption_video` does not accept platform links.
2. Call `caption_video { source }`.
   - `targetLanguage` translates the subtitles; leave it out to keep the original language.
   - `style`: `classic-outline` (default), `tiktok-bold`, `boxed`, `minimal`, `cinematic-top`, `neon-cyan`, `yellow-pop`, `subtitle-pill`, `red-punch`, `premium-gold`, `amber-italic-box`, `amber-italic-block`. Suggest `tiktok-bold` for vertical shorts.
3. You get a `jobId`: call `check_job { jobId }` with the same id until it finishes, then give the user the video link.

## Cost

About 1 to 2 VocaLingo tokens per minute of video. `download_media` is free.

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
