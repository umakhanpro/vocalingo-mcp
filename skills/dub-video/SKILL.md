---
name: dub-video
description: Dub a video into another language with the original speakers' cloned voices and optional lip-sync, using the VocaLingo MCP server. Works with an uploaded file or a YouTube, TikTok or Instagram link, up to 30 minutes. Use when the user wants a video translated with voice-over, dubbed, or localized for another audience.
---

# Dub a video into another language

Works through the VocaLingo MCP server (`https://nodejs.vocalingo.app/mcp`). Tools: `translate_video`, `download_media`, `check_job`, `get_upload_url`.

## Steps

1. Get the video file.
   - Uploaded file or direct file link: use it as `source` (see "Files" below). Max 100 MB and 30 minutes.
   - YouTube, TikTok, Instagram or another platform link: call `download_media { source: "<link>" }` first and use the `source` it returns. `translate_video` does not accept platform links.
2. Call `translate_video { source, targetLanguage }`. Add `audioOnly: true` to dub only the audio track without lip-sync: same price, faster.
3. Dubbing is expensive, so the first call always returns `confirmation_required` with the price. Tell the user the price in VocaLingo tokens and ask. Only after a clear yes repeat the same call with `confirmToken`.
4. You get a `jobId`. Dubbing takes 5 to 30 minutes: keep calling `check_job { jobId }` with the same id. `processing` is normal. Never start the dub again; the user can close the chat and find the result in the VocaLingo app later.

## Cost

About 240 VocaLingo tokens per minute of video, so a 5-minute video costs about 1,200 tokens. Use `dryRun: true` to show the estimate before anything starts.

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
