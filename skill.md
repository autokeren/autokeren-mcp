# AutoKeren MCP — Agent Skill

Autonomous AI video production: script → images → TTS → 1080p MP4, plus verified YouTube footage sourcing, Gemini QA, and premium Remotion renders. 40 tools, one credit wallet. This document teaches you everything — read it once, then produce videos end-to-end.

## Connect

- **Endpoint**: `https://mcp.autokeren.com` (MCP Streamable HTTP, JSON-RPC 2.0)
- **Auth**: `Authorization: Bearer ak_...` header on every call (self-service keys: studio.autokeren.com/settings)
- **Protocol versions**: 2024-11-05, 2025-03-26, 2025-06-18 (echoed on initialize; JSON-RPC batch accepted but not required)
- **Free, no key needed**: get_workflow, list_models, estimate_cost, suggest_packaging
- **Quick test**: POST with `{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"list_models","arguments":{}}}`

## Golden Rules

1. **Call `get_workflow` first** in every session — it returns the full production playbook.
2. **Budget before you spend**: `list_models` → `estimate_cost` → show the user the cost. Confirm before expensive ops (video clips = 60–740 CR per 5s, duration-scaled).
3. **Never trust an unverified media URL.** Every upload tool (fetch_to_r2, yt_dlp_download, cut_video) returns `verified: true` only after the object is confirmed in R2. set_scene_media HARD-REJECTS dead URLs (404/410).
4. **Scene format**: 20–25 words of narration per ~9 second scene. `duration` is in SECONDS (1–60). Values >600 are treated as milliseconds and normalized.
5. **Image prompts: NEVER use brand names** — moderation rejects them. Describe generically ("sleek German luxury sedan", not "BMW i7").
6. **After every render**: `get_status` → `verify_render` → `check_video`. The QA report classifies **real moving footage vs still-images-with-camera-motion** per time range — do not accept a video that was supposed to contain real footage if it only contains stills.
7. **Expensive calls auto-refund on failure**, but wasted wall-clock time does not — verify inputs first.

## Production Workflow (standard)

```
get_workflow → list_models → estimate_cost → create_project
→ load_scenes (your narrations + visual prompts + animations)
→ generate_assets (images + TTS — credits deduct here)
→ render_video → get_status (poll) → verify_render → check_video
→ suggest_packaging (titles/thumbnail/SEO — free)
```

Full free-tier production: Cloudflare models (script, image) + Aura TTS = **0 CR** — verified working.

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
prepare_remotion (validate composition — free-ish sanity check)
→ render_remotion (submits → returns poll_id INSTANTLY, 15 CR)
→ render_remotion_result({poll_id}) every 10–15s
   → "processing" (keep polling) | "completed" (MP4 URL) | "failed" (auto-refunded)
```

Same async pattern as elevenlabs_result. Renders typically take 1–5 min.

## Tool Catalog (40 tools)

### Pipeline

- `get_workflow` — Returns the complete AutoKeren production playbook: step-by-step workflow, scene format, pricing guide, tips, and troubleshooting.
- `list_models` — List AutoKeren AI models with per-model credit pricing (script/context/image/tts/video categories).
- `estimate_cost` — Estimate total credits (and approx IDR) for a production BEFORE generating.
- `get_credits` — Get the AutoKeren credit balance, subscription plan, and recent ledger.
- `topup_credits` — Create a real payment link (Tripay: QRIS/virtual account, IDR) to top up credits.
- `create_project` — Create a new video production project.
- `load_scenes` — Load your authored scenes into the project.
- `generate_assets` — Queue asset generation (images + TTS voice) for every scene.
- `render_video` — Queue the final MP4 render (1080p, loudness-normalized).
- `get_status` — Get project + render job status, progress and the final video URL when complete.
- `list_videos` — List recent video projects (id, title, status, video URL).
- `set_scene_media` — Set video or image URL for a specific scene (footage injection).
- `attach_scene_media_batch` — Batch-attach media to multiple scenes in ONE call (footage injection at scale).
- `reset_render` — Reset render_status for scenes so they get re-rendered.
- `verify_render` — Verify the final render output: project status, render job status, R2 output file metadata (size + etag/md5).
- `get_scenes` — List ALL scenes of a project: narration (script_text), visual_prompt, image/video/audio URLs, duration, animation, render_status + summary.
- `get_render_errors` — Deep render failure inspection: recent render jobs with error messages, plus every FAILED/stuck scene with its media URLs and suggested fixes.
- `get_usage` — Per-model usage report from your credit ledger: which models/stages you used, how many calls, credits spent per model, over N days (default 30).

### Media Generation

- `generate_image` — Playground: generate a single image with any image model (Flux, Gemini image.
- `generate_speech` — Playground: generate speech or a sound effect with any TTS model standalone.
- `generate_video_clip` — Playground: generate a short video clip with a video model (Veo, Seedance) standalone.
- `elevenlabs_image` — ElevenLabs image generation (multi-provider): text-to-image or image-to-image (pass image_url to edit/restyle).
- `elevenlabs_video` — ElevenLabs video generation (multi-provider): text-to-video OR image-to-video (pass image_url as start_frame — e.
- `elevenlabs_result` — Poll an ElevenLabs generation.
- `elevenlabs_credits` — Check the ElevenLabs subscription: remaining audio characters and image/video credits.

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
