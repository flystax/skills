# Upscale — popcraft_upscale_image / popcraft_upscale_video

Topaz-powered enhancement of EXISTING assets (prior results or imported media).
Charges credits → the R3 budget ritual applies (`get_cost:true` first).

## popcraft_upscale_image `[SPEC]`

| Param | Notes |
|---|---|
| `image_url` | GCS https URL (prior result or `media_import_url`) or `gs://` URI |
| `width`, `height` | **Caller-supplied source pixels — the server does not infer them.** Get them from the prior generation's result metadata; if unknown, ask or derive before calling |
| `scale` | 2 (default) or 4 |
| `mode` | `standard-mode` (default) · `high-fidelity-mode` · `low-res-mode` · `cgi-mode` · `text-and-shapes` · `standard-max` · `face-recovery-mode` · `art-and-anime` · `creative-mode` |
| `project_id` / `project_name` / `get_cost` | As everywhere |

**Mode picks:** photos/general → `standard-mode` · faces degraded/small → `face-recovery-mode` ·
illustrations/anime → `art-and-anime` · renders/product CGI → `cgi-mode` · posters with
typography → `text-and-shapes` · very small/blurry source → `low-res-mode` ·
maximum detail invention (can hallucinate) → `creative-mode` — warn the user it reinterprets.

## popcraft_upscale_video `[SPEC]`

| Param | Notes |
|---|---|
| `video_url` | GCS https URL or `gs://` |
| `width`, `height`, `duration` | Caller-supplied source properties; duration 1–300s max |
| `scale` | 2 (default) or 4 |
| `mode` | `live-action` (default) · `detail` · `sharpen` · `theia-detail` · `cgi` |
| `fps` | 24 / 30 (default) / 60 — **above 30 roughly doubles the cost**; say so before offering 60 |

**Mode picks:** filmed/generated live footage → `live-action` · soft AI output needing
crispness → `sharpen` or `detail` · animation/CGI → `cgi`.

## Behavior & budget notes

- Video upscales are long jobs (server allows up to ~30 min). Quote the expectation
  once up front; the result card self-refreshes — don't poll. Topaz ETAs are locked at
  first estimate on our platform `[OURS]` — treat the first countdown as the truth.
- Cost scales with source size × scale × (fps for video) — always `get_cost` with the
  real dimensions; a 4x 60fps quote can surprise.
- **Workflow — generate low, upscale the winner:** iterate video at 720p, then upscale
  the accepted take, instead of generating everything at 1080p/4K. Usually cheaper;
  quote both paths when the user cares.
- Chaining: prior `generate_*` results are GCS URLs — pass directly; no re-import.
