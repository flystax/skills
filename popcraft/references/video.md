# Video — popcraft_generate_video

The most expensive media type. Cost preview (`get_cost:true`) is mandatory before any
un-confirmed submission (HARD RULE R3). Model facts: [../specs/model-specs.md](../specs/model-specs.md).

## Tool contract `[SPEC]`

`popcraft_generate_video` parameters:

| Param | Notes |
|---|---|
| `model` | Model **id** from specs (e.g. `"seedance-2-0"`) — ids, never display names |
| `prompt` | The director-grade prompt ([prompt-craft.md](prompt-craft.md)) |
| `aspect_ratio` | Default `"9:16"`. MUST be in the model's enum (21:9 = Seedance family + Popcraft 1.0 only) |
| `resolution` | Model-specific string ("720p"); omit for model default |
| `duration` | Seconds. Tool accepts 1–60 but the MODEL's range in specs is the real limit (Veo: enum 4/6/8) |
| `count` | 1, 2, or 4 only (pricing tiers). Default 1. Use 2/4 for variance-harvesting a locked prompt |
| `medias` | Reference media array — see roles below |
| `project_id` / `project_name` | Filing; `project_name` auto-creates. Omit → user's current project (result echoes it) |
| `get_cost` | `true` = exact credit quote, nothing submitted |

**Delivery behavior:** result card self-refreshes on Apps hosts — do NOT poll;
`popcraft_job_display` at most once; obey `recovery_tool` immediately (SKILL.md § Job contract).

## Model selection matrix (from the specs snapshot — verify if stale)

| Priority | Pick | Why `[SPEC]` |
|---|---|---|
| Default / unsure | `popcraft-video` | Smart routing, full ratio set, 4–15s, 4K available |
| Identity/consistency-critical (animate a face/poster/product, reference-driven) | `seedance-2-0` | The connector's own routing rule; strongest reference handling |
| Best quality, budget secondary | `seedance-2-0`, `veo-3-1` | Veo = premium price (58 baseline) |
| Fast turnaround | `kling-o1` (~95s), `wan-2-7` (~105s), `kling-3-0` (~110s) | Lowest ETAs |
| Budget | `seedance-1-5` / `wan-2-6-fast` (8 baseline) | Free tier |
| Long clip 10–15s | Seedance family, Kling 3.0/Omni, Wan, Happyhorse | Veo caps at 8s |
| Ultrawide 21:9 | Seedance family or `popcraft-video` ONLY | Kling/Veo/Wan lack it |
| 4K | `popcraft-video`, `seedance-2-0` only | |
| Native audio required | Any except `happyhorse-1-1` | Happyhorse exposes no sound param |
| Slow-burn warning | `seedance-2-0` ETA ≈ 9 min, `-fast` ≈ 6 min | Tell the user upfront |

## Reference media — `medias[].role`

| role | Use | Prompt rule |
|---|---|---|
| `start_image` | Animate this image (i2v) | Prompt = **motion + camera ONLY** — never re-describe the still `[FIELD-HF]` |
| `start_image` + `end_image` | First-to-last-frame morph | Describe the transition path; keep it one continuous take |
| `image` | Style/subject reference | Give the reference ONE job in the text ("identity per reference; costume recolored blue") |
| `video` | Extend / continue / transform | Model-dependent — check capabilities first; see below |

**Validation is free:** the connector validates references against the model (count,
type, size, format) BEFORE charging. Known caps `[OURS]`: Seedance 2.0 family ≈ 9 images
+ 3 videos (input clips ≤15s each); Kling Omni & O1 ≈ 4–7 images + 1 video;
Happyhorse ≈ 9 images + 1 video; `kling-3-0` base takes no video input.
Kling prompts bind references via `<<<image_N>>>` / `<<<video_N>>>` tokens `[OURS]`.

**No prompt given for an i2v?** Write the motion prompt yourself from the image content
(the connector's own instruction): keep composition, faces and text stable and sharp;
add atmospheric motion — slow push-in, drifting particles, wind in hair/cloth, subtle
breathing and blinks.

## Extend / continue a clip (`role:"video"`)

The continuation formula `[FIELD-HF]`:
1. Open with ONE sentence describing the source clip's final frame (pose, frame
   position, gaze) — then start the new action on the very next frame. No time skip.
2. Re-paste the character identity block **verbatim** — shot boundaries are where
   identity dies.
3. Reference what just happened in one short phrase; re-describing the prior action in
   detail makes the model REPLAY it (the loop failure).
4. Match the source clip's resolution and duration — mismatches show as a quality jump
   at the join.
5. **Chains compound artifacts:** visible drift by the 4th–5th chained clip. Cap
   seamless chains at ~2–3, then re-anchor from original references or bridge with a
   cutaway. End each extension on a camera-angle change so the next join reads as
   intentional coverage.
6. If an identity feature is occluded in the final frame, attach the character image as
   a second reference and bind it in text.

## Transform existing footage (v2v)

The lock-plus-one-change contract `[FIELD-HF]`:
- **Lock everything recognizable** — identity, face, wardrobe, performance, framing,
  lens, camera path — then name **exactly one change**. Repeat the most fragile lock
  (face) again at the end of the action text.
- Declare the source clip as the base plate to preserve — never as a style reference.
- Write the change as physics over time (origin → spread → motion → emitted light) and
  script its interaction with the plate: light spill on skin, contact shadows, scale by
  comparison to in-frame structures (giants render life-size unless compared).
- Integration needs four channels, not just color: matched key direction/softness,
  environmental bounce onto the subject, matched optics/haze/grain, grounded edges.
- Difficulty scales with camera motion (locked-off = easy; handheld parallax = hardest).
- Cheap look-lock: generate the transformed FIRST FRAME as an image, approve it, then
  animate it via `start_image` — image iterations cost a fraction of video ones.

## Native audio (sound_enabled models)

Audio cue = the last formula slot. Think in four layers, include only what the shot
needs: **dialogue** (quoted lines + speaker + delivery verb + language) · **SFX**
(described at the action they belong to, one per beat) · **ambient** (≤2–3 elements +
the space's acoustic character) · **BGM** (instrumentation + tempo + mood — NEVER song
or artist names).

- **Diegetic-only rule (Seedance):** the prompt body describes only sounds that
  physically exist in the scene; the score is added in post. "Dramatic strings" yields
  generic mismatched music. `[FIELD-HF]`
- **Lip-sync rules** `[FIELD-HF]`: dialogue beats 3–8s (sync collapses past ~8s even
  when 15s is accepted) · medium close-up or tighter · ONE speaking face per shot ·
  camera static or slow push-in · strip all head-motion words (nodding, turning) ·
  when sync is the priority, omit ambient/music tokens so generated audio can't
  override the dialogue · exact scripted wording sets sync quality (voice is
  synthesized first, face mapped to it) — write the line in the target language.
- Multi-speaker lip-sync is unsolved: generate each speaker separately, composite in an editor.

## Workflows

- **t2v:** Stage 1 prompt → `get_cost` → confirm → submit.
- **"Animate this image":** [media.md](media.md) ingestion → `start_image` → motion-only prompt.
- **Image → video pipeline (recommended for briefs needing a precise look):** generate
  the key frame with `popcraft_generate_image` (cheap iterations) → user approves →
  animate via `start_image`. Same trick locks a v2v look (above).
- **Batch variance:** prompt correct but performance imperfect → same prompt,
  `count: 2|4`, cull. Cheaper than serial re-rolls of a correct prompt.
- **Video → upscale:** pass the result URL to `popcraft_upscale_video` ([upscale.md](upscale.md)).

## Cost & ETA notes

- Video is priced per-second-scaled — e.g. `seedance-2-0` quoted 125 credits for 5s/720p
  (baseline 25 × 5s) `[OURS — one observation, verify]`. Always `get_cost` per actual params.
- Quote ETA from the specs `eta` column, rounded up to a friendly range. Never invent one.
- Failures: diagnose via [troubleshoot.md](troubleshoot.md) before any re-spend;
  server refunds failed tasks automatically.
