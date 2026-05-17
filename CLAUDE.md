# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working in this repository.

## What this is

A local **Claude Code plugin marketplace** named `catalonia-local`. It currently hosts one plugin (`catalonia-trip-advisor`) but the layout is designed to grow into a multi-plugin marketplace — each plugin lives in its own folder at the repo root and is its own publishable unit.

The marketplace manifest is `.claude-plugin/marketplace.json`. It declares the marketplace name, owner, and the list of plugins (with relative `source` paths into the repo).

## Plugins in this marketplace

### catalonia-trip-advisor

Travel advisor for tourists visiting **Catalonia, with a focus on Barcelona**. Ships three skills:

- **`advisor`** — the main skill. Answers itinerary, transport, day-trip, food, culture, and logistics questions. Gates every answer on a one-time preferences interview.
- **`interview`** — one-time collection of the user's travel preferences (group, pace, budget, diet, mobility, language, base). Writes `profile/spain-trip-profile.md` at the repo root.
- **`report`** — exports the latest substantial advisor answer to `exports/` as markdown or HTML with a numbered Sources footer.

**Plugin authoring rules, conventions, and skill internals live in `catalonia-trip-advisor/CLAUDE.md`.** Read that file before editing anything inside the plugin folder.

## Repo layout

```
catalonia-trip-advisor/                       # repo root
├── CLAUDE.md                                 # this file
├── .claude-plugin/marketplace.json           # local-marketplace manifest
├── docs/                                     # supplementary docs, not loaded by any plugin
├── profile/                                  # user data — written by the interview, read by the advisor (gitignored)
│   └── spain-trip-profile.md
├── exports/                                  # generated documents from the report skill (gitignored, safe to delete)
└── catalonia-trip-advisor/                   # the plugin (see its own CLAUDE.md)
    ├── .claude-plugin/plugin.json
    ├── README.md
    └── skills/{advisor,interview,report}/
```

**The repo-root marker.** Pairing `CLAUDE.md` + `.claude-plugin/marketplace.json` at the repo root is the marker plugin skills use to locate the root from any working directory: walk up from CWD until both files are present in the same directory.

**Why `profile/` and `exports/` live at the repo root** rather than inside the plugin: keeps each plugin folder a pure publishable artifact. Tradeoff to know — the profile is tied to this working tree, so a fresh clone or reinstall starts with no profile and re-triggers the interview. Both folders are gitignored; `exports/` is safe to delete and gets recreated on demand by the report skill.

## Local testing

From the repo root:

```bash
claude --plugin-dir catalonia-trip-advisor
```

Inside the session, `/reload-plugins` picks up edits without restart. Confirm the plugin's skills appear under `/help` — they're registered as `/catalonia-trip-advisor:advisor`, `/catalonia-trip-advisor:interview`, `/catalonia-trip-advisor:report` (the prefix comes from the plugin's `name` field in `plugin.json`).

There's no build, no lint, no test suite. The repo is markdown; verification is loading the plugin in a real Claude session and watching the output.

## Adding a new plugin

1. Create a new folder at the repo root (e.g. `my-other-plugin/`) with its own `.claude-plugin/plugin.json` and `skills/`.
2. Add an entry to `.claude-plugin/marketplace.json` with `name`, `source: "./my-other-plugin"`, and a one-line description.
3. Give it its own `CLAUDE.md` covering authoring rules for that plugin.
4. Add a one-paragraph blurb under **Plugins in this marketplace** above.

## Pointer

When working on plugin internals (skills, research, manifest, version bumps), read that plugin's own `CLAUDE.md` first. This file only covers repository- and marketplace-level concerns.
