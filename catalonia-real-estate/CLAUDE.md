# CLAUDE.md — `catalonia-real-estate` plugin

Authoring guide for Claude when editing this plugin. End-user install and behavior overview live in `README.md` — this file does **not** duplicate them. Repository-level and marketplace concerns live in the parent `../CLAUDE.md`.

## Product

A Claude skill set that turns Claude into an advisor for people **buying real estate anywhere in Catalonia** — Barcelona and its neighborhoods, the wider province, Girona, Tarragona, Lleida, the Costa Brava/Daurada, and inland towns. No single-city bias. Three concerns:

1. **Buying advice** — the process, area and price benchmarks, taxes and fees (ITP, IVA/AJD, notary, registry), resident vs. non-resident mortgage financing and LTV, NIE and legal steps.
2. **Live listing search** — profile-scoped queries across Idealista, Fotocasa, and Habitaclia via WebSearch/WebFetch; a handful of representative listings with price, location, link, and a profile-fit note.
3. **Export** — turn a substantial answer into a markdown/HTML document with a numbered Sources footer.

**Scope discipline**: property outside Catalonia (Madrid, Valencia, etc.) is politely declined.

**Audience**: relocators, investors, and holiday-home buyers. Plain language, concrete logistics, short actionable answers. Gloss Catalan/Spanish terms (*gestor*, *advocat*, ITP, NIE) on first use.

**Status**: in active development.

## Plugin layout

```
catalonia-real-estate/
├── .claude-plugin/plugin.json      # manifest — bump `version` to publish an update
├── README.md                       # user-facing intro (install, how it works)
├── CLAUDE.md                       # this file
└── skills/
    ├── advisor/SKILL.md            # main advisor — STEP 0 profile gate, buying advice, live listing search, disclaimers
    ├── interview/SKILL.md          # one-time buyer-profile interview (7 questions), writes the profile
    └── report/
        ├── SKILL.md                # turns the latest advisor answer into a markdown/HTML doc
        └── assets/
            ├── template.html       # standalone HTML template with embedded CSS
            └── style-notes.md      # markdown layout cheat sheet
```

## The three skills

- **`advisor`** — entry point. Frontmatter description is the trigger; body is the playbook. **STEP 0** at the top resolves the repo root (walks up from CWD looking for a dir containing both `CLAUDE.md` and `.claude-plugin/marketplace.json`), checks `profile/catalonia-realestate-profile.md`, and invokes the `interview` skill if missing. STEP 0 is load-bearing — don't trim it. The rationalization table inside it documents real failure modes.
- **`interview`** — sub-skill. Collects buyer preferences (intent, budget, financing, legal status, location, property, timeline+language) and writes `profile/catalonia-realestate-profile.md` at the repo root. Invoked implicitly on first advisor run and explicitly via "update my Catalonia property preferences".
- **`report`** — sub-skill. Turns the most recent substantial advisor answer into a clean, source-cited document in `exports/`. Asks markdown or HTML, never re-fetches or re-searches, carries the disclaimer and listing time-sensitivity caveat, and renders inline superscripts plus a numbered Sources footer.

## Core authoring rule: stable vs. volatile

This is the central design rule of the plugin:

- **Stable** (what a tax/step *is*, the general notary→registry sequence, area character, what a *gestor* does) → explain in plain general language, always with the disclaimer.
- **Volatile** (current tax rates/bands, mortgage rates, LTV caps, notary/registry/agency fees, and every listing price/availability/link) → **fetched live** via WebSearch/WebFetch from an authoritative source, cited with fetch date. **Never quoted from memory.** A stale rate or price on a property purchase is the failure mode this plugin exists to prevent.
- When in doubt, treat a fact as volatile.

The advisor SKILL.md holds the portal list (Idealista/Fotocasa/Habitaclia) and the authoritative-source table (ATC for ITP, AEAT, NIE procedures, registry, notary) **inline** — this plugin deliberately has no separate `research/` corpus. Adding an authoritative URL there expands what the advisor can verify live.

## Mandatory disclaimers

Higher stakes than tourism. Two non-negotiable rules baked into the skills:

- **Not legal/financial/tax advice.** Every answer touching process, money, tax, or legal status closes with a short disclaimer recommending a licensed *gestor*, a property lawyer, and the buyer's own bank. The `report` skill carries it into exported documents.
- **Listings are time-sensitive, best-effort.** Every listing answer states that prices and availability change, listings may be under offer, and links can expire. Never present a fetched price as guaranteed. If a search returns nothing usable, say so and fall back to guidance — never invent listings.

## Frontmatter

- **`SKILL.md` description = the trigger.** Be specific about buyer phrases ("buy a flat in Barcelona", "non-resident mortgage", "what taxes buying a house in Catalonia", "2-bed under 400k near the beach"). Vague descriptions never fire.
- **Keep `SKILL.md` focused.** It's loaded into every relevant turn.

## Catalan-aware language

Prefer Catalan place names (Gràcia, Eixample, Girona) but acknowledge the Spanish form on first mention. Gloss Catalan/Spanish terms (*gestor*, *advocat*, ITP, AJD, NIE) the first time they appear in an answer.

## Releasing

Bump `version` in `.claude-plugin/plugin.json` on releases worth a fresh install. Without an explicit version bump, every commit SHA counts as a release.

## Reference

See the `superpowers:writing-skills` skill for the canonical skill authoring conventions. The sibling `../catalonia-trip-advisor/` plugin is the structural template this one mirrors.
