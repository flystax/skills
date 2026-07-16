# PopCraft Skills

[![Skill version](https://img.shields.io/badge/popcraft-v0.2.0-blue)](popcraft/SKILL.md)
[![Specs snapshot](https://img.shields.io/badge/model%20specs-2026--07--16-informational)](popcraft/specs/model-specs.md)
[![License](https://img.shields.io/badge/license-Apache--2.0-green)](LICENSE)
[![Platform](https://img.shields.io/badge/works%20with-Claude%20Code%20%7C%20claude.ai%20%7C%20Cursor%20%7C%20Cowork-purple)](https://popcraft.ai/mcp)

Official agent skills for [PopCraft](https://popcraft.ai) — the multi-model AI content
creation platform (video · image · audio · upscaling · effects).

**One skill today:** [`popcraft/`](popcraft/SKILL.md) — the **director-operator**. It
turns any MCP-connected agent into a PopCraft creative director that writes
expert prompts for you, an operator that confirms the price before spending a single
credit, and an editor that diagnoses misses instead of blindly re-rolling.

## What can you do with it?

Just tell your agent what you want:

| You say… | You get… |
|---|---|
| "Make a 10-second video of a cat surfing at sunset" | A finished video clip, filed into your PopCraft project |
| "Animate this product photo into a luxury ad" | Your photo imported and brought to life (image-to-video) |
| "A poster with the headline 'FIRST POUR FREE', warm vintage feel" | A text-perfect poster from the best text-rendering model |
| "Turn this script into a warm female voiceover" | A voice picked with you, then a finished MP3 |
| "Upscale this to 4K" | A Topaz-enhanced version of any previous result |
| "Continue this clip — what happens next?" | A seamless extension of your video |
| "Which model should I use for a 15-second 21:9 chase scene?" | A fact-checked recommendation from the live catalog |
| "Just write me the prompt, don't generate" | A director-grade prompt, zero credits spent |
| "How many credits do I have? What was I charged?" | Balance, plan, and a verified transaction history |

**Combined workflows** are where it gets fun — describe a goal and the agent chains the
tools: *"Launch kit for my new coffee blend"* → poster with your headline → the poster
animated into a 9:16 teaser → a voiceover to lay over it → everything upscaled and
filed into one project, with the total credits reported as it goes.

## How it works

1. **Connect PopCraft** — add the PopCraft MCP connector: [popcraft.ai/mcp](https://popcraft.ai/mcp)
2. **Install the skill**
   ```bash
   npx skills add flystax/skills          # any Agent-Skills-compatible CLI
   # or for Claude Code, clone into your skills folder:
   git clone https://github.com/flystax/skills ~/.claude/skills/popcraft
   ```
   *claude.ai Projects / Cowork:* drop the `popcraft/` folder into your workspace; the
   entry point is `popcraft/SKILL.md`.
3. **Create** — describe what you want in plain language, in any language.

## Why prompts from this skill come out better

Every request runs a three-stage pipeline:

```
DIRECTOR   →  routes your intent, picks the right model from the live catalog,
              writes a structured prompt (Subject · Action · Scene · Camera · Style
              + Audio) with model-specific craft — Seedance's positive-only register,
              Kling's reference tokens, Veo's exact duration enum…
OPERATOR   →  previews the exact credit cost, confirms with you ONCE, generates,
              and delivers the result with a link to where it's filed
ITERATE    →  if you don't like it, it diagnoses the symptom (static motion? identity
              drift? warped text?) and changes the one thing that failed
```

The craft layer isn't guesswork: it's built from a **dated snapshot of the live model
catalog** ([specs/model-specs.md](popcraft/specs/model-specs.md) — 34 models with real
durations, ratios, resolutions, and prices) plus field-tested prompting knowledge, with
every unverified claim explicitly tagged. If the snapshot is more than 30 days old, the
skill re-checks the live catalog before quoting anything.

## Your credits are protected

The skill's hard rules, in plain terms:

- **No surprise spend.** The exact credit cost is previewed and confirmed with you
  before the first paid call of a session — *always*, even if you say "skip the
  questions" (you can't consent to a price you haven't seen). After that, opt into
  auto-proceed and it stops asking but keeps reporting each cost.
- **Don't want to be asked at all? Say "express mode".** An explicit opt-in switches
  to quality-first, zero-questions operation: the skill picks the best models and
  generates immediately, reporting each item's cost as it's delivered. A safety
  ceiling still applies (it checks in if a single task exceeds 300 credits or the
  session's express spend crosses 1,000 — set your own with "express mode, budget
  3000"). Say "standard mode" to switch back. Express never activates by inference —
  only when you ask for it.
- **Prices come from the API, never from memory.** `get_cost` previews are free.
- **Failures are reported honestly** — failed tasks auto-refund server-side, and the
  skill points you at the transaction ledger to verify rather than promising numbers.
- **No fake capabilities.** Things PopCraft's connector can't do yet (voice cloning,
  true background removal, talking avatars) are answered honestly with a pointer to
  the web app — never simulated.

## What's inside

```
popcraft/
├── SKILL.md                        ← entry point: rules, routing, budget ritual
├── specs/model-specs.md            ← dated live-catalog snapshot (source of truth)
└── references/
    ├── prompt-craft.md             ← the director's handbook (per-model prompt craft)
    ├── negative-constraints.md     ← artifact prevention, phrased positively
    ├── video.md · image.md · audio.md · upscale.md
    ├── media.md                    ← getting your files/URLs into generations
    ├── library.md                  ← history, saved elements, effects, projects
    ├── account.md                  ← credits, plans, transactions
    └── troubleshoot.md             ← symptom · mechanism · counter
```

Progressive disclosure by design: a simple image request loads two files, not twelve —
your agent's context (and your token bill) stays small.

## Requirements

- A [PopCraft](https://popcraft.ai) account (free tier works — several models cost 0–2 credits)
- The PopCraft MCP connector ([popcraft.ai/mcp](https://popcraft.ai/mcp)) available to your agent
- No Python, no CLI, no API keys in files — the skill is pure markdown; the connector handles auth

## Versioning & freshness

The model catalog changes often. The specs snapshot in this repo is dated in its header
and in the badge above; the skill self-heals by checking the live catalog whenever the
snapshot is stale or contradicted. If you spot a drift, an issue or PR that regenerates
`specs/model-specs.md` (procedure documented at the bottom of that file) is welcome.

## License

[Apache-2.0](LICENSE)
