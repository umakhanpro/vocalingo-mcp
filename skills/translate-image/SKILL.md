---
name: translate-image
description: Redraw the text on a photo or image (menu, sign, document, poster, screenshot, infographic) in another language, keeping the layout, fonts and colors, with the VocaLingo MCP server. Use when the user wants the picture itself back in another language, for example to share a translated menu or localize a banner.
---

# Translate the text on an image

Works through the VocaLingo MCP server (`https://nodejs.vocalingo.app/mcp`). Tools: `translate_image`, `get_upload_url`.

## When to use it

If the user only wants to read or understand the text on a photo and you can see the image, translate it yourself: no VocaLingo call is needed. Use `translate_image` when they want a new image with the text redrawn.

## Steps

1. Get the image: a link or a `fileId` after upload (see "Files" below). jpg, png or webp up to 20 MB.
2. Call `translate_image { source, targetLanguage }`. It takes 20 to 40 seconds.
3. The reply has `imageUrl` (valid 7 days) and a small preview you can check. Hosts with MCP Apps show the image in the chat with Save and Open buttons.

## Cost

4 to 6 VocaLingo tokens per image.

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
