# Troubleshoot — symptom · mechanism · counter

Diagnose BEFORE re-spending (HARD RULE / Stage 3). Change the thing that failed, not
everything. Tags: `[FIELD-HF]` = adapted field knowledge, verify on ours ·
`[OURS]` = observed on our platform · `[SPEC]` = connector/specs contract.

## First: classify by WHEN it failed

| Timing | Meaning | Move |
|---|---|---|
| Rejected near-instantly | Content filter / validation blocked pre-render (validation failures are free `[SPEC]`) | Rewrite per § Rejections — NEVER resubmit unchanged |
| Failed mid/late generation | Infra or complexity | Simplify (shorter duration, one motion, lower res); server auto-refunds failed tasks — verify via `popcraft_transactions` if asked |
| "Failed" but user was told to wait | Maybe just slow | Check `popcraft_job_display` once before concluding anything |

Then: same failure on every roll = **systematic** → fix the prompt below. Different
failures, near-hits = **stochastic** → lock the prompt, batch `count:2|4`, cull.

## Rejections (filter / validation)

| Symptom | Mechanism | Counter |
|---|---|---|
| Prompt rejected, looks innocent | Filter judges whole-scene intent; bare subjects without scene context read as risky `[FIELD-HF]` | Rewrite in full shot register — all six slots (camera/subject/action/setting/style/lighting), then resubmit |
| Rejected repeatedly after word swaps | Word-swapping doesn't change scene intent `[FIELD-HF]` | Stop the loop: lint against [negative-constraints.md](negative-constraints.md) § Category 5 (names, brands, violence verbs, age words, anatomy terms), rewrite the SCENE, one resubmit |
| Clean prompt rejected (NSFW false positive) | Classifier fires on anatomy specificity / costume vocabulary / sensual-register adjectives `[FIELD-HF]` | Generalize anatomy terms, neutralize adjectives, re-check register |
| Reference rejected before charge | Connector validates count/type/size/format per model `[SPEC]` | Check the model's reference caps ([video.md](video.md)); for Seedance video refs: clip ≤15s, frame within the pixel envelope `[OURS]` |
| Rate-limit error | Per-user per-tool cap `[SPEC]` | Wait the returned `retry_after_seconds`, then continue — don't hammer |

## Output quality — image & video

| Symptom | Mechanism | Counter |
|---|---|---|
| Near-static i2v ("it barely moves") | Prompt re-described the still — two sources compete `[FIELD-HF]` | Motion + camera ONLY: what changes, atmospheric motion, one camera move |
| Generic "AI-looking" output | Filler adjectives sample diffuse training data | Anti-slop pass: named lighting/lens/palette referents ([prompt-craft.md](prompt-craft.md)) |
| Plastic skin / waxy faces | Realism-flagged wording; weak model | Film vocabulary ("35mm film photograph", "editorial portrait"), natural-texture cues; on images prefer `gpt-image-2`-class models with film phrasing `[FIELD-HF]` |
| Warped / gibberish text in image | Model can't render long or unquoted text | Verbatim text in quotes + size/placement; route text-heavy work to `gpt-image-2`; keep text short elsewhere |
| Flat sourceless lighting | No light specified | Name source + quality + direction |
| Style mush / texture flicker | Competing style tokens | ONE anchor + ≤2 support tokens, repeated verbatim across shots |
| Plastic product renders | No material spec | 2–4 material properties per surface |

## Motion & temporal — video

| Symptom | Mechanism | Counter |
|---|---|---|
| Jitter / wobble / unwanted rotation | Stacked camera moves or over-stuffed action `[FIELD-HF]` | One primary move (+ texture modifier); one action; split the rest |
| Model cut where a oner was wanted | Timing brackets read as cut instructions `[FIELD-HF]` | Remove per-segment timestamps; "one continuous unbroken take" |
| Model added its own cuts | Cut plan under-specified `[FIELD-HF]` | Declare the cut ladder + "no additional cuts" |
| Choppy playback / duplicated frames (fps drift) | Effective fps below requested `[FIELD-HF]` | State fps + "no repeated frames" in the prompt; regenerate hero shots (editor de-dup only salvages B-roll) |
| Adjacent objects move together | No physics engine `[FIELD-HF]` | Name the invariant ("only the photo moves; shelf stays fixed") |
| Impossible room geometry (doors, hallways) | Under-specified space `[FIELD-HF]` | Lock camera start position, obstacles, travel direction up front |
| Fast action smears | Exceeds temporal coherence | Generate in slow motion, speed up in the edit `[FIELD-HF]` |
| Extension replays the previous beat | Prior action re-described in detail `[FIELD-HF]` | Continuation formula ([video.md](video.md)): final-frame line + one-phrase recap + next-frame action |
| Quality degrades along an extension chain | Artifacts compound per hop `[FIELD-HF]` | Cap chains ~2–3; re-anchor from original references; bridge with a cutaway |

## Identity & characters

| Symptom | Mechanism | Counter |
|---|---|---|
| Face drifts across a shot / between shots | Paraphrased identity text; identity mixed into motion text `[FIELD-HF]` | Verbatim identity block, separate from motion; attach an element/reference image ([library.md](library.md)) |
| Characters swap or merge | >3 tracked characters, or bodies too close `[FIELD-HF]` | Cap at ~3; anchor blocks with positions + contact points; separate shots |
| Reference object appears where it shouldn't | Any mentioned reference gets forced into frame `[FIELD-HF]` | Never mention a reference in a shot it doesn't belong to |
| Kling ignores the reference | Prose mention instead of token binding `[OURS]` | Use `<<<image_N>>>` / `<<<video_N>>>` tokens |

## Audio & lip-sync

| Symptom | Mechanism | Counter |
|---|---|---|
| Lips out of sync | Clip too long or head-motion words present `[FIELD-HF]` | Dialogue beat 3–8s; strip nodding/turning; MCU or tighter; one speaking face |
| Music/ambience buried the dialogue | Generated audio override `[FIELD-HF]` | Remove ambient/music tokens when sync matters; diegetic-only body |
| Generic mismatched score | "Dramatic strings"-style score adjectives `[FIELD-HF]` | Instrumentation + tempo + arc, or generate music separately ([audio.md](audio.md)) |
| Robotic TTS delivery | Flat emotional register | `emotion` param + delivery-aware text (punctuation, breath-length sentences) |

## Platform & operations `[OURS/SPEC]`

| Symptom | Mechanism | Counter |
|---|---|---|
| "Stuck at 99%" | Progress caps at 99% once elapsed passes the estimate — job is still running `[OURS]` | Reassure; check `job_display` once; only treat as failed if the job says failed |
| Insufficient credits | — | `popcraft_show_plans_and_credits` with `intent:"topup"`, relay verbatim, stop until confirmed ([account.md](account.md)) |
| Result card didn't render (CLI host) | Non-Apps client | `popcraft_job_display` once; then ETA-based check-ins per SKILL.md job contract |
| `recovery_tool` in a result | Connector's self-healing directive `[SPEC]` | Call that tool immediately — no explanation, no asking |
| Was it refunded? | Failed tasks auto-refund server-side `[SPEC]` | Point at the refund row in `popcraft_transactions` |
| Model missing from specs / behaves differently | Snapshot drift | Live `models_explore`; live wins; flag specs regeneration |

## Pre-submit self-check (60 seconds, prevents most of the above)

One action · one camera move · six slots present · constraints positive · identity
block verbatim · no timestamps inside a oner · references each given one job, none
mentioned that shouldn't appear · fps/duration/ratio legal per specs · risky register
scrubbed. Then `get_cost` and go.
