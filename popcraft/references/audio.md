# Audio — popcraft_generate_audio + popcraft_list_voices

Standalone audio: speech, sound effects, music. For audio INSIDE a video (native audio,
lip-sync) see [video.md](video.md) § Native audio. Model facts: [../specs/model-specs.md](../specs/model-specs.md).

## Tool contract `[SPEC]`

`popcraft_generate_audio`:

| Param | Notes |
|---|---|
| `model` | Audio model id from specs |
| `kind` | `"tts"` (default) · `"sfx"` · `"music"` |
| `prompt` | tts: the text to speak · sfx/music: description of the sound. With `lyrics` set (song mode) prompt is ignored — a short style line is fine |
| `voice_id` | tts only — from `popcraft_list_voices`; omit = model default. **Never invent a voice id** |
| `duration` | Seconds, for sfx/music. Model ranges `[SPEC]`: SFX 0.5–22 · `music-v1` 3–200. Fixed-length models (`lyria-3-*`, song models) ignore it. When the user states a length, always pass it explicitly |
| `lyrics` / `tags` / `title` / `instrumental` / `clip_index` / `keep_all_takes` | Song mode (see below) |
| `project_id` / `project_name`, `get_cost` | As in video/image |

`popcraft_list_voices`: filter by `gender` / `language`, `limit` ≤50 (default 20). On
Apps hosts it renders a listen-and-pick widget — prefer letting the user pick.
⚠️ The voice catalog stores language **codes** — pass `language:"en"`, not `"english"`
(the long form returns an empty list `[OURS — verified live 2026-07-16]`); on an empty result, retry
with a looser filter before concluding no voices exist.

Delivery: same job contract (self-refreshing card, `job_display` ≤ once).

## Model matrix `[SPEC]`

| Need | Pick | Notes |
|---|---|---|
| Voiceover / narration | `elevenlabs-tts` (1 cr, ~8s) or `popcraft-audio` (router) | ⚠️ the `emotion` / `speech_rate` capabilities in specs are **web-UI params, NOT accepted by `popcraft_generate_audio`** `[OURS — verified live 2026-07-16]`; carry emotion via voice choice + text/punctuation |
| Sound effect | `elevenlabs-sfx` (2 cr) | duration 0.5–22s, `prompt_influence` 0–1 (default 0.3) |
| Background music, exact length | `music-v1` (1 cr) | duration 3–200s, `force_instrumental` default true |
| Music clip, quality | `lyria-3-clip` (8 cr) / `lyria-3-pro` (16 cr) | No duration param — fixed-length |
| Full song with vocals | `suno-chirp-v5` — ⚠️ documented by the connector but **absent from the models_explore catalog** (2026-07-16). Verify live with `models_explore query:"suno"` before offering; if absent, say so | Song mode params below |

## TTS craft

- **Voice first:** `list_voices` (filtered to the user's language code) → let them pick.
- **Delivery is voice choice + text technique:** pick a voice whose tags match the
  register (warm/professional/playful); inside the text, punctuation and phrasing carry
  pacing. (No emotion/rate params exist on the MCP tool — see the matrix note.)
- **Delivery honesty:** audio results may omit `project` echo and duration metadata
  `[OURS — verified live 2026-07-16]` — fill the result template with what the job actually returned,
  never invent the missing fields.
- Write for the ear: short breath-length sentences; numbers and abbreviations spelled
  out the way they should be spoken.
- Multi-segment narration: keep `voice_id`, `emotion`, and `speech_rate` identical
  across segments — consistency is a parameter job, not luck.

## SFX craft

- Describe the sound **event**, not a category: *"a heavy wooden door creaks open
  slowly, old iron hinges"* beats "door sound".
- Enumerated specifics shape output: *"frosted glass scratch, bubble-wrap pop"* level
  of concreteness. `[FIELD-HF]`
- One effect per generation; layered soundscapes = separate generations mixed in post.
- `prompt_influence`: raise toward 1.0 for literal prompt adherence, keep ~0.3 for
  natural variation.

## Music craft

- Describe **instrumentation + tempo + mood + arc** — never song titles or artist names
  (filter risk + weak steering): *"lo-fi hip hop, 80 BPM, warm vinyl crackle, mellow
  Rhodes chords, builds gently after the midpoint"*.
- Structure requests with entry/exit intent: intro-friendly start, loopable middle, or
  a build-into-drop arc, depending on where it sits in the user's edit.
- `music-v1` honors exact `duration` (3–200s) — match the user's edit length precisely
  rather than trimming in post.

### Song mode (`suno-chirp-v5`, when available)

- `prompt` = describe-the-song mode; OR `lyrics` = it sings YOUR words (mark sections
  `[Verse]` / `[Chorus]`), with `tags` as the strongest steering lever (genre, mood,
  vocal type — e.g. "synthwave, energetic, female vocals"), optional `title`.
- `instrumental` applies only in describe mode (ignored with `lyrics`).
- Suno renders **two takes per generation at no extra cost** — both kept by default
  (`keep_all_takes`); let the user listen and pick, `clip_index` sets the primary.

## Radio-drama pattern (multi-voice scenes)

The MCP generates one voice per TTS call — build scenes by composition `[FIELD-HF]`:
1. Script with speaker labels + emotion parentheticals per line.
2. One `generate_audio` call per line/speaker (stable `voice_id` per character).
3. SFX + music as separate generations; the user mixes in their editor.
Test one short segment per voice before generating the full scene — voice fit is
cheaper to fix at line 1 than line 40.

## Boundaries

- **Voice cloning is not exposed via MCP** → popcraft.ai → Audio → Clone Voice
  (SKILL.md § Capability boundaries).
- Driving a video's lip-sync from generated speech is a video-model workflow — see
  [video.md](video.md); generate the dialogue line THERE (scripted in-prompt), not here.
