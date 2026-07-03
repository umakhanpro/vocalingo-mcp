# VocaLingo MCP — Tools Reference

All 16 tools of the VocaLingo MCP server (`https://nodejs.vocalingo.app/mcp`).

Conventions:

- **Credits**: the same tokens you see in the VocaLingo app, spent from the same balance ("credits" and "tokens" are the same unit).
- **`fileId`**: returned by `get_upload_url` after the agent uploads a local file.
- **`url`**: a direct public link the server downloads itself (SSRF-protected).
- **Job tools** return a `jobId` immediately; poll `check_job` until `status` is `completed` or `failed`.
- **Languages**: tools accept ISO codes (`es`), English names (`Spanish`) and native names (`Español`).
- **`dryRun: true`**: paid tools return a cost estimate without starting or charging.
- **Result links** (audio/video URLs) are signed for 24 hours. Expired? Call `check_job` with the same `jobId` for a fresh link — results are stored permanently in the app history.

---

## Recipes (exact call sequences)

Follow these step by step. Do not add extra calls.

**1. Summarize a YouTube / TikTok / Instagram video**

```
summarize_media { url: "<link>" }        → returns jobId
check_job { jobId }                      → repeat every 10-30s until completed
```

No upload step needed — the server downloads platform links itself.

**2. Translate a local voice message, keep the speaker's voice**

```
get_upload_url { filename: "voice.ogg" } → returns fileId + uploadUrl + curl example
<run the curl command to upload the file>
check_upload { fileId }                  → verify uploaded: true
translate_audio { targetLanguage: "en", fileId: "<fileId>", dryRun: true }  → show estimate
translate_audio { targetLanguage: "en", fileId: "<fileId>" }                → returns jobId (auto-clone is default)
check_job { jobId }                      → repeat until completed
```

`voiceMode: "clone"` is the **default** (auto-clone): clones the speaker from this audio for a one-off voice-over — it does **not** save the voice. Qwen auto-clone for `en, ru, zh, de, fr, es, it, ja, ko, pt` (min 3s, cheap). Other target languages use MiniMax auto-clone (min 10s, ~150 credits extra). Use `voiceMode: "none"` for text-only, or `voiceMode: "saved"` with a voice from `save_voice`.

**3. Read a text aloud with the user's own voice**

```
list_voices {}                           → pick voiceId (status must be "completed")
text_to_speech { text: "...", voiceId: "<voiceId>" }   → returns audioUrl
```

If `list_voices` is empty: first `save_voice` from a 10-60s audio sample of the user, then `text_to_speech`.

**4. Add subtitles to a video (optionally translated)**

```
get_upload_url + curl upload + check_upload (or use a direct file URL)
caption_video { fileId: "<fileId>", translateTo: "en", dryRun: true }   → show estimate
caption_video { fileId: "<fileId>", translateTo: "en" }                 → returns jobId
check_job { jobId }                      → repeat until completed
```

**5. Dub a video into another language (EXPENSIVE — always dryRun first)**

```
translate_video { targetLanguage: "es", fileId: "<fileId>", dryRun: true }   → show estimate, ASK THE USER
translate_video { targetLanguage: "es", fileId: "<fileId>" }                 → returns jobId
check_job { jobId }                      → repeat every 30s; may take 5-30 minutes
```

## Common mistakes (do NOT do this)

- **Do not call a paid tool again while its job is `processing`.** Long processing is normal. Keep polling `check_job` with the same `jobId`.
- **Do not re-upload the same file.** A `fileId` is valid for 48 hours — reuse it across calls.
- **`dryRun: true` does not start anything.** After showing the estimate, call the same tool again without `dryRun` to actually start.
- **Do not retry on `insufficient_balance`.** Tell the user to top up in the VocaLingo app and stop.
- **Do not pass YouTube links to `translate_video` or `caption_video`** — they need a direct video file (`fileId` or a direct `.mp4` URL). Platform links only work in `summarize_media`.
- **Do not loop on language variants.** If a language is rejected, the error lists supported ones — pick from that list.

---

## Files

### `get_upload_url` — free

Creates a presigned upload URL for a local file.

| Param | Type | Description |
|---|---|---|
| `filename` | string | Original filename with extension, e.g. `voice.ogg` |

Returns `fileId`, `uploadUrl` (PUT, expires in ~2h), a ready `curlExample`, and `browserUploadUrl` — a drag&drop page for users without a shell. Files are deleted after 48 hours.

Size limits by consuming tool: audio tools (`translate_audio`, `speech_to_text`, `save_voice`) accept up to **50 MB**; video tools (`summarize_media`, `translate_video`, `caption_video`) up to **100 MB**.

Supported extensions: mp3, m4a, wav, ogg, oga, opus, webm, aac, flac, mp4, mov, mkv, avi, m4v, mpeg, mpg.

### `check_upload` — free

Verifies that a file uploaded via `get_upload_url` actually arrived. Call after uploading (especially via the browser page) and before starting a paid job.

| Param | Type | Description |
|---|---|---|
| `fileId` | string | From `get_upload_url` |

Returns `uploaded: true/false` and the file size. If `false` right after a browser upload, retry in ~10 seconds.

---

## Voice

### `save_voice` — paid (~150 credits)

Clones a voice from an audio sample and saves it to the account. Waits for completion (~1 min).

| Param | Type | Description |
|---|---|---|
| `name` | string | Voice name |
| `fileId` / `url` | string | Audio sample: **minimum 10 sec**, up to 60 sec of clean speech (longer audio is trimmed automatically) |
| `language` | string? | Sample language (auto-detected if omitted) |

Only clone voices you own or have explicit consent to use.

### `list_voices` — free

Lists saved voices with `voiceId`, name, language, status.

### `delete_voice` — free

Permanently deletes a saved voice from the library. Irreversible; clone credits are not refunded — agents should confirm with you first.

| Param | Type | Description |
|---|---|---|
| `voiceId` | string | From `list_voices` |

### `text_to_speech` — paid (~10 credits per 1000 chars with minimax)

Converts text to speech. Returns an audio URL. Three providers:

| Provider | When to use | Text limit |
|---|---|---|
| `minimax` (default) | The **only** provider with the user's saved cloned voices | 30 000 chars |
| `openai` | Natural voices, free-form style `instructions` | 4 000 chars |
| `elevenlabs` | High-quality multilingual voices | 10 000 chars |

| Param | Type | Description |
|---|---|---|
| `text` | string | Text to speak |
| `provider` | string? | `minimax` (default) / `openai` / `elevenlabs` |
| `voiceId` | string? | minimax: id from `list_voices` (default — the latest saved voice, else a system voice); openai: `cedar`, `marin`, `alloy`, `nova`, ... (default `cedar`); elevenlabs: an ElevenLabs voice id (default George) |
| `speed` | number? | minimax 0.5–2.0, elevenlabs 0.7–1.2. Ignored for openai |
| `emotion` | string? | minimax only: `happy`, `sad`, `angry`, `neutral`, ... |
| `instructions` | string? | openai only: free-form style, e.g. `"calm and slow, like a bedtime story"` |
| `dryRun` | bool? | Cost estimate only, nothing generated or charged |

To speak with the **user's own voice**, use `provider: "minimax"` (or just omit `provider`) + `voiceId` from `list_voices`.

Note: the **first** use of a saved voice created in an older app version may add a one-time ~150 credits activation fee. `dryRun` always shows the exact total — check it before generating with an old voice.

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
| `voiceMode` | string? | `clone` (**default**, auto-clone) — one-off clone of the speaker's voice from **this** audio (not saved; Qwen for `en, ru, zh, de, fr, es, it, ja, ko, pt`, min 3s; MiniMax for other languages, min 10s, ~150 credits extra); `saved` — voice from `save_voice`; `none` — text only |
| `savedVoiceId` | string? | Required when `voiceMode: "saved"` |
| `dryRun` | bool? | Cost estimate only |

Result (via `check_job`): transcript, translated text, summary, and — unless `voiceMode: "none"` — a translated audio URL with auto-cloned voice.

Auto-clone provider depends on **target language**: Qwen for `en, ru, zh, de, fr, es, it, ja, ko, pt` (a few credits); MiniMax for other languages (~150 credits extra). `dryRun` always returns the exact estimate — check it first.

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
| `audioOnly` | bool? | Dub the audio track only, no lip-sync (same price, faster) |
| `dryRun` | bool? | Cost estimate only |

Max video duration: **30 min** (longer videos are rejected before any charge). Takes 5–30 minutes.

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

Returns status, progress/stages, result (text + signed media URLs) or a human-readable error. Poll every 10–30 seconds. Media URLs are signed for 24 hours — if a link expired, call `check_job` again with the same `jobId` for a fresh one.

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
