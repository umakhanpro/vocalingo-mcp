# VocaLingo MCP — Tools Reference

All 15 tools of the VocaLingo MCP server (`https://nodejs.vocalingo.app/mcp`).

Conventions:

- **`source`** — every tool that takes a file has one `source` parameter: either a **fileId** from `get_upload_url` or a **public https link** (YouTube/TikTok/Instagram for `summarize_media` and `download_media`, a Telegram Bot API file link, any direct file URL). A local path like `/Users/me/voice.ogg` is rejected with instructions to upload it first.
- **`file`** (ChatGPT) — the same tools declare `_meta["openai/fileParams"]: ["file"]`, so a file the user attached in ChatGPT arrives as `{ download_url, file_id, mime_type, file_name }` and `source` can be omitted. The server imports it once per `file_id`.
- **Upload button** — `get_upload_url` carries an MCP App (`ui://vocalingo/upload-file`). Hosts that support MCP Apps (claude.ai, Claude apps, ChatGPT) show a file picker under the call; after the upload it posts a chat message with the `fileId`. Clients without MCP Apps use `curlCommand` or `browserUploadUrl` as before.
- **Tokens** — VocaLingo tokens, the in-app currency you see in the app (not LLM tokens), spent from the same balance.
- **Confirmation** — a call that would cost **50+ tokens** does not start. It returns `status: "confirmation_required"` with the price and a `confirmToken`. The agent asks you; if you agree, it repeats the same call with `confirmToken`. A token is bound to your account and to those exact arguments, valid 30 minutes.
- **`dryRun: true`** — returns only the estimate (plus a `confirmToken` for expensive calls); nothing starts.
- **No double charges** — an identical paid call repeated within 10 minutes returns the same job / result instead of running again.
- **Jobs** — long tools wait ~25 s and return the result if it's ready; otherwise they return a `jobId`. `check_job` then waits up to `waitSeconds` (default 20, max 45) per call. While `status` is `processing`, just call it again.
- **Errors** — `isError: true` with `{ error, code, nextStep }`. `nextStep` tells the agent exactly what to do (upload first, pick a supported language, top up…).
- **Languages** — ISO codes (`es`), English names (`Spanish`) or native/Russian names (`Español`, `испанский`).
- **Result links** are signed for 24 hours. Expired? `check_job` with the same `jobId` returns a fresh one. Everything is also stored in the app history.

---

## Recipes

**Voice message → text**

```
speech_to_text { source: "<fileId or link>" }            → { text, language }
```

**Translate a voice message, keep the speaker's voice**

```
get_upload_url { filename: "voice.ogg" }                  → { fileId, curlCommand }
<run curlCommand>
translate_audio { source: "<fileId>", targetLanguage: "en" }
   → usually the finished result right away (transcript, translation, translatedAudioUrl)
   → or { jobId, status: "processing" } → check_job { jobId }
```

`voice` defaults to `"clone"` — a one-off clone of the speaker from this audio, nothing is saved. Cheap for en, ru, zh, de, fr, es, it, ja, ko, pt (min 3 s of speech). Other target languages use MiniMax (min 10 s, ~180 tokens extra) → you'll get `confirmation_required` first. `voice: "none"` = text only; a `voiceId` from `list_voices` = that saved voice.

**Summarize a YouTube / TikTok / Instagram video**

```
summarize_media { source: "https://youtu.be/…" }          → summary, or jobId → check_job { jobId }
check_job { jobId, include: "transcript" }                → full transcript (paged with offset)
check_job { jobId, include: "segments" }                  → timecoded segments (SRT-like)
```

**Read a text aloud in your own voice**

```
list_voices {}                                            → pick voiceId
text_to_speech { text: "…", voice: "<voiceId>" }          → { audioUrl }
```

No saved voice? `save_voice { name, source }` with a 10–60 s sample (~1 token, the voice also appears in the app). Own voices speak en, ru, zh, de, fr, es, it, ja, ko, pt.

**Subtitles / dubbing (also for a YouTube/TikTok link)**

```
download_media { source: "https://youtu.be/…" }                   → { source: "<file link>" }   (skip for an uploaded file)
caption_video { source: "<file>", targetLanguage: "en" }          → jobId → check_job
translate_video { source: "<file>", targetLanguage: "es" }        → confirmation_required → (after you agree) jobId → check_job
```

**Download a video or its audio**

```
download_media { source: "<link>", kind: "audio" }                → { fileUrl }   (mp3; kind "video" → mp4)
```

**Another voice for a recording**

```
list_voices {}                                                    → stockVoices.elevenlabs
change_voice { source: "<fileId or link>", voice: "George" }      → { resultUrl }
```

**Translate a menu / sign / document photo as an image**

```
translate_image { source: "<image fileId or link>", targetLanguage: "en" }   → { imageUrl }
```

## Don't

- **Don't start the same paid job again** while it's `processing` — keep calling `check_job` with the same `jobId`. (A repeat within 10 minutes just returns the same job anyway.)
- **Don't re-upload** the same file: a `fileId` is valid for 48 hours.
- **Don't pass YouTube/TikTok links to `translate_video` / `caption_video` / `change_voice`** — they need the file: call `download_media` first. Platform links work directly in `summarize_media`.
- **Don't confirm a cost on the user's behalf.**
- **Don't retry on `insufficient_balance`** — top up in the app first.

---

## Files

### `get_upload_url` — free

Creates an upload slot for a local file.

| Param | Type | Description |
|---|---|---|
| `filename` | string | Original filename with extension, e.g. `voice.ogg` (only the extension is used) |

Returns `fileId`, a ready `curlCommand` (PUT), `uploadUrl` (expires in ~2 h), and `browserUploadUrl` — a drag&drop page for users without a shell. Uploaded files are deleted after 48 hours.

Size limits: `translate_audio` 150 MB; `speech_to_text` and `save_voice` 50 MB; video tools and `change_voice` 100 MB; images 20 MB. Extensions: mp3, m4a, wav, ogg, oga, opus, webm, aac, flac, mp4, mov, mkv, avi, m4v, mpeg, mpg, jpg, jpeg, png, webp.

There is no separate "check upload" step: if the file hasn't arrived yet, the tool you call returns `code: "file_not_uploaded"` with what to do.

---

## Voice

### `save_voice` — paid (~1 token)

Clones a voice from 10–60 s of clean speech and saves it to the same library as the app (longer audio is trimmed). Takes ~10–30 s. The voice speaks en, ru, zh, de, fr, es, it, ja, ko, pt.

| Param | Type | Description |
|---|---|---|
| `name` | string | Voice name |
| `source` | string | fileId or link to the sample |
| `language` | string? | Sample language (improves quality) |
| `dryRun` | bool? | Estimate only |

Only clone voices you own or have explicit consent to use.

### `list_voices` — free

`ownVoices` — the user's saved voices (`voiceId`, name, language, status, `usableInTextToSpeech`). `stockVoices` — OpenAI voice names and ElevenLabs stock voices (`voiceId`, name, gender, accent, style) for `text_to_speech` and `change_voice`.

### `delete_voice` — free, destructive

| Param | Type | Description |
|---|---|---|
| `voiceId` | string | From `list_voices` |

Irreversible; tokens spent on the clone are not refunded.

### `text_to_speech` — paid

Voices a text and returns `audioUrl`. The provider is picked from `voice`:

| `voice` | Provider | Text limit |
|---|---|---|
| a `voiceId` from `list_voices`, or omitted (your latest saved voice, else a stock narrator) | **your own voice** (en, ru, zh, de, fr, es, it, ja, ko, pt) | 10 000 chars |
| `cedar`, `marin`, `alloy`, `ash`, `coral`, `echo`, `fable`, `nova`, `onyx`, `sage`, `shimmer` | OpenAI (supports `instructions`) | 4 000 chars |
| an ElevenLabs voice name (`George`) or id from `list_voices` | ElevenLabs | 10 000 chars |

| Param | Type | Description |
|---|---|---|
| `text` | string | Text to speak |
| `voice` | string? | See the table |
| `provider` | string? | Usually omit — inferred from `voice` |
| `speed` | number? | 0.5–2.0 (own voices), 0.7–1.2 (ElevenLabs) |
| `emotion` | string? | Own voices: `happy`, `sad`, `angry`, `neutral`, … |
| `instructions` | string? | OpenAI: free-form style, e.g. `"calm and slow"` |
| `language` | string? | Language of the text; helps your own voice pronounce it right |
| `dryRun` | bool? | Estimate only |
| `confirmToken` | string? | For runs of 50+ tokens |

~11–12 tokens per 1000 characters with your own voice. Long texts may come back as `status: "processing"` + `jobId`.

---

## Speech & audio

### `speech_to_text` — paid (~1 token per 2 min)

| Param | Type | Description |
|---|---|---|
| `source` | string | fileId or link (max 50 MB) |
| `dryRun` | bool? | Estimate only |

Returns `text`, `language`, `durationSeconds`.

### `translate_audio` — paid

Transcript → translation → summary → (by default) voice-over in the speaker's voice.

| Param | Type | Description |
|---|---|---|
| `source` | string | fileId or link (max 150 MB) |
| `targetLanguage` | string | Target language |
| `voice` | string? | `"clone"` (default), `"none"` (text only) or a `voiceId` |
| `dryRun` | bool? | Estimate only |
| `confirmToken` | string? | For runs of 50+ tokens (e.g. MiniMax clone) |

Result (directly or via `check_job`): `transcript`, `translatedText`, `summary`, `translatedAudioUrl`, `stages`.

---

## Video

### `summarize_media` — paid (~0.8 tokens/min)

| Param | Type | Description |
|---|---|---|
| `source` | string | YouTube/TikTok/Instagram/X/Vimeo link, direct media URL, or fileId |
| `summaryLanguage` | string? | Summary language (default — your app language) |
| `dryRun` | bool? | Estimate only |
| `confirmToken` | string? | For very long uploads (50+ tokens) |

Result: title, structured summary (content type, key moments with timecodes, verbatim quotes, takeaway, chapters; Q&A, terms or action items depending on content), a transcript preview, and — via `check_job` `include` — the full transcript and timecoded segments.

### `translate_video` — paid (~240 tokens/min, confirmed)

Dubbing with voice cloning and optional lip-sync. Needs the video **file**. Max 30 minutes; takes 5–30 minutes.

| Param | Type | Description |
|---|---|---|
| `source` | string | fileId or direct file link (max 100 MB) |
| `targetLanguage` | string | Target language |
| `audioOnly` | bool? | Dub the audio track only, no lip-sync (same price, faster) |
| `dryRun` | bool? | Estimate only |
| `confirmToken` | string? | From `confirmation_required` |

### `caption_video` — paid (~1–2 tokens/min)

| Param | Type | Description |
|---|---|---|
| `source` | string | fileId or direct file link (max 100 MB) |
| `targetLanguage` | string? | Translate the subtitles; omit for the original language |
| `style` | string? | `classic-outline` (default), `tiktok-bold`, `boxed`, `minimal`, `cinematic-top`, `neon-cyan`, `yellow-pop`, `subtitle-pill`, `red-punch`, `premium-gold` |
| `dryRun` | bool? | Estimate only |
| `confirmToken` | string? | For long videos (50+ tokens) |

### `download_media` — free (needs a positive balance)

Downloads a video (mp4) or its audio (mp3) from YouTube, TikTok, Instagram, X, Vimeo and most other sites. Returns `fileUrl` (valid 24 h) and the same link as `source` for other tools. Waits up to ~40 s, otherwise returns a `jobId`.

| Param | Type | Description |
|---|---|---|
| `source` | string | Link to the video page or file |
| `kind` | string? | `video` (default) or `audio` |
| `quality` | string? | `360`, `480`, `720` (default), `1080` — video only; other tools accept files up to 100 MB |

### `change_voice` — paid (by duration)

Re-voices speech in an audio or video file (up to 5 min) into an ElevenLabs stock voice, keeping intonation and timing. Video keeps its picture; non-Premium videos get a small watermark. The user's own cloned voices are not supported here.

| Param | Type | Description |
|---|---|---|
| `source` | string | fileId or direct link to the audio/video (max 100 MB) |
| `voice` | string | ElevenLabs voice name (`George`) or `voiceId` from `list_voices` |
| `dryRun` | bool? | Estimate only |
| `confirmToken` | string? | For runs of 50+ tokens |

### `translate_image` — paid (4–6 tokens)

Redraws all text on an image (menu, sign, document, poster, screenshot) in another language, keeping layout, fonts and colors. Takes ~20–40 s. Returns `imageUrl` (valid 7 days) plus a compressed preview as image content, so the agent can check the text. Hosts with MCP Apps (claude.ai, Claude apps, ChatGPT) show the image right in the chat with Save and Open buttons (`ui://vocalingo/image-result`).

| Param | Type | Description |
|---|---|---|
| `source` | string | fileId (jpg, png, webp) or link to the image (max 20 MB) |
| `targetLanguage` | string | Target language |
| `dryRun` | bool? | Estimate only |

---

## Jobs & account

### `check_job` — free, read-only

| Param | Type | Description |
|---|---|---|
| `jobId` | string? | From a job tool (`va_…`, `sm_…`, `dl_…`, `tv_…`, `cv_…`, `vx_…`, `ts_…`). **Omit to list recent jobs.** |
| `waitSeconds` | int? | Wait for completion (default 20, max 45; 0 = return immediately) |
| `include` | string? | For finished transcriptions: `summary` (default: summary + transcript preview), `transcript` (full text, paged), `segments` (timecodes, paged) |
| `offset` | int? | Continue from `nextOffset` of the previous page |

A job that stays `processing` far longer than normal is reported as `failed` with `interrupted: true` — then it's OK to start it again.

### `get_account` — free, read-only

Balance, Premium status and recent charges/top-ups.

| Param | Type | Description |
|---|---|---|
| `transactions` | int? | How many recent transactions (default 10, 0 = none) |

---

## Errors

Every error is `isError: true` with a machine-readable `code` and a `nextStep` for the agent:

```json
{ "error": "\"/Users/me/voice.ogg\" looks like a local file path. The VocaLingo server cannot read files on your computer.",
  "code": "local_path",
  "nextStep": "Call get_upload_url, upload the file with the returned curl command (or give the user browserUploadUrl), then pass the returned fileId as source." }
```

Common codes: `local_path`, `file_not_uploaded`, `file_too_large`, `not_a_media_file`, `not_an_image`, `media_too_long`, `platform_link_not_supported`, `unknown_language`, `invalid_voice`, `voice_not_found`, `audio_too_short`, `too_many_jobs`, `insufficient_balance`, `balance_unavailable` (temporary — retry in a minute), `internal_error`.

Rate limits: 60 requests/min per account; max 3 concurrent jobs.
