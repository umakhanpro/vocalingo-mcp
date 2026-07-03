# VocaLingo MCP Server

Connect your AI agent to [VocaLingo](https://vocalingo.app) — voice cloning, text-to-speech, audio/video translation with voice preservation, subtitles and video summarization. One personal token, no API keys, no local install.

**MCP URL:** `https://nodejs.vocalingo.app/mcp` (Streamable HTTP, remote — nothing to run locally)

Works with Claude Code, Cursor, Windsurf, VS Code, Claude Desktop (via `mcp-remote`), OpenClaw, Hermes and any other MCP client that supports remote servers.

## What your agent can do

- 🎤 **Save your voice** from a short audio sample and use it anywhere (`save_voice`, `list_voices`)
- 🗣️ **Voice any text with your voice** — or OpenAI / ElevenLabs voices (`text_to_speech`)
- 🌍 **Translate audio with your voice preserved** — a voice message in Spanish becomes the same voice speaking English (`translate_audio`)
- 🎬 **Dub videos into another language** with voice cloning and lip-sync (`translate_video`)
- 💬 **Burn subtitles into videos**, optionally translated — a cheap alternative to dubbing (`caption_video`)
- 📝 **Transcribe and summarize** any video or audio — YouTube/TikTok/Instagram link or a file (`summarize_media`, `speech_to_text`)
- 💳 Check balance, history and job status (`get_balance`, `check_job`, `list_jobs`, `get_history`, `get_transactions`)

Everything an agent does appears in your VocaLingo app history — start a translation from your desktop agent, get the result on your phone.

## Quick start

1. Install [VocaLingo](https://vocalingo.app) (iOS / Android / Web) and sign in with your email.
2. Open **Settings → API / MCP Token** and copy the config.
3. Paste it into your MCP client (see below).
4. Ask your agent: *"Summarize this YouTube video: <link>"*.

### Claude Code

```bash
claude mcp add --transport http vocalingo https://nodejs.vocalingo.app/mcp \
  --header "Authorization: Bearer vlmcp_YOUR_TOKEN"
```

### Cursor

`.cursor/mcp.json` (project) or `~/.cursor/mcp.json` (global):

```json
{
  "mcpServers": {
    "vocalingo": {
      "url": "https://nodejs.vocalingo.app/mcp",
      "headers": { "Authorization": "Bearer vlmcp_YOUR_TOKEN" }
    }
  }
}
```

### Claude Desktop

Claude Desktop's connector UI has no field for an `Authorization` header yet, so use the `mcp-remote` bridge in `claude_desktop_config.json` (**Settings → Developer → Edit Config**):

```json
{
  "mcpServers": {
    "vocalingo": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://nodejs.vocalingo.app/mcp", "--header", "Authorization:${AUTH_HEADER}"],
      "env": { "AUTH_HEADER": "Bearer vlmcp_YOUR_TOKEN" }
    }
  }
}
```

Then fully restart Claude Desktop. Full walkthrough (incl. Windows path and why `Authorization:${AUTH_HEADER}` has no space): [configs/claude-desktop.md](configs/claude-desktop.md).

To upload local files from chat attachments, also allow the upload domain once: **Settings → Capabilities → Code execution → Additional allowed domains** → add `sb.vocalingo.app`.

Claude.ai (browser) is not supported yet — it cannot pass an auth header to custom connectors.

### Other clients (OpenClaw, Hermes, ...)

Any client that supports remote MCP (Streamable HTTP) works with the same URL + `Authorization: Bearer` header. See [configs/](configs/) for ready-made snippets.

## How files get to the server

The server is remote, so your agent passes files one of three ways:

1. **Direct URL** — YouTube/TikTok/Instagram links (for `summarize_media`), Telegram Bot API file links, any public file URL. Zero friction: just paste the link.
2. **Presigned upload** — agent calls `get_upload_url`, uploads the local file with `curl`, then passes the returned `fileId` to any tool. Works in Claude Code, Cursor, OpenClaw and inside Claude's code-execution sandbox.
3. **Browser upload page** — no shell? `get_upload_url` also returns a drag&drop link the agent can send you; drop the file in the browser and tell the agent to continue.

## Pricing

Tools spend tokens from your VocaLingo balance — "credits" in tool responses and "tokens" in the app are the **same unit**. There is no separate API billing — top up in the app.

Typical costs (anchors, actual cost depends on length and settings):

| Operation | Approximate cost |
|---|---|
| Transcribe a voice message (`speech_to_text`) | ~1 credit per 2 min |
| Summarize a 10-min YouTube video (`summarize_media`) | ~4 credits |
| Voice 1000 characters of text (`text_to_speech`, minimax) | ~10 credits |
| Translate a 1-min voice message with Qwen auto-clone (`translate_audio`, default) | ~2–3 credits (+~150 one-time if target language needs MiniMax auto-clone) |
| Save a voice (`save_voice`) | ~150 credits one-time |
| Subtitles for a 5-min video (`caption_video`) | ~5–10 credits |
| Dub a 5-min video (`translate_video`) | ~1100 credits — expensive, always confirm first |

Free tools: `get_upload_url`, `check_upload`, `list_voices`, `delete_voice`, `check_job`, `list_jobs`, `get_balance`, `get_history`, `get_transactions`.

Every paid tool supports `dryRun: true` — the agent gets a cost estimate before starting, without charging anything. Agents are instructed to confirm expensive operations with you first.

## Tools reference

See [docs/tools.md](docs/tools.md) for the full reference of all 16 tools with parameters and examples.

| Tool | Type | Description |
|---|---|---|
| `get_upload_url` | free | Presigned URL to upload a local file, returns `fileId` |
| `check_upload` | free | Verify an uploaded file arrived before starting a paid job |
| `save_voice` | paid | Clone a voice from audio (fileId/url) → saved voice |
| `list_voices` | free | List saved voices |
| `delete_voice` | free | Permanently delete a saved voice |
| `text_to_speech` | paid | Text → speech: your saved voice (MiniMax), OpenAI or ElevenLabs voices |
| `speech_to_text` | paid | Audio → text + detected language |
| `translate_audio` | paid, job | Translate audio; default auto-clone (Qwen/MiniMax) / saved / none |
| `summarize_media` | paid, job | Video/audio (URL or file) → transcript + structured summary |
| `translate_video` | paid, job | Dub a video into another language (voice clone + lip-sync) |
| `caption_video` | paid, job | Burn styled subtitles, optionally translated |
| `check_job` | free | Job status / result / error |
| `list_jobs` | free | Recent jobs (recover results in a new session) |
| `get_balance` | free | Current credit balance |
| `get_history` | free | Operation history per tool |
| `get_transactions` | free | Credit charges and top-ups |

## Example prompts

- *"Here's a voice message in German (file attached) — translate it to English keeping the original voice."*
- *"Save my voice from this recording, then read this article aloud with it."*
- *"Summarize this YouTube lecture in Russian: <link>"*
- *"Add English subtitles to this video."*
- *"Dub this video into Spanish. How much will it cost first?"*

## Troubleshooting

- **Claude Desktop: can't find where to enter the token.** The connector UI has no header field — use the `mcp-remote` config above ([full guide](configs/claude-desktop.md)) and fully restart the app.
- **File upload fails with a network error in Claude's sandbox.** Whitelist `sb.vocalingo.app` (Settings → Capabilities → Code execution → Additional allowed domains), or use the `browserUploadUrl` returned by `get_upload_url` and drop the file in the browser.
- **A job stays `processing` for a long time.** That's normal: dubbing takes 5–30 min, translations a few minutes. Keep polling `check_job` with the same `jobId` — never restart the job. You can also close the chat: the job continues on the server, results appear in the app history and via `list_jobs`.
- **A result link has expired.** Media links are valid for 24 hours. Call `check_job` with the same `jobId` — it returns a fresh link. Results are also stored permanently in the app history.
- **`insufficient_balance` error.** The response includes how many credits are required vs available. Top up in the VocaLingo app (Profile → Balance) and retry.
- **`429 rate limit`.** Max 60 requests/min per token and 3 concurrent jobs. Wait a minute; if two agents share your token, they share the limit too.

## Security & fair use

- One personal token per account (`vlmcp_...`). Regenerate it in the app at any time — the old one stops working within a minute. You can also fully disable MCP access in the app (Settings → API / MCP Token → Disable).
- Anyone with your token can spend your balance. Keep it secret.
- Voice cloning: only clone voices you own or have explicit consent to use.
- Uploaded files are automatically deleted after 48 hours. Results live in your app history.

## Docs & support

- Setup guide: [Connect VocaLingo to AI agents](https://help.vocalingo.app/en/articles/7973166-connect-vocalingo-to-ai-agents-mcp-server)
- Help center: [help.vocalingo.app](https://help.vocalingo.app)
- App: [vocalingo.app](https://vocalingo.app)

---

*The server implementation lives in the VocaLingo backend (Node.js + `@modelcontextprotocol/sdk`, Streamable HTTP, stateless). This repository is the public home for docs, configs and issues.*
