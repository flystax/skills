# Library — history, media, elements, effects, projects

The user's saved world. All read-only except project creation. On Apps hosts most of
these render interactive widgets — a picked item comes back as the next user message.

## popcraft_show_generations — generation history `[SPEC]`

- Lists recent jobs (image/video/audio), newest first; filter `type`, `limit` ≤50 (default 20).
- Each item carries its **full original params** (model, ratio, prompt, credits) and
  result URLs → a past result is directly reusable: pass its URL as `medias[].value`.
- Use it to: find a `job_id` from an earlier session; "show me what I made"; recover a
  prompt the user liked.
- **Never use it as a post-generation poll** — the result card already live-updates.

## popcraft_show_medias — media library

- Thumbnail grid of the user's saved/uploaded media. Use when the user references
  something they uploaded before ("use my product photo") instead of re-asking for the file.

## popcraft_show_elements — reusable characters/objects/backgrounds/styles `[SPEC]`

- Each element is backed by a reference image. Filter by `type`
  (character/object/background/style) or `search` on name.
- **To use one in a generation:** pass its `image_url` as `medias[].value` —
  `role:"reference"` for image gen, `role:"image"` (or `"start_image"`) for video —
  and mention the element by name in the prompt.
- This is OUR answer to cross-shot character consistency: element reference + verbatim
  identity block ([prompt-craft.md](prompt-craft.md) § Universal laws).

## popcraft_effects_show — curated effects gallery `[SPEC]`

- Curated video effects (cinematic moves, VFX, viral looks) + the user's saved effects.
  Each = reference video + recipe prompt + `default_model_id`. Filter by `category`/`query`.
- **To apply an effect:** `popcraft_generate_video` with `model` = the effect's
  `default_model_id`, `medias` = `{value: reference_video_url, role:"video"}`, and a
  prompt combining the effect's recipe `prompt` with the user's subject (add the user's
  photo as `role:"start_image"`/`"image"` if provided).
- Route here when the user asks "what effects/looks do you have" or names a viral effect.

## popcraft_projects — folders `[SPEC]`

- `action:"list"` (default): id, title, asset count, which is active.
  `action:"create"` + `title`: new project.
- Generations auto-file into the active/last-used project; to target one, pass
  `project_id` (or `project_name` — auto-creates) on the generate call. Every result
  echoes where it landed — relay that per HARD RULE R5.
- Multi-asset briefs (campaign, series): create one project up front, file everything
  into it, deliver the project link once at the end.
