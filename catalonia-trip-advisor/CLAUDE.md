# CLAUDE.md — `catalonia-trip-advisor` plugin

Authoring guide for Claude when editing this plugin. End-user install and behavior overview live in `README.md` — this file does **not** duplicate them. Repository-level and marketplace concerns live in the parent `../CLAUDE.md`.

## Product

A Claude skill that turns Claude into a travel advisor for tourists visiting **Catalonia, with a focus on Barcelona**. Four topic areas:

1. **Tips** — safety, scams, tipping, opening hours, day-of logistics.
2. **Culture** — Catalan vs. Spanish identity, language, food, festivals, etiquette.
3. **Travelling** — itineraries, day trips from Barcelona (Montserrat, Girona, Costa Brava, Sitges, Tarragona), neighborhoods.
4. **Transportation** — metro, T-Casual / Hola Barcelona passes, RENFE/Rodalies, buses, airport transfers, taxis vs. ride-share.

**Scope discipline**: out-of-region questions (Madrid, Andalusia, etc.) are politely declined. Depth on Catalonia is the whole point.

**Audience**: tourists planning or currently on a trip. Plain language, concrete logistics, short actionable answers. Gloss Catalan terms (*plaça*, *carrer*, *menú del dia*) on first use.

**Status**: in active development.

## Plugin layout

```
catalonia-trip-advisor/
├── .claude-plugin/plugin.json      # manifest — bump `version` to publish an update
├── README.md                       # user-facing intro (install, how it works)
├── CLAUDE.md                       # this file
└── skills/
    ├── advisor/
    │   ├── SKILL.md                # main advisor — STEP 0 profile gate, stable/volatile workflow
    │   └── research/               # supporting corpus for stable facts
    │       ├── sources.md          # authoritative URLs for WebFetch (highest-leverage file)
    │       └── culture/  tips/  transportation/  travelling/
    ├── interview/
    │   └── SKILL.md                # one-time preferences interview, writes the profile
    └── report/
        ├── SKILL.md                # turns the latest advisor answer into a markdown/HTML doc
        └── assets/
            ├── template.html       # standalone HTML template with embedded CSS
            └── style-notes.md      # markdown layout cheat sheet
```

## The three skills

- **`advisor`** — entry point. Frontmatter description is the trigger; body is the playbook. **STEP 0** at the top resolves the repo root (walks up from CWD looking for a dir containing both `CLAUDE.md` and `.claude-plugin/marketplace.json`), checks `profile/spain-trip-profile.md`, and invokes the `interview` skill if missing. STEP 0 is load-bearing — don't trim it as "preamble". The rationalization table inside it documents real test failures.
- **`interview`** — sub-skill. Collects user preferences (group, pace, interests, budget, diet, mobility, language, base) and writes them to `profile/spain-trip-profile.md` at the repo root. Invoked implicitly on first advisor run and explicitly via phrases like "update my Catalonia preferences".
- **`report`** — sub-skill. Turns the most recent substantial advisor answer into a clean, source-cited document in `exports/`. Asks the user for markdown or HTML, never re-fetches volatile facts, renders inline superscripts plus a numbered Sources footer (with fetch dates) so a tourist can verify any claim by clicking the original URL.

## Core authoring rule: stable vs. volatile

This is the central design rule of the plugin:

- **Stable** (concepts, history, neighborhood character, dishes, scam patterns, general line maps) → lives in `skills/advisor/research/`, cited by file path.
- **Volatile** (prices, fares, hours, schedules, dated festivals, ticket availability) → **WebFetched live** from URLs in `research/sources.md`, cited with fetch date. **Never quoted from a research file.** A stale price is the failure mode this plugin exists to prevent.
- When in doubt, treat a fact as volatile.

## Research folder

Lives under `skills/advisor/research/{culture,tips,transportation,travelling}/`, with `sources.md` at the top level. The `research/` folder lives **inside** the advisor skill (matching how Anthropic's own plugins co-locate supporting files — `${CLAUDE_SKILL_DIR}` resolves to this skill's folder).

- **`sources.md` is the highest-leverage file.** Adding an authoritative URL expands what volatile facts the plugin can verify live. Edits here are content changes, reviewable by non-engineers.
- **Adding a new research file**: drop it under the relevant topic folder and add a one-line pointer from `advisor/SKILL.md` so Claude can discover it. Files Claude can't discover are dead weight.

## Skill-specific assets

Static files used by a single skill (HTML templates, style cheatsheets) live in `<skill>/assets/` — e.g. `skills/report/assets/template.html`. Reference them with relative paths from `SKILL.md` (`assets/template.html`) or with `${CLAUDE_SKILL_DIR}/assets/...` for absolute resolution. Like research files, these are content, not code.

## Frontmatter

- **`SKILL.md` description = the trigger.** Be specific about phrases that should activate it ("Barcelona transport", "Catalonia day trip", "is the tap water safe in Barcelona"). Vague descriptions never fire.
- **Keep `SKILL.md` short.** It's loaded into every relevant turn. Bulk content goes in `research/` and is referenced.
- **Research file frontmatter**: `title`, `topic` (one of the four areas, or `meta` for `sources.md`), `last_verified` (YYYY-MM-DD).

## Catalan-aware language

Prefer Catalan place names (Plaça de Catalunya, *not* Plaza de Cataluña) but acknowledge the Spanish form on first mention. Gloss Catalan terms (*plaça*, *carrer*, *menú del dia*) the first time they appear in an answer.

## Releasing

Bump `version` in `.claude-plugin/plugin.json` on releases worth a fresh install for downstream users. Without an explicit version bump, every commit SHA counts as a release.

## Reference

See the `superpowers:writing-skills` skill for the canonical skill authoring conventions (description targeting, lazy-loading references, frontmatter format).
