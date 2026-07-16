# Negative Constraints — artifact prevention phrasing

How to stop known generation artifacts BEFORE they happen. Two delivery mechanisms:

1. **In the prompt body — positive assertions only, on every model.** Some engines
   (the Seedance family, and therefore anything `popcraft-video` may route to) have no
   negative syntax at all: every token is a positive instruction, so "no shaky camera"
   *paints* a shaky camera. `[FIELD-HF]` Write what must be TRUE instead.
2. **The `negative_prompt` parameter — only where specs expose it** `[SPEC]`:
   `veo-3-1`, `veo-3-1-fast`, `kling-3-0`, `kling-3-0-omni-video`, `wan-2-7`,
   `wan-2-6-fast`. Artifact nouns go there ("extra fingers, warped text, flicker");
   the prompt body stays positive regardless.

**Conversion template:** name the artifact → assert its opposite state.
"no shaky camera" → *locked-off static camera* · "don't change the face" → *identical
facial features throughout* · "no blur" → *sharp focus edge to edge*.

---

## Category 1 — Body & Motion

| Artifact | Cause | Prevention phrasing (append to prompt) |
|---|---|---|
| Extra/merged limbs when characters touch or stand close | Model blends nearby bodies | *"clear physical separation between the two figures; each with anatomically complete hands"* — or split them into separate shots |
| Jittery, oscillating motion | Over-stuffed action (several simultaneous motions) | Cut to ONE primary action: *"her only movement is the slow raise of her right arm; everything else still"* |
| Broken martial-arts / dance choreography | Named techniques demand multi-frame precision | Describe force + intent, not biomechanics: *"a single decisive strike knocks him back; impact carries real weight"* |
| Fast action smears into incoherence | Exceeds temporal coherence | Generate slow: *"in smooth slow motion, every movement deliberate"* — speed up in the edit `[FIELD-HF]` |

## Category 2 — Face & Identity

| Artifact | Cause | Prevention phrasing |
|---|---|---|
| Identity drift across a shot / between shots | Paraphrased or scattered identity text | Verbatim identity block, re-pasted per shot + an identity reference image (`role:"image"`); *"exact same face as the reference throughout"* |
| Face re-interpreted mid-shot | Identity words mixed into motion/camera text | Keep identity in its own static block, motion in another |
| Waxy / plastic skin | Realism-flagged wording on a weak model | *"natural skin texture with visible pores, soft film grain"*; pick a realism-capable model; on GPT Image 2 use film vocabulary, never "photorealistic" `[FIELD-HF]` |
| Expression flip-flop | Multiple emotion labels | ONE emotion + one physical tell per shot |

## Category 3 — Texture & Lighting

| Artifact | Cause | Prevention phrasing |
|---|---|---|
| Flat, even, sourceless light | No lighting specified | Always name source + quality: *"single warm practical lamp from frame left, soft falloff"* |
| Texture flicker / style mush | Competing or vague style tokens | ONE style anchor + grade, repeated verbatim across shots |
| Plastic CGI sheen on products | No material spec | 2–4 material properties per surface: *"brushed aluminum, fine machining marks, matte finish, softly beveled edges"* |

## Category 4 — Temporal & Consistency

| Artifact | Cause | Prevention phrasing |
|---|---|---|
| Near-static image-to-video output | Prompt re-described the still | Describe ONLY motion + camera: *"steam rises from the cup; slow push-in"* |
| Unwanted cuts in a continuous shot | Per-segment timing brackets read as cut instructions `[FIELD-HF]` | Remove all timestamps; *"one continuous unbroken take"* |
| Model invents its own cuts (multi-shot) | Cut count under-specified | Declare the cut ladder explicitly + *"no additional cuts beyond those specified"* `[FIELD-HF]` |
| Contradictory camera (in AND out) | Opposing moves in one shot | One move; or timed phases; or separate chained clips |
| Adjacent objects move together (the "magnet follows the photo" bug) | No physics engine | Name the invariant: *"the shelf and everything else on it stays fixed; only the photo moves"* `[FIELD-HF]` |
| Wrong room geometry (doors, hallways, walk-through-furniture) | Under-specified space | Lock geometry up front: camera start position, key obstacles, direction of travel `[FIELD-HF]` |

## Category 5 — Content filter & safety

The Seedance-family filter is an intent judge (LLM), not a keyword list — a fully
legible filmmaker's scene passes where a bare risky subject fails. `[FIELD-HF]`

| Rejection trigger | Substitution |
|---|---|
| Real person / celebrity names | Detailed anonymous archetype ("a silver-haired action star in his 60s") |
| Brand & IP names | Pure visual attributes, trademark stripped + *"generic unbranded design"* (also generates more reliably) |
| Violence verbs & weapon nouns | Aftermath, tension, force physics, silhouettes, atmosphere |
| Age markers (child, kid, teen, boy, girl) | Role + clothing + action |
| Casual/conversational register | Rewrite in shot-description register: camera + subject + action + setting + style + lighting |
| Anatomy-specific or sensual-register wording | Generalize anatomy terms, neutralize adjectives — never resubmit unchanged `[FIELD-HF]` |

## Category 6 — PopCraft-specific quirks `[OURS]`

- Kling reference mentions must use `<<<image_N>>>` / `<<<video_N>>>` tokens — a prose
  "the attached image" is ignored by Kling's reference binding.
- Prompt-length caps: Kling family & Happyhorse ≈ 2,500 chars; Wan ≈ 1,500 — the
  connector may hard-reject beyond these.
- Never mention a reference in a shot where that object shouldn't appear — the model
  forces it into frame. `[FIELD-HF]`
- Seedance video references: keep input clips ≤15s (hard tolerance 15.2s) and each
  frame within the supported pixel envelope — oversized refs fail validation (free) but
  waste a round-trip.

## Usage ritual

Map the request type to its category checklist and append the matching phrases:
character shot → 2 + 1 · action shot → 1 + 4 · product shot → 3 · multi-shot sequence →
4 + 2 · dialogue/lip-sync → the lip-sync rules in [video.md](video.md) · anything
borderline → 5. Keep every added phrase positive.

**Always-on micro-check:** if hands are visible in the shot (pouring, gripping,
typing…), add a natural-hands lock ("hands natural and anatomically complete") — hands
are the most common artifact site even in calm shots `[OURS — benchmark-verified 2026-07-16]`.
