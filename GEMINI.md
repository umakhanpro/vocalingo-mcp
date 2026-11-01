# VocaLingo

The `vocalingo` MCP server works with speech, audio and video: summarize a YouTube, TikTok or Instagram video, transcribe a voice message, translate speech keeping the speaker's voice, read text in the user's own cloned voice, change the voice in a recording, add subtitles, dub a video, redraw the text on an image in another language.

- The first call opens the browser to sign in to VocaLingo. Results also appear in the user's VocaLingo app.
- Local files: call `get_upload_url`, run the returned `curlCommand` with the file path, then pass the `fileId` as `source`. Links go into `source` directly.
- Tools spend VocaLingo tokens from the user's account. A call that costs 50 tokens or more returns `confirmation_required` with the price: ask the user, and only after a yes repeat the same call with `confirmToken`.
- Long jobs return a `jobId`: keep calling `check_job` with the same id, never start the job again.
- Every error has `nextStep`: follow it.

Step-by-step recipes are in the `skills/` folder of this extension.
