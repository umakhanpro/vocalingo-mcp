# VocaLingo MCP Server

Give your AI agent ears and a voice: transcribe and summarize any video or voice message, translate speech **keeping the speaker's voice**, read text aloud **in your own cloned voice**, add subtitles or dub a video. Results also land in your [VocaLingo](https://vocalingo.app) app.

**MCP URL:** `https://nodejs.vocalingo.app/mcp` (remote, Streamable HTTP — nothing to install)

Works with claude.ai, Claude Desktop, Claude Code, ChatGPT, Cursor, Windsurf, VS Code, OpenClaw, Hermes and any MCP client that supports remote servers.

## What your agent can do

- 📝 **Watch videos for you** — summary, key moments with timecodes, quotes, full transcript of a YouTube/TikTok/Instagram link or a file (`summarize_media`)
- 🎧 **Understand voice messages** — speech → text (`speech_to_text`)
- 🌍 **Translate speech in the same voice** — a voice message in Spanish becomes the same voice speaking English (`translate_audio`)
- 🗣️ **Speak in your voice** — the voices you saved in the app work right away, or save a new one; also OpenAI / ElevenLabs voices (`list_voices`, `text_to_speech`, `save_voice`)
- ⬇️ **Download a video or its mp3** from YouTube, TikTok, Instagram and more (`download_media`)
- 🎭 **Change the voice** in a recording or video (`change_voice`)
- 🖼️ **Translate a photo of a menu, sign or document** as an image (`translate_image`)
- 💬 **Subtitles**, optionally translated (`caption_video`) and 🎬 **dubbing** with lip-sync (`translate_video`)
- 💳 Job status and account (`check_job`, `get_account`)

[Full reference](docs/tools.md).

## Connect

### claude.ai, Claude Desktop, Claude mobile (sign in, no token)

1. Open **[Add VocaLingo to Claude](https://claude.ai/customize/connectors?modal=add-custom-connector&connectorName=VocaLingo&connectorUrl=https%3A%2F%2Fnodejs.vocalingo.app%2Fmcp)** — the "Add custom connector" dialog opens with the name and URL filled in. Or add it by hand: Customize → **Connectors** → **Add custom connector**, URL `https://nodejs.vocalingo.app/mcp`.
2. Click **Add**, then **Connect**.
3. Sign in to VocaLingo: email code, Google, or a one-time code from the app (**Settings → AI agents (MCP)** → "Get a sign-in code" — handy if you signed up with Apple).

The connector then works in claude.ai, Claude Desktop and the Claude mobile apps.

### ChatGPT (sign in, no token)

1. Open [chatgpt.com/plugins](https://chatgpt.com/plugins) → **+** → **Add custom MCP server** (availability depends on your plan and workspace policy).
2. Name `VocaLingo`, URL `https://nodejs.vocalingo.app/mcp`, authentication **OAuth** → **I understand and want to continue** → **Create**.
3. Sign in to VocaLingo the same way as above.

No VocaLingo account yet? Install the app ([vocalingo.app](https://vocalingo.app)) and sign up with your email first.

### Claude Code

```bash
claude mcp add --transport http vocalingo https://nodejs.vocalingo.app/mcp
```

Claude Code opens the browser to sign in. Prefer a token? Copy it from the app (**Settings → AI agents (MCP)**) and add `--header "Authorization: Bearer vlmcp_YOUR_TOKEN"`.

### Cursor, Windsurf, VS Code and others

`.cursor/mcp.json` (or your client's MCP config):

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

The token is in the app: **Settings → AI agents (MCP)**. More snippets (incl. Claude Desktop via `mcp-remote` with a token): [configs/](configs/).

## Try it

- *"Summarize this YouTube lecture in Russian: <link>"*
- *"Here's a voice message in German — translate it to English keeping the voice."*
- *"Save my voice from this recording, then read this article aloud with it."*
- *"Add English subtitles to this TikTok: <link>"*
- *"Download the audio of this YouTube video."*
- *"Translate this menu photo into English."*
- *"Dub this video into Spanish."*

## Files

How to send audio, video or a photo:

1. **A link** — YouTube/TikTok/Instagram, a Telegram Bot API file link, any public file URL. Just paste it into the message.
2. **ChatGPT** — attach the file to your message. ChatGPT hands it to the tool directly (the `file` parameter).
3. **claude.ai, Claude apps** — ask Claude to upload a file. It calls `get_upload_url`, and an **upload button appears right in the chat**: pick the file, and the chat gets a message with its `fileId` so Claude continues on its own. No button? Claude gives you a link to an upload page instead — drop the file there and write "done".
4. **Agents with a shell** (Claude Code, Cursor, OpenClaw) — just give the path. The agent calls `get_upload_url`, uploads with the returned `curl` command and passes the `fileId`.

Only need the text on a photo read or translated? Claude and ChatGPT can do that themselves — VocaLingo's `translate_image` is for getting the picture itself back with the text redrawn in another language.

## Pricing and safety

Tools spend VocaLingo tokens from your balance, the same tokens as in the app. No separate API billing.

| Operation | Approximate cost |
|---|---|
| Transcribe a voice message | ~1 token per 2 min |
| Summarize a 10-min video | ~4 tokens |
| Translate a 1-min voice message in the same voice | ~2–3 tokens (+150 for languages outside en, ru, zh, de, fr, es, it, ja, ko, pt) |
| Voice 1000 characters in your voice | ~9–10 tokens |
| Save a voice | ~1 token |
| Translate a photo | 4–6 tokens |
| Change the voice in a 1-min recording | ~12 tokens |
| Download a video or audio | free |
| Subtitles for a 5-min video | ~5–10 tokens |
| Dub a 5-min video | ~1100 tokens |

Built so that even a small model can't burn your balance:

- **Anything that costs 50+ tokens asks you first.** The server returns the price and a one-time `confirmToken`; the job starts only when the agent repeats the call with it.
- **No double charges.** Repeating the same paid request within 10 minutes returns the same job.
- **Long jobs don't need babysitting.** Short ones come back finished in the same call; long ones return a `jobId`, and `check_job` waits on the server.
- **Errors say what to do next** (`code` + `nextStep`), e.g. "this is a local path — upload it first".

## Troubleshooting

- **A job stays `processing`.** Normal for dubbing (5–30 min). Keep calling `check_job` with the same `jobId`. You can close the chat — results appear in the app history and in `check_job` without a `jobId`.
- **No upload button in Claude.** Ask Claude for the upload page link and drop the file there. If Claude tries to upload from its code sandbox and gets a network error, allow `sb.vocalingo.app` once (Settings → Capabilities → Code execution → Additional allowed domains).
- **`insufficient_balance`.** Top up in the app (Profile → Balance) and retry.
- **`429`.** Max 60 requests/min per account and 3 concurrent jobs.

## Security

- Sign-in connections and the personal token (`vlmcp_…`) can be turned off any time: **Settings → AI agents (MCP) → Disable**. Regenerating the token kills the old one within a minute.
- Anyone with your token can spend your balance — keep it secret.
- Only clone voices you own or have consent to use. Uploaded files are deleted after 48 hours.

## Docs & support

- [Tools reference](docs/tools.md)
- Setup guide: [Connect VocaLingo to AI agents](https://help.vocalingo.app/en/articles/7973166-connect-vocalingo-to-ai-agents-mcp-server)
- Help center: [help.vocalingo.app](https://help.vocalingo.app)

---

*The server lives in the VocaLingo backend (Node.js, `@modelcontextprotocol/server` v2, stateless Streamable HTTP; OAuth via Supabase Auth). This repository is the public home for docs, configs and issues.*
