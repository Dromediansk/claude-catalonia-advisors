# catalonia-advisors

A Claude Code **plugin marketplace** for getting around — and settling into — Catalonia. It ships two plugins:

- **[catalonia-trip-advisor](./catalonia-trip-advisor)** — travel advisor for tourists in Catalonia and Barcelona: itineraries, day trips, transport, neighborhoods, food, scams, and culture.
- **[catalonia-real-estate](./catalonia-real-estate)** — buying advisor for anyone purchasing property in Catalonia: buying process, price/area benchmarks, taxes and fees, resident vs. non-resident financing, and live listing search across Idealista, Fotocasa, and Habitaclia.

## Install via the marketplace

Add this repo as a marketplace, then install whichever plugin you want. Works from inside a Claude Code session (slash commands) or from your shell (the `claude` CLI) — both accept the same `owner/repo` shorthand.

### From inside Claude Code

```text
/plugin marketplace add Dromediansk/trip-advisor-claude-marketplace
/plugin install catalonia-trip-advisor@catalonia-advisors
/plugin install catalonia-real-estate@catalonia-advisors
```

`/plugin marketplace add` clones the repo and reads its `.claude-plugin/marketplace.json`, which registers the marketplace under the name **`catalonia-advisors`** — that's the name after the `@` when you install. Install only the plugin you need; you don't have to add both.

Prefer to browse? Run `/plugin` for the interactive menu (Discover / Installed / Marketplaces tabs), find the plugins under `catalonia-advisors`, and enable them there.

After installing, run `/reload-plugins` (or restart Claude Code) and confirm the skills show up under `/help` as `/catalonia-trip-advisor:{advisor,interview,report}` and `/catalonia-real-estate:{advisor,interview,report}`.

### From your shell

```bash
claude plugin marketplace add Dromediansk/trip-advisor-claude-marketplace
claude plugin install catalonia-trip-advisor@catalonia-advisors
claude plugin install catalonia-real-estate@catalonia-advisors
```

Useful companions: `claude plugin marketplace list` (see configured marketplaces), `claude plugin marketplace update catalonia-advisors` (refresh listings after the repo updates), and `claude plugin marketplace remove catalonia-advisors` (remove it).

### Specific branch or tag

Append `#ref` to the source to pin a branch or tag:

```text
/plugin marketplace add https://github.com/Dromediansk/trip-advisor-claude-marketplace.git#main
```

## Using the plugins

You usually don't invoke skills by name — just ask. A Catalonia travel question ("Barcelona for 5 days", "is the T-Casual worth it") fires the trip advisor; a property question ("buy a flat in Gràcia under 400k", "can a non-resident get a mortgage") fires the real-estate advisor. Each plugin runs a one-time preferences interview on first use, then tailors every answer to your profile. See each plugin's own README for what it covers and how it works.

## Local development

Working on the plugins themselves? Load one ephemerally without going through the marketplace:

```bash
claude --plugin-dir ./catalonia-trip-advisor   # or: ./catalonia-real-estate
```

`/reload-plugins` picks up edits without a restart. See [CLAUDE.md](./CLAUDE.md) for repo layout and authoring conventions, and each plugin's `CLAUDE.md` for its internals.
