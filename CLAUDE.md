# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working in this repository.

## What this is

A local **Claude Code plugin marketplace** named `catalonia-local`. It hosts two plugins (`catalonia-trip-advisor` and `catalonia-real-estate`) and the layout is designed to grow further — each plugin lives in its own folder at the repo root and is its own publishable unit.

The marketplace manifest is `.claude-plugin/marketplace.json`. It declares the marketplace name, owner, and the list of plugins (with relative `source` paths into the repo).

## Plugins in this marketplace

### catalonia-trip-advisor

Travel advisor for tourists visiting **Catalonia, with a focus on Barcelona**. Ships three skills:

- **`advisor`** — the main skill. Answers itinerary, transport, day-trip, food, culture, and logistics questions. Gates every answer on a one-time preferences interview.
- **`interview`** — one-time collection of the user's travel preferences (group, pace, budget, diet, mobility, language, base). Writes `profile/spain-trip-profile.md` at the repo root.
- **`report`** — exports the latest substantial advisor answer to `exports/` as markdown or HTML with a numbered Sources footer.

**Plugin authoring rules, conventions, and skill internals live in `catalonia-trip-advisor/CLAUDE.md`.** Read that file before editing anything inside the plugin folder.

### catalonia-real-estate

Advisor for people **buying real estate anywhere in Catalonia** (Barcelona, Girona, Tarragona, Lleida, the coast, inland towns — no single-city bias). Ships the same three-skill structure as the trip advisor:

- **`advisor`** — the main skill. Answers buying-process, price/area benchmark, tax/fee, and resident-vs-non-resident financing questions, and runs **live listing search** across Idealista, Fotocasa, and Habitaclia. Gates every answer on a one-time buyer-profile interview, and carries mandatory not-legal/financial/tax disclaimers.
- **`interview`** — one-time collection of the buyer's profile (intent, budget, financing, legal status, location, property, timeline, language). Writes `profile/catalonia-realestate-profile.md` at the repo root.
- **`report`** — exports the latest substantial advisor answer to `exports/` as markdown or HTML with a numbered Sources footer.

**Plugin authoring rules and skill internals live in `catalonia-real-estate/CLAUDE.md`.** Read that file before editing anything inside the plugin folder.

## Repo layout

```
catalonia-trip-advisor/                       # repo root
├── CLAUDE.md                                 # this file
├── README.md                                 # marketplace intro + install instructions
├── .claude-plugin/marketplace.json           # local-marketplace manifest
├── profile/                                  # user data — written by each plugin's interview, read by its advisor (gitignored)
│   ├── spain-trip-profile.md                 # catalonia-trip-advisor
│   └── catalonia-realestate-profile.md       # catalonia-real-estate
├── exports/                                  # generated documents from the report skills (gitignored, safe to delete)
├── catalonia-trip-advisor/                   # plugin (see its own CLAUDE.md)
│   ├── .claude-plugin/plugin.json
│   ├── README.md
│   └── skills/{advisor,interview,report}/
└── catalonia-real-estate/                    # plugin (see its own CLAUDE.md)
    ├── .claude-plugin/plugin.json
    ├── README.md
    └── skills/{advisor,interview,report}/
```

**The repo-root marker.** Pairing `CLAUDE.md` + `.claude-plugin/marketplace.json` at the repo root is the marker plugin skills use to locate the root from any working directory: walk up from CWD until both files are present in the same directory.

**Why `profile/` and `exports/` live at the repo root** rather than inside the plugin: keeps each plugin folder a pure publishable artifact. Tradeoff to know — the profile is tied to this working tree, so a fresh clone or reinstall starts with no profile and re-triggers the interview. Both folders are gitignored; `exports/` is safe to delete and gets recreated on demand by the report skill.

## Local testing

From the repo root, load whichever plugin you're working on:

```bash
claude --plugin-dir catalonia-trip-advisor   # or: catalonia-real-estate
```

Inside the session, `/reload-plugins` picks up edits without restart. Confirm the plugin's skills appear under `/help` — they're registered under the plugin's `name` field in `plugin.json` as `/catalonia-trip-advisor:{advisor,interview,report}` and `/catalonia-real-estate:{advisor,interview,report}` respectively.

There's no build, no lint, no test suite. The repo is markdown; verification is loading the plugin in a real Claude session and watching the output.

## Adding a new plugin

1. Create a new folder at the repo root (e.g. `my-other-plugin/`) with its own `.claude-plugin/plugin.json` and `skills/`.
2. Add an entry to `.claude-plugin/marketplace.json` with `name`, `source: "./my-other-plugin"`, and a one-line description.
3. Give it its own `CLAUDE.md` covering authoring rules for that plugin.
4. Add a one-paragraph blurb under **Plugins in this marketplace** above.

## Pointer

When working on plugin internals (skills, research, manifest, version bumps), read that plugin's own `CLAUDE.md` first. This file only covers repository- and marketplace-level concerns.
