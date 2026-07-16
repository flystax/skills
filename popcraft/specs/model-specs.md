# PopCraft Model Specs — Snapshot

```yaml
snapshot_date: 2026-07-16
source_env: production (https://popcraft.ai/api/mcp)
source_tool: popcraft_models_explore (action:"list" + action:"get" per model)
model_count: 34 (video 14 · image 14 · audio 6)
```

> **STALENESS RULE (restated from SKILL.md HARD RULES):** if `snapshot_date` is more than
> **30 days** ago, do NOT quote model facts from this file — call `popcraft_models_explore`
> live (`action:"get"`, the exact `model_id`) and prefer its answer. If a live answer
> contradicts this file, the live answer wins and this snapshot should be regenerated.

> **COST NOTE:** `credits` below is the model's **listed baseline cost** as returned by
> `models_explore`. The exact price of a specific call depends on resolution, duration,
> count, and sound — always confirm with `get_cost: true` on the generation tool before
> submitting. `eta` is the model's estimated generation time in seconds (`estimatedSeconds`).

---

## Video models (14)

| id | name | tier | credits | eta (s) | duration (s) | aspect ratios | resolutions | native audio | negative_prompt |
|---|---|---|---|---|---|---|---|---|---|
| `popcraft-video` | Popcraft 1.0 | pro | 27 | 325 | 4–15 (def 8) | 16:9, 9:16, 1:1, 4:3, 3:4, 21:9 | 480p, 720p, 1080p, 4K (def 720p) | ✅ (def on) | — |
| `seedance-2-0` | Seedance 2.0 | pro | 25 | 540 | 4–15 (def 8) | 16:9, 9:16, 1:1, 4:3, 3:4, 21:9 | 480p, 720p, 1080p, 4K (def 720p) | ✅ (def on) | — |
| `seedance-2-0-fast` | Seedance 2.0 Fast | pro | 20 | 335 | 4–15 (def 8) | 16:9, 9:16, 1:1, 4:3, 3:4, 21:9 | 480p, 720p (def 720p) | ✅ (def on) | — |
| `seedance-2-0-mini` | Seedance 2.0 Mini | pro | 14 | 180 | 4–15 (def 8) | 16:9, 9:16, 1:1, 4:3, 3:4, 21:9 | 480p, 720p (def 720p) | ✅ (def on) | — |
| `veo-3-1` | Veo 3.1 | pro | 58 | 160 | enum 4 / 6 / 8 (def 8) | 16:9, 9:16 | 720p, 1080p (def 720p) | ✅ (def on) | ✅ |
| `veo-3-1-fast` | Veo 3.1 Fast | free | 15 | 145 | enum 4 / 6 / 8 (def 8) | 16:9, 9:16 | 720p, 1080p (def 720p) | ✅ (def on) | ✅ |
| `kling-3-0` | Kling 3.0 | pro | 18 | 110 | 3–15 (def 5) | 16:9, 9:16, 1:1 | 720p, 1080p (def 720p) | ✅ (def on) | ✅ |
| `kling-3-0-omni-video` | Kling 3.0 Omni | pro | 16 | 310 | 3–15 (def 5) | 16:9, 9:16, 1:1 | 720p, 1080p (def 720p) | ✅ (def on) | ✅ |
| `kling-o1` | Kling O1 | pro | 18 | 95 | 3–10 (def 5) | 16:9, 9:16, 1:1 | 720p, 1080p (def 720p) | ✅ (def on) | — |
| `seedance-1-5` | Seedance 1.5 | free | 8 | 150 | enum 5 / 10 (def 5) | 16:9, 9:16, 1:1, 4:3, 3:4, 21:9 | 480p, 720p, 1080p (def 720p) | ✅ (def on) | — |
| `wan-2-7` | Wan 2.7 | free | 12 | 105 | 2–15 (def 5) | 16:9, 9:16, 1:1, 4:3, 3:4 | 720p, 1080p (def 1080p) | ✅ (def on) | ✅ |
| `wan-2-6-fast` | Wan 2.6 Fast | free | 8 | 115 | enum 5 / 10 / 15 (def 5) | 16:9, 9:16, 1:1, 4:3, 3:4 | 720p, 1080p (def 1080p) | ✅ (def on) | ✅ |
| `gemini-omni-flash` | Gemini Omni Flash | pro | 14 | 180 | enum 10 only | 16:9, 9:16 | 720p only | ✅ (def on) | — |
| `happyhorse-1-1` | Happyhorse 1.1 | pro | 16 | 280 | 3–15 (def 5) | 16:9, 9:16, 1:1, 4:3, 3:4 | 720p, 1080p (def 1080p) | ❌ (no sound capability exposed) | — |

**Video notes (derived from this snapshot only):**
- 21:9 is supported only by the Seedance family + Popcraft 1.0 — Kling / Veo / Wan / Gemini do NOT accept it.
- 4K is only `popcraft-video` and `seedance-2-0`.
- Veo duration is an **enum (4/6/8)**, not a range — 10s Veo requests are invalid.
- `gemini-omni-flash` is fixed: exactly 10s @ 720p, 16:9 or 9:16.
- `negative_prompt` is a capability only on Veo 3.1 / 3.1 Fast, Kling 3.0, Kling 3.0 Omni, Wan 2.7, Wan 2.6 Fast.
- Slowest ETAs: `seedance-2-0` (540s) · `seedance-2-0-fast` (335s) · `popcraft-video` (325s) · `kling-3-0-omni-video` (310s). Fastest: `kling-o1` (95s), `wan-2-7` (105s), `kling-3-0` (110s).

## Image models (14)

| id | name | tier | credits | eta (s) | aspect ratios | sizes |
|---|---|---|---|---|---|---|
| `popcraft-image` | Popcraft 1.0 | pro | 16 | 70 | 1:1, 9:16, 3:4, 2:3, 21:9, 16:9, 4:3, 3:2, 5:4, 4:5 (def 1:1) | 1K |
| `gpt-image-2` | GPT Image 2 | pro | 9 | 390 | 1:1, 16:9, 9:16, 3:2, 2:3, 4:3, 3:4, 4:5, 5:4, 2:1 (def 1:1) | 1K, 2K, 4K (def 1K) |
| `nano-banana-pro` | Nano Banana Pro | pro | 20 | 70 | 1:1, 3:2, 2:3, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9 (def 1:1) | 1K, 2K, 4K (def 2K) |
| `nano-banana-2` | Nano Banana 2 | free | 15 | 60 | 1:1, 3:2, 2:3, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9, 4:1, 1:4, 8:1, 1:8 (def 1:1) | 0.5K, 1K, 2K, 4K (def 2K) |
| `nano-banana` | Nano Banana | free | 6 | 12 | 1:1, 9:16, 3:4, 2:3, 21:9, 16:9, 4:3, 3:2, 5:4, 4:5 (def 4:3) | 1K |
| `seedream-5-pro` | Seedream 5 Pro | pro | 8 | 80 | 1:1, 2:3, 3:2, 3:4, 4:3, 16:9, 9:16, 21:9, 4:5, 5:4 (def 1:1) | 1K, 2K (def 2K) |
| `seedream-5-lite` | Seedream 5 Lite | pro | 5 | 65 | 1:1, 2:3, 3:2, 3:4, 4:3, 16:9, 9:16, 21:9, 4:5, 5:4 (def 1:1) | 1K, 2K (def 2K) |
| `seedream-4-0` | Seedream 4.0 | free | 5 | 50 | 1:1, 2:3, 3:2, 3:4, 4:3, 16:9, 9:16, 21:9, 4:5, 5:4 (def 1:1) | 1K, 2K, 4K (def 2K) |
| `midjourney-v7` | Midjourney | free | 13 | 360 | 1:1, 16:9, 9:16, 4:3, 3:4 (def 1:1) | — (no size param) |
| `qwen-image-2.0` | Qwen-Image 2.0 | free | 5 | 30 | 1:1, 16:9, 9:16, 4:3, 3:4, 4:5, 5:4 (def 1:1) | 0.5K, 1K, 2K (def 1K) |
| `grok-imagine-image` | Grok Imagine | free | 3 | 30 | 1:1, 16:9, 9:16, 2:1, 20:9, 4:3, 3:4, 3:2, 2:3, 9:20, 1:2 (def 1:1) | — (no size param) |
| `z-image-turbo` | Z Image Turbo | free | 2 | 15 | 1:1, 2:3, 3:2, 3:4, 4:3, 9:16, 16:9, 1:2, 2:1, 4:5, 5:4 (def 1:1) | 0.5K, 1K (def 1K) |
| `Z Image` | MiniMax Hailuo Image 01 | free | 0 | 0 | 1:1, 16:9, 9:16, 4:3, 3:4 (def 1:1) | 1K, 2K (def 2K) |
| `agnes-image` | Agnes | free | 0 | 40 | 1:1, 4:3, 3:4, 3:2, 2:3, 16:9, 9:16, 21:9, 9:21, 5:4, 4:5 (def 1:1) | 0.5K, 1K, 2K (def 1K) |

**Image notes (derived from this snapshot only):**
- ⚠️ The model id `Z Image` contains a **space** and its display name is "MiniMax Hailuo Image 01" — pass the id exactly as `"Z Image"`.
- `Z Image` and `agnes-image` list **0 baseline credits** — still run `get_cost:true` before submitting; do not promise "free" to the user without a live quote.
- Extreme banner ratios (4:1, 1:4, 8:1, 1:8) exist **only** on `nano-banana-2`.
- 4K images: `gpt-image-2`, `nano-banana-pro`, `nano-banana-2`, `seedream-4-0`.
- Slow ETAs to warn about: `gpt-image-2` (390s ≈ 6.5 min) and `midjourney-v7` (360s); fastest: `nano-banana` (12s), `z-image-turbo` (15s).

## Audio models (6)

| id | name | tier | credits | eta (s) | key params |
|---|---|---|---|---|---|
| `popcraft-audio` | Popcraft 1.0 | free | 1 | 0 | voice_id; model_id enum `eleven_v3` / `eleven_multilingual_v2` (def multilingual_v2); emotion enum (neutral/happy/sad/angry/excited/calm/nervous/frustrated/whispers/cheerful); speech_rate 0.5–2 (def 1); output mp3_44100_128 / mp3_22050_32 |
| `elevenlabs-tts` | ElevenLabs TTS | free | 1 | 8 | same param set as `popcraft-audio` |
| `elevenlabs-sfx` | ElevenLabs SFX | free | 2 | 15 | duration_seconds 0.5–22 (def 5); prompt_influence 0–1 (def 0.3) |
| `music-v1` | Music V1 | free | 1 | 25 | duration 3–200s (def 30); force_instrumental bool (def true); output format enum |
| `lyria-3-clip` | Lyria 3 Clip | free | 8 | 20 | force_instrumental bool (def true) |
| `lyria-3-pro` | Lyria 3 Pro | pro | 16 | 65 | force_instrumental bool (def true) |

**Audio notes (derived from this snapshot only):**
- TTS voices come from `popcraft_list_voices` — `voice_id` here is a free string; never invent one.
- Only `music-v1` and `elevenlabs-sfx` expose a duration parameter in capabilities; the Lyria models do not.

---

## Regeneration procedure

1. In a session with the PopCraft MCP connected (production), call `popcraft_models_explore` with `action:"list"` for `type` = `video`, `image`, `audio`.
2. For every returned id, call `action:"get"` with that exact `model_id`.
3. Rewrite this file from scratch (do not hand-edit rows), update `snapshot_date` and `model_count`, and re-derive the notes sections — delete any note no longer supported by the data.
4. Grep the skill's `references/*.md` for model ids/caps that changed and fix them in the same commit.
