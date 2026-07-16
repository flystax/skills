# Image — popcraft_generate_image

Model facts: [../specs/model-specs.md](../specs/model-specs.md). Cheapest media type —
but R3 still applies to the session's first paid call.

## Tool contract `[SPEC]`

| Param | Notes |
|---|---|
| `model` | Model **id** (`"gpt-image-2"`, `"nano-banana-2"`, `"seedream-4-0"`, …) |
| `prompt` | See prompt formats below |
| `aspect_ratio` | Default `"1:1"`; must be in the model's enum (extreme banner ratios 4:1/1:4/8:1/1:8 exist ONLY on `nano-banana-2`) |
| `resolution` | Size string ("1K"/"2K"/"4K"); omit for model default; Midjourney & Grok expose no size param |
| `count` | 1, 2, or 4 (pricing tiers) — batch variants for style exploration |
| `medias` | `[{value, role:"reference"}]` = condition on a reference image (edit / style / subject transfer) |
| `project_id` / `project_name` | Filing, as in video |
| `get_cost` | Exact quote without submitting |

Delivery: same job contract as video (self-refreshing result card; `job_display` ≤ once).

## Model selection matrix `[SPEC unless tagged]`

| Need | Pick | Why |
|---|---|---|
| Text in the image, layout-dense design (posters, UI, infographics) | `gpt-image-2` | Signature text rendering + regional layout obedience `[FIELD-HF]` — but warn: ETA ≈ 6.5 min, the slowest image model |
| High-fidelity general images, fast | `nano-banana-2` (free) / `nano-banana-pro` (pro) | 4K, wide ratio set, ~60–70s |
| Cheap quality / high volume | `seedream-4-0`, `seedream-5-lite` (5 cr) | 2K/4K capable |
| Instant drafts / thumbnails | `z-image-turbo` (2 cr, ~15s), `nano-banana` (6 cr, ~12s) | Cheap look-lock iterations before an expensive video step |
| Artistic / stylized | `midjourney-v7` | Warn: ~6 min ETA, no size param |
| Banner / ultra-wide formats | `nano-banana-2` | Only model with 4:1 / 1:4 / 8:1 / 1:8 |
| Smart default | `popcraft-image` | Router; 1K only |

## Prompt formats — route by output shape

**The GPT Image 2 three-format taxonomy `[FIELD-HF]` (works on other strong image models too):**

1. **Structured JSON** — output has discrete regions, labels, UI chrome, grids, info
   hierarchy (mockup, dashboard, infographic, character/reference sheet, paneled poster).
   One JSON object accounting for every visible region:
   - Core fields: `type` (one line) · `style` · `subject` (concrete attributes) ·
     nested `layout` object with position-named regions (header / centerpiece /
     left_side / footer) · `background`.
   - **Count-and-label:** repeated items get an explicit count + a parallel array of the
     actual labels.
   - Positions by name (top-left, mid-right, bottom-center) — positional vocabulary is honored.
   - Typography directed inline in field values (font family, weight, case).
2. **Dense prose paragraph** — one scene / subject / frame, no regions. Ordering
   scaffold: medium → subject specifics → pose/action → setting → environment detail →
   lighting → palette/film-stock/texture → mood. Front-load concrete visual anchors;
   itemized wardrobe/prop detail beats mood words.
3. **Meta-prompt** — user gave only a theme ("a poster about X, you design it"). Ask the
   model to derive the whole visual system, but still pin style, composition rules,
   quality bar, and typography. ANY concrete layout detail from the user disqualifies
   this format → switch to JSON.

**Routing tie-break:** torn between JSON and prose → JSON (mis-routing costs are
asymmetric: JSON-when-prose-was-fine is cheap; prose-when-JSON-was-needed loses layout
control). Commit to exactly one format per prompt.

## Text in the image

- Embed display text **in quotes, verbatim, exactly as it should render** — never
  paraphrase; keep non-Latin scripts in their original script.
- Add size/orientation/placement for each text element.
- Text rendering is `gpt-image-2`'s strength; on other models keep text short and
  expect retries.

## Realism & people

- The word "photorealistic" produces plastic skin on GPT Image 2 `[FIELD-HF]` — use
  film vocabulary instead: *"35mm film photograph, direct flash"*, *"editorial
  portrait"*, *"cinematic still"*, plus natural-skin-texture cues.
- Named-referent rule from [prompt-craft.md](prompt-craft.md) applies: exact objects
  (make, color, era), named lighting setups, no filler adjectives.

## Composition vocabulary (single-frame film grammar)

- **Framing ladder:** EWS (world/scale) → wide (context) → full (costume/intro) →
  medium-long (knees up) → cowboy (mid-thigh) → medium (waist) → MCU (chest) → CU
  (face/emotion) → ECU (detail/tension) → macro (texture).
- **Angles = the emotional lever:** eye-level neutral · low = power · high =
  vulnerability · overhead = geometry/patterns · worm's-eye = monumental · Dutch =
  unease · OTS = dialogue · POV = immersion · selfie = casual social.
- **Motion hints in stills** read as implied trajectory/blur: crash zoom, dolly zoom,
  bullet time, rack focus, handheld follow, pull-back reveal.
- Strong still prompt = size + angle (+ optional motion hint), then subject,
  environment, lighting, style.

## Editing with a reference (`role:"reference"`)

- Ingest via [media.md](media.md) → pass `medias:[{value, role:"reference"}]`.
- State what to KEEP and the ONE change ("same product, same angle; background replaced
  with a marble countertop, soft morning window light").
- Style transfer: name which axis the reference drives ("match the reference's palette
  and grain; composition from the prompt").
- Boundary honesty: this is generative editing, not pixel-precise retouching — for
  exact cutouts (background removal), route per SKILL.md capability boundaries.

## Workflows

- **Poster → video:** iterate the poster here (cheap) → animate via video.md
  `start_image` (motion-only prompt).
- **Character sheet for consistency:** one JSON reference-sheet image (grid of views /
  expressions, count-and-label) → use as `role:"image"` identity reference in video shots.
- **Style exploration:** same prompt, `count: 4`, present the grid, lock the winner's
  wording for subsequent shots.
