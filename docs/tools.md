# VocaLingo MCP — Tools Reference

All 14 tools of the VocaLingo MCP server (`https://nodejs.vocalingo.app/mcp`).

Conventions:

- **Credits**: the same tokens you see in the VocaLingo app, spent from the same balance.
- **`fileId`**: returned by `get_upload_url` after the agent uploads a local file.
- **`url`**: a direct public link the server downloads itself (SSRF-protected).
- **Job tools** return a `jobId` immediately; poll `check_job` until `status` is `completed` or `failed`.
- **Languages**: tools accept ISO codes (`es`), English names (`Spanish`) and native names (`Español`).
- **`dryRun: true`**: job tools return a cost estimate without starting or charging.

---

## Files

### `get_upload_url` — free

Creates a presigned upload URL for a local file.

| Param | Type | Description |
|---|---|---|
| `filename` | string | Original filename with extension, e.g. `voice.ogg` |

Returns `fileId`, `uploadUrl` (PUT, expires in ~2h), a ready `curlExample`, and `browserUploadUrl` — a drag&drop page for users without a shell. Files are deleted after 48 hours. Max 100 MB.

Supported extensions: mp3, m4a, wav, ogg, oga, opus, webm, aac, flac, mp4, mov, mkv, avi, m4v, mpeg, mpg.

---

## Voice

### `save_voice` — paid (~150 credits)

Clones a voice from an audio sample and saves it to the account. Waits for completion (~1 min).

| Param | Type | Description |
|---|---|---|
| `name` | string | Voice name |
| `fileId` / `url` | string | Audio sample (3–60 sec is used; longer audio is trimmed) |
| `language` | string? | Sample language (auto-detected if omitted) |

Only clone voices you own or have explicit consent to use.

### `list_voices` — free

Lists saved voices with `voiceId`, name, language, status.

### `text_to_speech` — paid (~10 credits per 1000 chars)

Converts text to speech. Returns an mp3 URL.

| Param | Type | Description |
|---|---|---|
| `text` | string | Up to 30 000 chars (auto-chunked and merged) |
| `voiceId` | string? | From `list_voices`; default — the latest saved voice, else a system voice |
| `speed` | number? | 0.5–2.0 |
| `emotion` | string? | e.g. `happy`, `sad`, `angry`, `neutral` |

---

## Speech & audio

### `speech_to_text` — paid (~1 credit per ~2 min)

Transcribes audio. Returns text + detected language.

| Param | Type | Description |
|---|---|---|
| `fileId` / `url` | string | Audio (max 50 MB) |

### `translate_audio` — paid, job

Translates audio into another language: transcription → translation → optional voice-over.

| Param | Type | Description |
|---|---|---|
| `targetLanguage` | string | Target language |
| `fileId` / `url` | string | Source audio |
| `voiceMode` | string? | `clone` — clone the speaker's voice for the voice-over; `saved` — use a saved voice; `none` (default) — text only |
| `savedVoiceId` | string? | Required when `voiceMode: "saved"` |
| `dryRun` | bool? | Cost estimate only |

Result (via `check_job`): transcript, translated text, summary, and — with a voice mode — a translated audio URL.

---

## Video

### `summarize_media` — paid, job (~1 credit per ~2.5 min)

Transcribes and summarizes a video or audio. Accepts platform links (YouTube, TikTok, Instagram, X, Vimeo, ...) — the server downloads them itself.

| Param | Type | Description |
|---|---|---|
| `url` / `fileId` | string | Video page URL, direct file URL, or uploaded file |
| `outputLanguage` | string? | Language of the summary (default — your app language) |
| `dryRun` | bool? | Cost estimate only |

Result: title, structured summary (key moments, quotes, takeaway), full transcript and timecoded segments (SRT-compatible).

### `translate_video` — paid, job (~220 credits/min — EXPENSIVE)

Dubs a video into another language with voice cloning and lip-sync (HeyGen). Agents should always show you the `dryRun` estimate first.

| Param | Type | Description |
|---|---|---|
| `targetLanguage` | string | Target language |
| `fileId` / `url` | string | Video **file** (platform links are not supported here) |
| `audioOnly` | bool? | Dub the audio track only |
| `dryRun` | bool? | Cost estimate only |

Max video duration: 30 min. Takes 5–30 minutes.

### `caption_video` — paid, job (~1–2 credits/min)

Burns styled subtitles into a video; optionally translates them (a cheap alternative to dubbing).

| Param | Type | Description |
|---|---|---|
| `fileId` / `url` | string | Video file |
| `translateTo` | string? | Translate subtitles to this language; omit for same-language captions |
| `style` | string? | `classic-outline` (default), `tiktok-bold`, `boxed`, `minimal`, `cinematic-top`, `neon-cyan`, `yellow-pop`, `subtitle-pill`, `red-punch`, `premium-gold` |
| `dryRun` | bool? | Cost estimate only |

---

## Jobs & account

### `check_job` — free

| Param | Type | Description |
|---|---|---|
| `jobId` | string | From any job tool (`va_…`, `sm_…`, `dl_…`, `tv_…`, `cv_…`, `vc_…`) |

Returns status, progress/stages, result (text + signed media URLs) or a human-readable error. Poll every 10–30 seconds.

### `list_jobs` — free

Recent jobs across all tools. Use it to recover results if the chat session was interrupted mid-job.

### `get_balance` — free

Current credit balance.

### `get_history` — free

Recent operations for a given tool (`text_to_speech`, `translate_audio`, `summarize_media`, `translate_video`, `caption_video`). Non-premium accounts see up to 3 latest records per tool.

### `get_transactions` — free

Recent credit charges and top-ups — what was spent on what.

---

## Errors

Paid tools return a structured error when the balance is too low:

```json
{ "error": "insufficient_balance", "requiredCredits": 42, "availableCredits": 10, "howToTopUp": "Top up your balance in the VocaLingo app (Profile -> Balance)." }
```

Rate limits: 60 requests/min per token; max 3 concurrent active jobs.
