# VocaLingo MCP Server

Give your AI agent ears and a voice: transcribe and summarize any video or voice message, translate speech **keeping the speaker's voice**, read text aloud **in your own cloned voice**, add subtitles or dub a video. Results also land in your [VocaLingo](https://vocalingo.app) app.

**MCP URL:** `https://nodejs.vocalingo.app/mcp` (remote, Streamable HTTP — nothing to install)

Works with claude.ai, Claude Desktop, Claude Code, ChatGPT, Cursor, Windsurf, VS Code, OpenClaw, Hermes and any MCP client that supports remote servers.

## What your agent can do

- 📝 **Watch videos for you** — summary, key moments with timecodes, quotes, full transcript of a YouTube/TikTok/Instagram link or a file (`summarize_media`)
- 🎧 **Understand voice messages** — speech → text (`speech_to_text`)
- 🌍 **Translate speech in the same voice** — a voice message in Spanish becomes the same voice speaking English (`translate_audio`)
- 🗣️ **Speak in your voice** — save your voice once, then voice any text with it, or use OpenAI / ElevenLabs voices (`save_voice`, `list_voices`, `text_to_speech`)
- 💬 **Subtitles**, optionally translated (`caption_video`) and 🎬 **dubbing** with lip-sync (`translate_video`)
- 💳 Job status and account (`check_job`, `get_account`)

12 tools in total — [full reference](docs/tools.md).

## Connect

### claude.ai, Claude Desktop, ChatGPT (sign in, no token)

1. Settings → **Connectors** → **Add custom connector** (ChatGPT: Settings → Apps & Connectors → Create).
2. URL: `https://nodejs.vocalingo.app/mcp`
3. Sign in with the email of your VocaLingo account and allow access.

No VocaLingo account yet? Install the app ([vocalingo.app](https://vocalingo.app)) and sign up with your email first.

### Claude Code

```bash
claude mcp add --transport http vocalingo https://nodejs.vocalingo.app/mcp
```

Claude Code opens the browser to sign in. Prefer a token? Copy it from the app (**Settings → API / MCP Token**) and add `--header "Authorization: Bearer vlmcp_YOUR_TOKEN"`.

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

The token is in the app: **Settings → API / MCP Token**. More snippets (incl. Claude Desktop via `mcp-remote` with a token): [configs/](configs/).

## Try it

- *"Summarize this YouTube lecture in Russian: <link>"*
- *"Here's a voice message in German — translate it to English keeping the voice."*
- *"Save my voice from this recording, then read this article aloud with it."*
- *"Add English subtitles to this video."*
- *"Dub this video into Spanish."*

## Files

Your agent passes files in one `source` parameter:

1. **A link** — YouTube/TikTok/Instagram (for `summarize_media`), a Telegram Bot API file link, any public file URL. Just paste it.
2. **A local file** — the agent calls `get_upload_url`, uploads with the returned `curl` command, and passes the `fileId`. Works in Claude Code, Cursor, OpenClaw and in Claude's code-execution sandbox (allow the domain `sb.vocalingo.app` once: Settings → Capabilities → Code execution → Additional allowed domains).
3. **No shell?** `get_upload_url` also returns a drag&drop page link — drop the file in the browser.

## Pricing and safety

Tools spend credits from your VocaLingo balance — the same tokens as in the app. No separate API billing.

| Operation | Approximate cost |
|---|---|
| Transcribe a voice message | ~1 credit per 2 min |
| Summarize a 10-min video | ~4 credits |
| Translate a 1-min voice message in the same voice | ~2–3 credits (+150 for languages outside en, ru, zh, de, fr, es, it, ja, ko, pt) |
| Voice 1000 characters in your voice | ~10 credits |
| Save a voice | 150 credits |
| Subtitles for a 5-min video | ~5–10 credits |
| Dub a 5-min video | ~1100 credits |

Built so that even a small model can't burn your balance:

- **Anything that costs 50+ credits asks you first.** The server returns the price and a one-time `confirmToken`; the job starts only when the agent repeats the call with it.
- **No double charges.** Repeating the same paid request within 10 minutes returns the same job.
- **Long jobs don't need babysitting.** Short ones come back finished in the same call; long ones return a `jobId`, and `check_job` waits on the server.
- **Errors say what to do next** (`code` + `nextStep`), e.g. "this is a local path — upload it first".

## Troubleshooting

- **A job stays `processing`.** Normal for dubbing (5–30 min). Keep calling `check_job` with the same `jobId`. You can close the chat — results appear in the app history and in `check_job` without a `jobId`.
- **Upload fails with a network error in Claude's sandbox.** Allow `sb.vocalingo.app` (see Files) or use the browser upload link.
- **`insufficient_balance`.** Top up in the app (Profile → Balance) and retry.
- **`429`.** Max 60 requests/min per account and 3 concurrent jobs.

## Security

- Sign-in connections and the personal token (`vlmcp_…`) can be turned off any time: **Settings → API / MCP Token → Disable**. Regenerating the token kills the old one within a minute.
- Anyone with your token can spend your balance — keep it secret.
- Only clone voices you own or have consent to use. Uploaded files are deleted after 48 hours.

## Docs & support

- [Tools reference](docs/tools.md)
- Setup guide: [Connect VocaLingo to AI agents](https://help.vocalingo.app/en/articles/7973166-connect-vocalingo-to-ai-agents-mcp-server)
- Help center: [help.vocalingo.app](https://help.vocalingo.app)

---

*The server lives in the VocaLingo backend (Node.js, `@modelcontextprotocol/server` v2, stateless Streamable HTTP; OAuth via Supabase Auth). This repository is the public home for docs, configs and issues.*
