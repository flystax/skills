# Prompt Craft — the director's handbook for PopCraft models

How to turn a user brief into a prompt that generates well on OUR models.
Model facts (durations, ratios, params) come from [../specs/model-specs.md](../specs/model-specs.md) — this file is about the words.

**Provenance tags:** `[FIELD-HF]` = practitioner claim adapted from field use of the same
underlying model on another platform; treat as high-probability, verify on ours (P4).
`[OURS]` = observed on our own deployment. `[SPEC]` = from our specs snapshot. Untagged = universal film grammar.

---

## The house formula

Every video prompt: **Subject · Action · Scene · Camera · Style** (+ **Audio** on
native-audio models). Order matters more than completeness:

1. **Subject + Action land in the first 20–30 words** — early tokens dominate; by the
   third sentence the model samples loosely rather than obeys. [FIELD-HF]
2. **Scene** (setting, light source, atmosphere) next.
3. **Camera** third — one named move (see vocabulary below).
4. **Style** near the end — one anchor + 1–2 supporting tokens, never more.
5. **Constraints last** — positive phrasing only (see [negative-constraints.md](negative-constraints.md)).

**Length:** 50–80 words is the coherence sweet spot for a single shot; ~200 words is the
hard ceiling before quality drops. [FIELD-HF] More words buy diffusion, not control.

**Per-model input caps [OURS]:** Kling family & Happyhorse ≈ 2,500 chars; Wan ≈ 1,500
chars. Stay far below caps anyway — the sweet spot is shorter than every cap.

## Universal laws

- **One primary action per clip** (+ at most 1–2 secondary motions). Complex sequences
  → split into chained shots. Over-stuffed action is the #1 cause of jitter.
- **One primary camera move per shot.** Stacked moves ("dolly in while panning while
  craning") render as instability. A texture modifier is fine ("slow dolly-in, slightly
  handheld"). Multiple moves → timed phases ("pans 0–3s, holds, pushes in 4–8s"). [FIELD-HF]
- **Physics, not mood.** "Fast car" → "tires spin, gravel sprays backward." "Looks
  angry" → "jaw clenches, nostrils flare." Models render what's observable.
- **Set intensity with degree adverbs** — slowly, violently, gently, explosively —
  models can't infer intensity from context.
- **Kill filler adjectives.** beautiful / stunning / epic / dynamic / high-quality / 4K
  add zero information. "Cinematic" does nothing — every output is already cinematic.
  Replace with a named referent: a lighting setup, lens spec, film stock, palette.
- **Image-to-video prompts describe ONLY what moves or changes.** Re-describing what
  the still already shows makes two competing sources for one subject → drift. [FIELD-HF]
- **In medias res.** Start scenes mid-progress; script an opening/closing beat only when asked.
- **Recurring characters:** keep identity descriptors in a static block, separate from
  motion/camera text, and re-paste it **verbatim** across shots — paraphrasing at shot
  boundaries is where identity drifts. [FIELD-HF]
- **Avoid age words** (boy, girl, child, kid, teen, young) — age inference drifts
  between shots and triggers filters; describe role, clothing, action instead. [FIELD-HF]
- **Emotion = one base label + one physical tell** ("upset, lower lip trembling") —
  better than either a bare label or full anatomical decomposition.
- **Homograph check:** verbs with a second physical reading get rendered literally
  (wind "tearing" at a coat → shredding). Watchlist: tearing, shoot, duck, bolt, draw,
  wave, charge, rock, drop, fire, strike, break, snap. [FIELD-HF]

## Per-model deltas

### Seedance family (`seedance-2-0`, `-fast`, `-mini`, `seedance-1-5`) — also the safe default register for `popcraft-video`
- **Six-slot completeness:** camera, subject, action, setting, style, lighting — a ~30-word
  prompt covering all six almost never trips the content filter; omitting 3+ slots is
  the main rejection source. The filter judges whole-scene intent (an LLM, not a keyword
  list) — a legible filmmaker's shot description passes where a bare subject fails. [FIELD-HF]
- **NO negative syntax.** Every token is a positive instruction; "no shaky camera"
  paints shaky camera. Convert to assertions ("locked-off static camera"). [FIELD-HF]
- **The word "fast" degrades output** when motion layers; let ONE element carry speed,
  expressed as body mechanics. [FIELD-HF]
- Engine constraints [FIELD-HF]: tracks ~3 characters max; a character who exits frame
  is gone for that shot; off-screen = nonexistent (show state changes on camera);
  avoid mirror/reflection shots; spatial continuity resets at cuts — re-anchor after each.
- Timing brackets ("0–3s: …") are read as CUT instructions — never put per-segment
  timing inside an intended continuous shot. [FIELD-HF]
- Multi-shot production prompts drop the word cap and use a block scaffold (context →
  space/timing → action/physics → style-next-to-what-it-governs → locks last); never
  open with a style prefix. [FIELD-HF]
- FOV in degrees from an anchor ladder (180/107/84/63/47/29/18/12/8°), not millimeters. [FIELD-HF]

### Kling family (`kling-3-0`, `kling-3-0-omni-video`, `kling-o1`)
- **Reference mentions [OURS]:** Kling models use `<<<image_N>>>` / `<<<video_N>>>`
  in-prompt tokens for attached references (the platform auto-converts `@Image`/`@Video`
  mentions at submit). Give each reference exactly one job in the text.
- Prompt like a screenplay: action, camera, mood, and quoted dialogue cues written
  together; strong multi-speaker attribution. [FIELD-HF]
- `kling-3-0` and `kling-3-0-omni-video` expose a **`negative_prompt` parameter** [SPEC] —
  put unwanted artifacts there; keep the prompt body positive regardless. `kling-o1` has
  no such param.
- Durations are ranges (3–15s; O1 3–10s) [SPEC].

### Veo (`veo-3-1`, `veo-3-1-fast`)
- Durations are an **enum: 4 / 6 / 8s** — never promise 10s Veo [SPEC].
- Strongest English dialogue + environmental audio; quote dialogue lines verbatim. [FIELD-HF]
- Exposes `negative_prompt` [SPEC]. 16:9 / 9:16 only [SPEC].

### Wan (`wan-2-7`, `wan-2-6-fast`)
- Shortest input cap (~1,500 chars [OURS]) — keep prompts tight.
- Exposes `negative_prompt` [SPEC]. Defaults to 1080p [SPEC].

### Popcraft 1.0 router (`popcraft-video`, `popcraft-image`, `popcraft-audio`)
- Smart routing picks the provider — write in the universal register above (six-slot,
  positive-only, one action/one move) so the prompt lands well wherever it routes.

## Camera vocabulary (film grammar — safe on all models)

- **Dolly:** dolly in (intimacy/tension) · dolly out (isolation/context) · lateral track ·
  dolly zoom (subject holds size, background rushes — vertigo/realization)
- **Crane:** crane up (intimate→grand) · crane down (world→personal) · overhead (surveillance/geometry)
- **Orbit:** 360 orbit (isolate/emphasize) · partial arc (turning point) · turntable (product)
- **Zoom:** slow push-in · crash zoom (shock punch)
- **Follow:** tracking shot · FPV weave (chase energy) · low behind-follow (pursuit) ·
  handheld (documentary/urgency)
- **Specialty:** bullet time · Dutch angle (unease) · whip pan (≥0.8s of blur travel or it
  renders as a hard cut [FIELD-HF]) · hyperlapse · rack focus
- **Angles:** low = power · high = vulnerability · eye-level = neutral · bird's-eye = god view ·
  worm's-eye = towering scale · OTS = dialogue · POV = immersion
- **Shot sizes:** EWS (scale/isolation) → wide (context) → full → medium (dialogue) →
  MCU (reaction) → CU (emotion) → ECU (single feature) → macro (texture)
- **Camera-emotion sync:** the camera is the focal character's emotional double —
  anger = jittery handheld · calm = smooth drift · sadness = slow low downward drift ·
  shock = freeze then minimal push. Mismatch (jittery cam, calm scene) is a visible AI tell.
- **Genre defaults:** product = orbit/push-in/static (never handheld) · drama = slow push /
  tracking · horror = static/slow creep/Dutch (fast tracking kills dread) · action =
  FPV/tracking/handheld · comedy = deadpan static + snap zoom.

## Style & lighting vocabulary

- **Lighting (usable verbatim):** golden hour · blue hour · overcast diffuse · neon ·
  volumetric · practical-only · side-lit · backlit rim · low-key · high-key · Rembrandt ·
  butterfly · split · chiaroscuro
- **Mood→grade:** thriller = teal-orange desaturated · nostalgia = golden amber, soft
  shadows · cyberpunk = neon magenta-cyan · horror = sickly green-yellow, crushed blacks ·
  romance = soft pink-gold, lifted shadows · noir = high-contrast B&W · sci-fi = ice
  blue-silver · post-apocalyptic = desaturated orange-brown haze
- **One style anchor** + 1–2 support tokens; >3 style tokens dilutes attention. [FIELD-HF]
- **Period looks:** evoke the era via materials + light (Kodachrome + tungsten amber, not
  "1970s"). **Aspect ratio is a parameter, not prompt text** — "anamorphic/2.35:1" is a
  *look* you describe; the output ratio comes from the enum in specs.
- **CGI/product surfaces:** 2–4 material properties per surface (base material, roughness,
  imperfection, edge treatment) to beat plastic sheen.

## Spatial anchoring (multi-character / precise blocking)

When a shot has 2+ characters, or blocking must hold across cuts, put a per-character
**anchor block** before the action text [FIELD-HF]:
- Per character: identity ref · screen position (qualitative third **plus** x/y %) ·
  depth layer · body orientation · pose · gaze target · **contact points** (what touches
  what — prevents floating) · state lock.
- Relations block: distance, eyelines, axis-crossing rule, negative space.
- Close with continuity locks (costume/prop invariants) + a final-frame target.
- Single character: same block minus relations; one dominant action verb, secondary
  motion listed separately.

## Advanced: facial acting via FACS (Seedance close-ups)

AU codes in the prompt steer facial muscles with higher resolution than emotion labels.
High-probability, never guaranteed [FIELD-HF]. Rules: ≤3–4 expression beats per clip,
1–2 AUs per beat, one continuous close-up camera, quoted dialogue drives the mouth while
AUs drive brow/eyes/cheeks. Recipes: genuine smile = AU6+AU12 · forced smile = AU12
alone · sadness = AU1+AU4+AU15 · fear = AU1+AU2+AU4+AU5+AU7+AU20 · anger = AU4+AU5+AU7+AU23.

## Iteration discipline

1. **Classify the miss first.** Same failure every roll = systematic → fix the prompt.
   Different failures, near-hits = stochastic → lock the prompt, batch (`count: 2|4`), cull.
   Serial re-rolling a correct prompt just burns credits.
2. **Change exactly one variable per regeneration.**
3. **Diagnose in order** Subject → Action → Camera → Style → Audio → Output; most
   failures live in the first two. No improvement by pass 4 → structural rewrite.
4. **Prompt hygiene between shots:** replace clauses in place (never append edits),
   delete stale blocks and stale references — leftovers pull old props into new output.
5. Failure symptom lookup: [troubleshoot.md](troubleshoot.md).

## Worked example (before → after)

**User:** "an epic cinematic video of a cool cyberpunk girl walking in the rain, 4K"

**After (Seedance register, ~70 words):**
> A woman in a translucent rain poncho over silver techwear walks toward camera through
> a narrow neon-lit alley, puddles rippling with each step, rain streaking through
> volumetric signage glow. Slow dolly-out at eye level, keeping her centered as the
> alley reveals around her. Neon magenta-cyan grade, wet-surface reflections, shallow
> depth of field. Steady natural gait; sharp focus on her face throughout.

What changed: filler adjectives → named palette + light behavior; "epic/4K" deleted
(resolution is a parameter); one action, one camera move; constraint stated positively.
