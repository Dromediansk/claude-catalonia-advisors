# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Product

A Claude **skill** (or set of skills) that turns Claude into a travel advisor for tourists visiting **Catalonia, with a focus on Barcelona**. There is no frontend and no API — the user invokes the skill inside their Claude harness (Claude Code, Claude.ai, etc.) and Claude answers from the research corpus shipped with the skill.

Four topic areas:

1. **Tips** — safety, scams, tipping, opening hours, day-of logistics.
2. **Culture** — Catalan vs. Spanish identity, language, food, festivals, etiquette.
3. **Travelling** — itineraries, day trips from Barcelona (Montserrat, Girona, Costa Brava, Sitges, Tarragona), neighborhoods.
4. **Transportation** — metro, T-Casual / Hola Barcelona passes, RENFE/Rodalies, buses, airport transfers, taxis vs. ride-share.

**Scope discipline**: out-of-region questions (Madrid, Andalusia, etc.) should be politely declined. Depth on Catalonia is the whole point.

**Audience**: tourists planning or currently on a trip. Plain language, concrete logistics, short actionable answers. Gloss Catalan terms (*plaça*, *carrer*, *menú del dia*) on first use.

**Status**: in active development. Plugin is packaged with manifest, two skills, and all four research areas.

## Repository Layout

This repo ships a Claude Code **plugin** named `catalonia-trip-advisor`. The plugin lives in `catalonia-trip-advisor/` so the repo root can host docs and grow into a multi-plugin marketplace later.

```
catalonia-trip-advisor/                       # repo root
├── CLAUDE.md                                 # this file (paired with .claude-plugin/marketplace.json marks the repo root)
├── .claude-plugin/marketplace.json           # local-marketplace manifest
├── docs/                                     # supplementary, not loaded by the plugin
├── profile/                                  # user data — written by the interview, read by the advisor
│   └── spain-trip-profile.md                 # the actual profile (gitignore this if the repo becomes git-tracked)
└── catalonia-trip-advisor/                   # the plugin
    ├── .claude-plugin/plugin.json            # manifest — required for plugin discovery
    ├── README.md                             # user-facing intro
    └── skills/
        ├── advisor/
        │   ├── SKILL.md                      # main advisor — STEP 0 profile gate, stable/volatile workflow
        │   └── research/                     # supporting corpus for stable facts
        │       ├── sources.md                # authoritative URLs for WebFetch (highest-leverage file)
        │       ├── culture/  tips/  transportation/  travelling/
        └── interview/
            └── SKILL.md                      # one-time preferences interview, writes the profile
```

Once installed, the skills are registered as `/catalonia-trip-advisor:advisor` and `/catalonia-trip-advisor:interview` — the prefix comes from the `name` field in `plugin.json`.

- **`catalonia-trip-advisor/.claude-plugin/plugin.json`** — plugin manifest. Bump `version` to publish an update; without an explicit version every commit SHA counts as a release.
- **`catalonia-trip-advisor/skills/advisor/`** — entry point for the main advisor. Frontmatter triggers the skill; body is the playbook. STEP 0 resolves the repo root (the dir holding both `CLAUDE.md` and `.claude-plugin/marketplace.json`), checks `profile/spain-trip-profile.md` there, and invokes the interview if missing.
- **`catalonia-trip-advisor/skills/advisor/research/`** — the source of truth for stable facts, referenced from the advisor SKILL.md with plain relative paths (`research/...`). Lives **inside** the advisor skill, matching how Anthropic's own plugins (e.g. `skill-creator`, `math-olympiad`) co-locate supporting files. `${CLAUDE_SKILL_DIR}` resolves to this skill's folder.
- **`catalonia-trip-advisor/skills/interview/`** — sub-skill. Collects user preferences (group, pace, interests, budget, diet, mobility, language, base) and writes them to `profile/spain-trip-profile.md` at the repo root. Invoked implicitly on first run and explicitly via "update my Catalonia preferences". No supporting files of its own.
- **`docs/`** — design specs under `docs/superpowers/specs/`, example outputs (e.g. `transport-tickets-4day-couple.md`). Not loaded by the plugin.
- **`profile/spain-trip-profile.md`** — user-data file the interview writes and the advisor reads. Lives at the repo root (alongside `CLAUDE.md` and `docs/`), not inside the plugin, so the plugin folder stays a pure publishable artifact. Tradeoff to know: the profile is tied to this working tree — a fresh clone or reinstall starts with no profile and re-triggers the interview. Resolved at runtime by Bash walking up from CWD until it finds a directory containing both `CLAUDE.md` and `.claude-plugin/marketplace.json`.

## Authoring Rules

- **The `description` field in `SKILL.md` frontmatter is the trigger.** Be specific about the phrases that should activate it ("Barcelona transport", "Catalonia day trip", "is the tap water safe in Barcelona", etc.). Vague descriptions never fire.
- **Keep `SKILL.md` short.** It's loaded into every relevant turn. Put bulk content in `research/` and reference it.
- **Stable vs. volatile split** — the core design rule of this skill:
  - **Stable** (concepts, history, neighborhood character, dishes, scam patterns, general line maps) → lives in `research/` (inside the advisor skill), cited by file.
  - **Volatile** (prices, fares, hours, schedules, dated festivals, ticket availability) → **WebFetched live** from URLs in `research/sources.md`, cited with fetch date. **Never quoted from a research file.** A stale price is the failure mode this skill exists to prevent.
  - When in doubt, treat a fact as volatile.
- **Frontmatter on research files**: `title`, `topic` (one of the four areas, or `meta` for `sources.md`), `last_verified` (YYYY-MM-DD).
- **Catalan-aware**: prefer Catalan place names (Plaça de Catalunya, not Plaza de Cataluña) but acknowledge the Spanish form on first mention.
- **Research edits are content changes, not code changes** — reviewable by non-engineers. `sources.md` is the highest-leverage file: adding a new authoritative URL there expands what the skill can verify.

## Working in This Repo

- No build, no lint, no tests — it's markdown. Verification is loading the plugin in a real Claude session and watching the output.
- **Local testing**: from the repo root, run `claude --plugin-dir catalonia-trip-advisor`. After editing any file, run `/reload-plugins` inside the session to pick up changes without restarting. Confirm both skills appear under `/help`.
- **Adding research files**: drop them under `catalonia-trip-advisor/skills/advisor/research/<topic>/` and add a one-line pointer from the advisor's `SKILL.md` so Claude can discover them. Files Claude can't discover are dead weight.
- **Adding sources**: add the authoritative URL to `catalonia-trip-advisor/skills/advisor/research/sources.md`. This is the highest-leverage edit — it expands what volatile facts the skill can verify live.
- The STEP 0 profile gate at the top of the advisor's `SKILL.md` is load-bearing — it enforces the interview sub-skill before any answer. Don't trim it as "preamble"; the rationalization table inside it documents real test failures.
- Bump `version` in `catalonia-trip-advisor/.claude-plugin/plugin.json` on releases worth a fresh install for downstream users.

## Reference

See the `superpowers:writing-skills` skill for the canonical authoring conventions (description targeting, lazy-loading references, frontmatter format).
