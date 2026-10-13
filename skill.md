# AutoKeren MCP — Agent Skill

Autonomous AI video production: script → images → TTS → 1080p MP4, plus verified YouTube footage sourcing, Gemini QA, and premium Remotion renders. 48 tools, one credit wallet. This document teaches you everything — read it once, then produce videos end-to-end.

## Connect

- **Endpoint**: `https://mcp.autokeren.com` (MCP Streamable HTTP, JSON-RPC 2.0)
- **Auth**: `Authorization: Bearer ak_...` header on every call (self-service keys: studio.autokeren.com/settings)
- **Protocol versions**: 2024-11-05, 2025-03-26, 2025-06-18 (echoed on initialize; JSON-RPC batch accepted but not required)
- **Free, no key needed**: get_workflow, list_models, estimate_cost, suggest_packaging
- **Quick test**: POST with `{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"list_models","arguments":{}}}`

## Voice Routing (TTS) — 45 verified voices

Pass a voice **NAME (case-insensitive)** or raw **voice_id** to `generate_speech` or any `voice` param. Unknown names are **REJECTED with a clear error** — never a silent Adam fallback.

- **English (US):** Adam, Antoni, Brian, Dave, Callum, Harry, James, Jeremy, Liam, Michael, Sam, Sarah, Rachel, Nicole, Charlie
- **English (UK):** Daniel, Freya, George, Charlotte, Dorothy, Jessie
- **Deutsch (native):** Otto, Leon Stern, Leonie, Christian, Lea, Joha
- **Multilingual (cocok Bahasa Indonesia):** Clyde, Arnold, Domi, Ethan, Fin, Mimi, Matilda, Grace, Giovanni, Serena, Thomas, Drew, Elli, Emily, Gigi, Glinda, Matthew, Patrick

Models: multilingual v2 (29 languages incl. Indonesian + German), Flash v2.5 (32 languages), Eleven v4 (90+ languages).
Call `list_voices` for the live catalog. ⚠️ ElevenLabs Default voices expire Dec 31 2026 — after that, pass voice_ids from your workspace.

**Persona → Voice auto-mapping** (no manual voice needed for automotive content):
- `aggressive` persona + German → Leon Stern | + English → Callum | + Indonesian → Arnold
- `nuanced` persona + German → Otto | + English → George | + Indonesian → Matilda
- createScript (videoType `automotive`) stamps `scenePersona` into every scene automatically.
- Voice priority per scene: `sceneVoice` (explicit) > persona-mapped voice > global `voice`. Set `scenePersona` per scene via load_scenes (e.g. film characters), or `voicePersona` via project metadata / generate-assets options.

## Golden Rules

1. **Call `get_workflow` first** in every session — it returns the full production playbook.
2. **Budget before you spend**: `list_models` → `estimate_cost` → show the user the cost. Confirm before expensive ops (video clips = 60–740 CR per 5s, duration-scaled).
3. **Never trust an unverified media URL.** Every upload tool (fetch_to_r2, yt_dlp_download, cut_video) returns `verified: true` only after the object is confirmed in R2. set_scene_media HARD-REJECTS dead URLs (404/410).
4. **Scene format**: 20–25 words of narration per ~9 second scene. `duration` is in SECONDS (1–60). Values >600 are treated as milliseconds and normalized.
5. **Image prompts: NEVER use brand names** — moderation rejects them. Describe generically ("sleek German luxury sedan", not "BMW i7").
6. **After every render**: `get_status` → `verify_render` → `check_video`. The QA report classifies **real moving footage vs still-images-with-camera-motion** per time range — do not accept a video that was supposed to contain real footage if it only contains stills.
7. **Expensive calls auto-refund on failure**, but wasted wall-clock time does not — verify inputs first.

## Worked Example — EN-US Channel Car Video (copy this pattern)

The proven full-production pattern for US-market automotive channel videos — research freshness gate, per-photo Gemini verification, official-film hook windows, sticker/countdown script format, US number formats, stretch tiers, loop-gate, clean-voice mastering, and the 7 QA gates. Reference production scored 9.6/10.

Read it: **https://mcp.autokeren.com/examples/example_us_channel_car.md**

## Production Workflow (standard)

```
get_workflow → list_models → estimate_cost → create_project
→ load_scenes (your narrations + visual prompts + animations)
→ generate_assets (images + TTS — credits deduct here)
→ render_video → get_status (poll) → verify_render → check_video
→ suggest_packaging (titles/thumbnail/SEO — free)
```

Full free-tier production: Cloudflare models (script, image) + Aura TTS = **0 CR** — verified working.

**Formats**: `create_project({aspect_ratio: "9:16"})` → portrait canvas + portrait images (Shorts). `render_video({resolution: "4k"})` → 3840x2160 (also: 1080p default, 2k 2560x1440).

## Limits & Access Rules (enforced server-side)

- **Daily cap**: 700 expensive-tool calls per user per day (generation/sourcing/QA/render tools). Cheap tools (get_status, get_scenes, list_models, estimate_cost...) are uncapped. The cap resets at midnight UTC; hitting it returns a clear error.
- **Project ownership**: every project-scoped tool verifies the project belongs to YOUR key's account. Other users' projects return "Access denied" — there is no way around this by design.
- **R2 key namespaces**: `projects/<pid>/...` keys must target YOUR projects; `members/<uid>/...` keys must match your uid. Any other prefix (e.g. `scratch/...`) is allowed — use unique keys.
- **Dead-URL guard**: set_scene_media rejects 404/410 media URLs — upload through fetch_to_r2 / yt_dlp_download / cut_video first (all return `verified: true`).
- **Attached media is sacred**: generate_assets NEVER overwrites media you attached via set_scene_media / attach_scene_media_batch — it only fills what's missing (null image → image, null audio → TTS). You can safely attach official assets BEFORE or AFTER generate_assets.
- **Asset Library**: every verified MCP output (fetch_to_r2, yt_dlp, cut_video, slides) is auto-indexed into the dashboard Asset Library — browse it with `list_assets({asset_type, search})` and re-attach any asset to scenes for revisions.

### ReviewSceneData schema (for render_remotion_slide)

```json
{
  "kind": "chapter | footage | stats | compare",
  "audio": "https://... (public TTS/audio URL)",
  "frames": 149,
  "chapter": {"title": "MIT TRANSFORM", "subtitle": "optional", "bgImage": "https://... optional"},
  "coverImage": "https://... full-bleed background image",
  "coverVideo": "https://... full-bleed background video",
  "footage": {"videoSrc": "https://...", "imgSrc": "https://...", "caption": "text under frame"},
  "stats": [{"value": "621 HP", "label": "Power", "delay": 15, "color": "#8b5cf6"}],
  "lowerThird": {"model": "Denza D9", "price": "$50,000", "spec": "EV SUV"},
  "compare": {"leftImg": "https://...", "rightImg": "https://...", "leftLabel": "A", "rightLabel": "B", "leftPrice": "$40k", "rightPrice": "$50k"},
  "ticker": ["string1", "string2"],
  "tickerPosition": "bottom | top",
  "theme": {"accent": "#8b5cf6", "textColor": "#ffffff", "subtitleColor": "#9ca3af", "titleSize": 72, "bg": "#0a0e1a"},
  "compare": {"leftImg": "https://...", "rightImg": "https://...", "leftLabel": "A", "rightLabel": "B", "leftPrice": "$40k", "rightPrice": "$50k", "leftProgress": 45, "rightProgress": 85}
}
```
- `chapter`: object with `title` (required), `subtitle`, `bgImage`. Shorthand: string also works — `"chapter": "TITLE"` auto-converts.
- `stats`: array of `{value, label, delay}` — delay in frames (15 ≈ 0.5s). Shorthand: array of strings also works.
- `lowerThird`: `{model, price, spec}` — slide-in info card from bottom-left.
- `kind` determines the layout: chapter = title card with bg, footage = framed video/image, stats = centered stat cards, compare = side-by-side.
- **Honest failures**: extract_frames refunds if frames never reach R2; render_remotion times out after 20 min of processing (full refund); every failed generation auto-refunds. If a tool reports success, its outputs exist.

### Runway image models (paid)

- Pure text-to-image: `gen4_image`, `muse_image`, `gpt_image_2`, `gemini_image3_pro`, `seedream5_pro`...
- `runway-gen4_image_turbo` is a **reference-based EDITING model** — it REQUIRES 1-3 `reference_images` URLs (per the official API spec). Without them you get a clear error + auto-refund.
- Aspect ratios are auto-mapped per model from the official OpenAPI spec (16:9 → muse `1920:1280`, turbo `1920:1080`, etc.) — pass OpenAI-style ratios (`16:9`, `9:16`, `1:1`, `4:3`) and the adapter picks the closest allowed value.

## Footage Pipeline (real b-roll — fully verified)

```
youtube_search → yt_dlp_download (verified in R2)
→ index_shots (Gemini watches the video, indexes every shot with timestamps)
→ cut_video(segments from the shot index, separate: true)
   → returns UNIQUE clip URL per segment (verified)
→ attach_scene_media_batch (up to 50 scenes in ONE call)
→ render_video → check_video (confirms real footage, not stills)
```

- `cut_video` without `separate:true` concatenates segments into ONE clip — fine for a single scene, wrong for 20.
- WebMCP: agents visiting autokeren.com can also upload via `/upload?key=ak_...`.

## Still-Image Animations (per scene, via load_scenes)

`zoom_in` | `zoom_out` | `pan_left` | `pan_right` | `pan_up` | `pan_down` | `rotate_cw` | `rotate_ccw` | `shake` | `fade_in` | `static`

Match the mood: rotate_cw for product reveals, shake for impact moments, pan_left for landscape sweeps. Default: zoom_in. Want REAL motion from a still instead? Use `elevenlabs_video` (image-to-video, Veo 3.1/Seedance) → `elevenlabs_result` → `set_scene_media`.

## Premium Render (ReviewLayout — async)

```
One-shot: prepare_remotion (validate) → render_remotion (15 CR, poll_id instantly).
Per-slide INCREMENTAL (recommended for long productions & Shorts): render_remotion_slide (ONE scene, full motion graphics, 0.5 CR, 16:9/9:16/1:1) → QA that slide (analyze_media) → fix & re-render → repeat → stitch_remotion_slides (1 CR).
→ render_remotion_result({poll_id}) every 10–15s
   → "processing" (keep polling) | "completed" (MP4 URL) | "failed" (auto-refunded)

### Production Standard: Premium Video with ElevenLabs + AI Video + Motion Graphics

Battle-tested pipeline for a complete premium video using ALL ElevenLabs products. Every scene has motion: Ken Burns on images, AI video backgrounds, Remotion overlays.

**Step 1 — TTS + exact frames:**
  generate_speech({model_id: "elevenlabs-eleven_multilingual_v2", voice: "Adam", text: "..."})
  get_audio_duration({audio_url: "..."}) → returns EXACT frames — never guess

**Step 2 — Visuals per scene (pick one):**
  - Ken Burns image (title scenes): elevenlabs_image → pass as chapter.bgImage
  - AI video background (cinematic scenes): elevenlabs_video (duration_secs: 4|6|8) → pass as coverVideo
  - AI video is auto-muted + auto-slowed (coverVideoRate = video_dur / scene_dur)

**Step 3 — Render per slide:**
  render_remotion_slide({scene: {kind: "chapter", audio: "TTS url", frames: EXACT,
    chapter: {title: "...", subtitle: "...", bgImage: "ElevenLabs image"},
    // OR: coverVideo: "ElevenLabs AI video url", coverVideoRate: 0.5,
    stats: [{value: "90+", label: "Languages", delay: 15}],
    lowerThird: {model: "...", price: "...", spec: "..."},
    ticker: ["tagline"], theme: {accent: "#e8a23c"}
  }, output_r2key: "my-video/s1.mp4"})

**Step 4 — Stitch:**
  stitch_remotion_slides({slides: [url1...urlN], output_r2key: "my-video/final.mp4"})

**Scene type guide:**
  | Scene type | Visual | Overlays |
  | Opening/Title | elevenlabs_image (Ken Burns) | chapter card |
  | Problem/Stats | elevenlabs_video (AI motion) | stats cards |
  | Solution/Demo | elevenlabs_image (Ken Burns) | chapter + footage |
  | Feature/How | elevenlabs_video (AI motion) | chapter card |
  | Proof/Results | elevenlabs_image (Ken Burns) | chapter card |
  | Closing/CTA | elevenlabs_image (Ken Burns) | chapter + lowerThird + ticker |

**coverVideoRate:** video_duration / scene_duration (e.g. 6s video in 12s scene = 0.5).
AI video is auto-muted — only ElevenLabs TTS is audible.
```

Same async pattern as elevenlabs_result. Renders typically take 1–5 min.

## Tool Catalog (48 tools)

### Pipeline

- `get_workflow` — Returns the complete AutoKeren production playbook: step-by-step workflow, scene format, pricing guide, tips, and troubleshooting.
- `list_models` — List AutoKeren AI models with per-model credit pricing (script/context/image/tts/video categories).
- `estimate_cost` — Estimate total credits (and approx IDR) for a production BEFORE generating.
- `get_credits` — Get the AutoKeren credit balance, subscription plan, and recent ledger.
- `topup_credits` — Create a real payment link (Tripay: QRIS/virtual account, IDR) to top up credits.
- `create_project` — Create a new video production project.
- `load_scenes` — Load your authored scenes into the project.
- `generate_assets` — Queue asset generation (images + TTS voice) for every scene.
- `render_video` — Queue the final MP4 render (loudness-normalized).
- `get_status` — Get project + render job status, progress and the final video URL when complete.
- `list_videos` — List recent video projects (id, title, status, video URL).
- `set_scene_media` — Set video or image URL for a specific scene (footage injection).
- `attach_scene_media_batch` — Batch-attach media to multiple scenes in ONE call (footage injection at scale).
- `reset_render` — Reset render_status for scenes so they get re-rendered.
- `verify_render` — Verify the final render output: project status, render job status, R2 output file metadata (size + etag/md5).
- `get_scenes` — List ALL scenes of a project: narration (script_text), visual_prompt, image/video/audio URLs, duration, animation, render_status + summary.
- `get_render_errors` — Deep render failure inspection: recent render jobs with error messages, plus every FAILED/stuck scene with its media URLs and suggested fixes.
- `get_usage` — Per-model usage report from your credit ledger: which models/stages you used, how many calls, credits spent per model, over N days (default 30).

### Film Mode (cinematic_film)

- `generate_character_sheet` — FILM MODE: generate a multi-angle character reference sheet (front/3-4/side/back views + 3 expression studies) from a visual description.
- `extract_last_frame` — FILM MODE: extract the FINAL frame of a scene clip → JPG in R2.
- `restart_container` — OPS: force-restart a director container DO (default: tool-runner).

### Media Generation

- `generate_image` — Playground: generate a single image with any image model (Flux, Gemini image.
- `generate_speech` — Playground: generate speech or a sound effect.
- `generate_video_clip` — Playground: generate a short video clip with a video model (Veo, Seedance) standalone.
- `elevenlabs_image` — ElevenLabs image generation (multi-provider): text-to-image or image-to-image (pass image_url to edit/restyle).
- `elevenlabs_video` — ElevenLabs video generation (multi-provider): text-to-video OR image-to-video (pass image_url as start_frame — e.
- `elevenlabs_result` — Poll an ElevenLabs generation.
- `elevenlabs_credits` — Check the ElevenLabs subscription: remaining audio characters and image/video credits.
- `list_voices` — The full TTS voice catalog (45 verified voices grouped by language: EN-US, EN-UK, German native, Indonesian-capable multilingual) plus workspace voices.

### Research & Sourcing

- `fetch_to_r2` — Download any public URL (image/video/audio/file) and save directly to R2.
- `youtube_search` — Search YouTube for videos.
- `yt_dlp_download` — Download a YouTube video via yt-dlp at up to 1080p → upload to R2.
- `index_shots` — Gemini watches a video and indexes EVERY distinct camera shot with precise timestamps (start, end, view description).
- `extract_frames` — Extract still frames from a video at given timestamps → upload to R2 as JPEGs.
- `cut_video` — Cut one or more segments from a video → upload to R2.

### AI Vision & QA

- `check_vision` — Analyze an image using Cloudflare Workers AI vision.
- `analyze_media` — Deep video/image understanding via Google Gemini (default gemini-3.
- `check_video` — DEFAULT QA PIPELINE: full-video audit via Gemini native video understanding (watches the ENTIRE video at ~1fps with audio).
- `review_video` — FINAL REVIEW: Gemini watches the ENTIRE video and gives a production score /10 (visual, audio, accuracy, viral, anti-slop breakdown), upload-worthy verdict, top 3 improvements.

### Packaging & Upload

- `suggest_packaging` — YouTube packaging assistant: 3 A/B title options with CTR rationale, thumbnail concept (text overlay + image prompt), SEO description, tags.
- `prepare_remotion` — Prepares a Remotion ReviewLayout composition: validates scene data, calculates total duration, and returns the exact steps + CLI command to render.
- `get_presigned_upload` — Generates a presigned R2 upload URL for DIRECT-TO-R2 uploads (bypasses the 100MB Worker body limit).

### Premium Render

- `render_remotion` — ASYNC one-call Remotion render via container (absurd-engine, kept warm 2h).
- `render_remotion_slide` — PREMIUM PER-SLIDE RENDER: render ONE scene with full Remotion motion graphics (chapter/stats/lower-third animations — same quality as render_remotion, just this scene's duration).
- `stitch_remotion_slides` — FINAL STEP of the per-slide pipeline: ffmpeg-concat your rendered slides (in order) into the final MP4 → R2.
- `get_audio_duration` — Probe a public audio URL (WAV/MP3) and return the EXACT duration + frame count at 30fps.
- `list_assets` — Browse YOUR Asset Library: images, audio, and videos from previous generations, uploads, and pipeline outputs (fetch_to_r2, yt_dlp, cut_video, slides — all auto-indexed).
- `render_remotion` — ASYNC one-call Remotion render via container (absurd-engine, kept warm 2h).
- `render_remotion_result` — Poll an async render_remotion job.

## Pricing Snapshot

| Stage | Free (0 CR) | Paid |
|-------|-------------|------|
| Script | @cf/zai-org/glm-5.3-flash | gemini 0.05–0.1 CR |
| Image | @cf/black-forest-labs/flux-2-klein-9b | gemini-3-pro-image 8 CR |
| TTS | @cf/deepgram/aura-2-en (EN) | ElevenLabs 2–10 CR |
| Video clip | — | Runway models 60–740 CR per 5s, duration-scaled (veo3.1 700, seedance2_5 740, seedance2_fast 600, gen4.5 250, seedance2 60) |
| fetch_to_r2 / presigned | 1 CR | — |
| cut_video | 3 CR | — |
| analyze/check/review (Gemini) | 1–6 CR | — |
| render_remotion | 15 CR (auto-refund on failure) | — |

All prices live in the credit_configs DB table — call `list_models` for live values. `get_usage` answers "where did my credits go" per model.

## Hard-Won Rules (from live agent audits)

- A media URL that was never verified IS a dead URL. If a tool response lacks `verified: true`, treat it as suspect.
- QA scores lie if you don't ask the right question: always check `media_type_report` — "high score + all stills" = FAIL if real footage was requested.
- Batch attachments: use `attach_scene_media_batch`; JSON-RPC batch is NOT supported by most MCP clients (spec 2025-06-18 removed it) — don't rely on it.
- `render_remotion` returns INSTANTLY — never wait inside a single call; poll with `render_remotion_result`.
