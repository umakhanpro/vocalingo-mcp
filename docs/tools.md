# VocaLingo MCP — Tools Reference

All 12 tools of the VocaLingo MCP server (`https://nodejs.vocalingo.app/mcp`).

Conventions:

- **`source`** — every tool that takes a file has one `source` parameter: either a **fileId** from `get_upload_url` or a **public https link** (YouTube/TikTok/Instagram for `summarize_media`, a Telegram Bot API file link, any direct file URL). A local path like `/Users/me/voice.ogg` is rejected with instructions to upload it first.
- **Credits** — the same tokens you see in the VocaLingo app, spent from the same balance.
- **Confirmation** — a call that would cost **50+ credits** does not start. It returns `status: "confirmation_required"` with the price and a `confirmToken`. The agent asks you; if you agree, it repeats the same call with `confirmToken`. A token is bound to your account and to those exact arguments, valid 30 minutes.
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

`voice` defaults to `"clone"` — a one-off clone of the speaker from this audio, nothing is saved. Cheap for en, ru, zh, de, fr, es, it, ja, ko, pt (min 3 s of speech). Other target languages use MiniMax (min 10 s, ~150 credits extra) → you'll get `confirmation_required` first. `voice: "none"` = text only; a `voiceId` from `list_voices` = that saved voice.

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

No saved voice? `save_voice { name, source }` with a 10–60 s sample (150 credits → confirmation first).

**Subtitles / dubbing**

```
caption_video { source: "<fileId>", targetLanguage: "en" }       → jobId → check_job
translate_video { source: "<fileId>", targetLanguage: "es" }     → confirmation_required → (after you agree) jobId → check_job
```

## Don't

- **Don't start the same paid job again** while it's `processing` — keep calling `check_job` with the same `jobId`. (A repeat within 10 minutes just returns the same job anyway.)
- **Don't re-upload** the same file: a `fileId` is valid for 48 hours.
- **Don't pass YouTube/TikTok links to `translate_video` / `caption_video`** — they need the video file. Platform links work in `summarize_media`.
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

Size limits: `translate_audio` 150 MB; `speech_to_text` and `save_voice` 50 MB; video tools 100 MB. Extensions: mp3, m4a, wav, ogg, oga, opus, webm, aac, flac, mp4, mov, mkv, avi, m4v, mpeg, mpg.

There is no separate "check upload" step: if the file hasn't arrived yet, the tool you call returns `code: "file_not_uploaded"` with what to do.

---

## Voice

### `save_voice` — paid (150 credits, always confirmed)

Clones a voice from 10–60 s of clean speech and saves it to the library (longer audio is trimmed). Usually done in ~10 s; otherwise returns a `jobId` for `check_job`.

| Param | Type | Description |
|---|---|---|
| `name` | string | Voice name |
| `source` | string | fileId or link to the sample |
| `language` | string? | Sample language (improves quality) |
| `dryRun` | bool? | Estimate only |
| `confirmToken` | string? | From the `confirmation_required` response |

Only clone voices you own or have explicit consent to use.

### `list_voices` — free

Saved voices with `voiceId`, name, language, status and `usableInTextToSpeech`.

### `delete_voice` — free, destructive

| Param | Type | Description |
|---|---|---|
| `voiceId` | string | From `list_voices` |

Irreversible; clone credits are not refunded.

### `text_to_speech` — paid

Voices a text and returns `audioUrl`. The provider is picked from `voice`:

| `voice` | Provider | Text limit |
|---|---|---|
| a `voiceId` from `list_voices`, or omitted (your latest saved voice, else a stock narrator) | MiniMax — **your own voice** | 30 000 chars |
| `cedar`, `marin`, `alloy`, `ash`, `coral`, `echo`, `fable`, `nova`, `onyx`, `sage`, `shimmer` | OpenAI (supports `instructions`) | 4 000 chars |
| a 20-character ElevenLabs voice id | ElevenLabs | 10 000 chars |

| Param | Type | Description |
|---|---|---|
| `text` | string | Text to speak |
| `voice` | string? | See the table |
| `provider` | string? | Usually omit — inferred from `voice` |
| `speed` | number? | 0.5–2.0 (own voices), 0.7–1.2 (ElevenLabs) |
| `emotion` | string? | Own voices: `happy`, `sad`, `angry`, `neutral`, … |
| `instructions` | string? | OpenAI: free-form style, e.g. `"calm and slow"` |
| `dryRun` | bool? | Estimate only |
| `confirmToken` | string? | For runs of 50+ credits |

~10 credits per 1000 characters with your own voice. Long texts may come back as `status: "processing"` + `jobId`.

---

## Speech & audio

### `speech_to_text` — paid (~1 credit per 2 min)

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
| `confirmToken` | string? | For runs of 50+ credits (e.g. MiniMax clone) |

Result (directly or via `check_job`): `transcript`, `translatedText`, `summary`, `translatedAudioUrl`, `stages`.

---

## Video

### `summarize_media` — paid (~0.42 credits/min)

| Param | Type | Description |
|---|---|---|
| `source` | string | YouTube/TikTok/Instagram/X/Vimeo link, direct media URL, or fileId |
| `summaryLanguage` | string? | Summary language (default — your app language) |
| `dryRun` | bool? | Estimate only |
| `confirmToken` | string? | For very long uploads (50+ credits) |

Result: title, structured summary (content type, key moments with timecodes, verbatim quotes, takeaway, chapters; Q&A, terms or action items depending on content), a transcript preview, and — via `check_job` `include` — the full transcript and timecoded segments.

### `translate_video` — paid (~220 credits/min, confirmed)

Dubbing with voice cloning and optional lip-sync. Needs the video **file**. Max 30 minutes; takes 5–30 minutes.

| Param | Type | Description |
|---|---|---|
| `source` | string | fileId or direct file link (max 100 MB) |
| `targetLanguage` | string | Target language |
| `audioOnly` | bool? | Dub the audio track only, no lip-sync (same price, faster) |
| `dryRun` | bool? | Estimate only |
| `confirmToken` | string? | From `confirmation_required` |

### `caption_video` — paid (~1–2 credits/min)

| Param | Type | Description |
|---|---|---|
| `source` | string | fileId or direct file link (max 100 MB) |
| `targetLanguage` | string? | Translate the subtitles; omit for the original language |
| `style` | string? | `classic-outline` (default), `tiktok-bold`, `boxed`, `minimal`, `cinematic-top`, `neon-cyan`, `yellow-pop`, `subtitle-pill`, `red-punch`, `premium-gold` |
| `dryRun` | bool? | Estimate only |
| `confirmToken` | string? | For long videos (50+ credits) |

---

## Jobs & account

### `check_job` — free, read-only

| Param | Type | Description |
|---|---|---|
| `jobId` | string? | From a job tool (`va_…`, `sm_…`, `dl_…`, `tv_…`, `cv_…`, `vc_…`, `ts_…`). **Omit to list recent jobs.** |
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

Common codes: `local_path`, `file_not_uploaded`, `file_too_large`, `not_a_media_file`, `platform_link_not_supported`, `unknown_language`, `invalid_voice`, `voice_not_found`, `audio_too_short`, `too_many_jobs`, `insufficient_balance`, `balance_unavailable` (temporary — retry in a minute), `internal_error`.

Rate limits: 60 requests/min per account; max 3 concurrent jobs.
