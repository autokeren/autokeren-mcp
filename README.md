# AutoKeren MCP — AI Video Studio for Agents

![MCP](https://img.shields.io/badge/MCP-Streamable_HTTP-6E56CF) ![Tools](https://img.shields.io/badge/tools-40-8b5cf6) ![Protocol](https://img.shields.io/badge/protocol-2024--11--05%20%7C%202025--03--26%20%7C%202025--06--18-blue) ![Free tier](https://img.shields.io/badge/free_tier-0_CR-10b981)

**Give any AI agent a complete, autonomous video production studio.**

AutoKeren MCP turns Claude Code, opencode, Cursor, Codex, or any MCP client into an end-to-end video producer: write the script, generate images, voice the narration, source real YouTube footage, QA the result with Gemini, and ship a finished 1080p MP4 — **all from a single MCP server**.

- **Endpoint**: `https://mcp.autokeren.com` (Streamable HTTP)
- **Auth**: self-service API keys — [studio.autokeren.com/settings](https://studio.autokeren.com/settings)
- **Agent skill (one URL, everything)**: [mcp.autokeren.com/skill.md](https://mcp.autokeren.com/skill.md)
- **Landing**: [mcp.autokeren.com](https://mcp.autokeren.com)

## Quickstart

**Claude Code:**
```bash
claude mcp add --transport http autokeren https://mcp.autokeren.com \
  --header "Authorization: Bearer ak_YOUR_KEY"
```

**opencode** (`~/.config/opencode/opencode.json`):
```json
{
  "mcp": {
    "autokeren": {
      "type": "remote",
      "url": "https://mcp.autokeren.com",
      "headers": { "Authorization": "Bearer ak_YOUR_KEY" }
    }
  }
}
```

**Cursor / Windsurf** (MCP Settings → JSON):
```json
{
  "mcpServers": {
    "autokeren": {
      "url": "https://mcp.autokeren.com",
      "headers": { "Authorization": "Bearer ak_YOUR_KEY" }
    }
  }
}
```

**Any HTTP client:**
```bash
curl -X POST https://mcp.autokeren.com \
  -H "Authorization: Bearer ak_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call",
       "params":{"name":"get_workflow","arguments":{}}}'
```

Then just tell your agent: *"Read https://mcp.autokeren.com/skill.md and follow it."*

## The 40 Tools

| Category | What it does |
|---|---|
| **Pipeline** (18) | `get_workflow`, `estimate_cost`, `create_project`, `load_scenes`, `generate_assets`, `render_video`, `set_scene_media`, `attach_scene_media_batch` (50 scenes/call), `verify_render`, `get_render_errors`, `get_usage`... — full production + billing telemetry |
| **Media Generation** (7) | `generate_image`, `generate_speech`, `generate_video_clip` (Veo/Seedance), ElevenLabs image/video flows with async polling |
| **Research & Sourcing** (6) | `youtube_search`, `yt_dlp_download`, `index_shots` (Gemini watches & timestamps every shot), `cut_video` (per-scene clips via `separate:true`), `extract_frames`, `fetch_to_r2` |
| **AI Vision & QA** (4) | `check_vision` (free), `analyze_media`, `check_video` — full-video audit that **classifies real footage vs AI stills per time range**, `review_video` (production score /10) |
| **Packaging & Upload** (3) | `suggest_packaging` (free A/B titles + thumbnail + SEO), `get_presigned_upload` (bypass 100MB limits), `prepare_remotion` |
| **Premium Render** (2) | `render_remotion` (async submit → poll_id instantly) + `render_remotion_result` (container Remotion renders) |

## Why agents love it

- **Verified uploads everywhere** — every media URL the server returns is HEAD-verified in R2 (`verified: true`); `set_scene_media` hard-rejects dead URLs (404/410). No more silent AI-image fallbacks.
- **Auto-refund on failure** — credits never leak on failed generations; the ledger records every refund with a reason.
- **Honest QA** — `check_video` reports `media_type_report`: which time ranges contain real moving footage vs still images with camera motion. Your agent can't fool itself.
- **0-CR free tier** — Cloudflare models (script + image) + Aura TTS produce a complete video for **zero credits**.
- **Async premium renders** — submit returns `poll_id` instantly; poll `render_remotion_result` like a pro.
- **All the formats** — `create_project({aspect_ratio: "9:16"})` for Shorts (portrait canvas + portrait images), `render_video({resolution: "4k"})` for 3840x2160 masters.
- **Battle-tested** — hardened by live agent audits; every hard-won rule is baked into [skill.md](https://mcp.autokeren.com/skill.md).

## Pricing snapshot

| Stage | Free (0 CR) | Paid |
|---|---|---|
| Script | `@cf/zai-org/glm-5.3-flash` | gemini 0.05–0.1 CR |
| Image | `@cf/black-forest-labs/flux-2-klein-9b` | gemini-3-pro-image 8 CR, runway 5–170 CR |
| TTS | `@cf/deepgram/aura-2-en` | ElevenLabs 2–10 CR, Gemini TTS 1 CR |
| Video clip | — | Runway 60–740 CR per 5s (duration-scaled) |

Self-service top-up (QRIS/VA for Indonesia, card for international) via `topup_credits` or the studio. Live prices: `list_models`.

## Links

- 🤖 **Agent skill**: [mcp.autokeren.com/skill.md](https://mcp.autokeren.com/skill.md) — the complete playbook in one URL
- ⚙️ **Get an API key**: [studio.autokeren.com/settings](https://studio.autokeren.com/settings)
- 🎬 **Studio (web app)**: [studio.autokeren.com](https://studio.autokeren.com)
- 🏠 **WebMCP**: `autokeren.com/mcp` — browser agents auto-discover via `document.modelContext`

## Status & reliability

- MCP protocol versions 2024-11-05 / 2025-03-26 / 2025-06-18 (echoed on `initialize`)
- JSON-RPC notifications per spec (202 + empty body), strict-client compatible (opencode verified)
- Optional-field discipline: `structuredContent` only when present
- Runs on Cloudflare Workers + Containers (Region: Earth)

---

*AutoKeren MCP is a hosted service. This repo is the public documentation, onboarding, and registry home — the live server is operated at [mcp.autokeren.com](https://mcp.autokeren.com).*
