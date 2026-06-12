# catalonia-real-estate

A Claude Code plugin that turns Claude into an advisor for people **buying real estate anywhere in Catalonia** — Barcelona neighborhoods, the wider province, Girona, Tarragona, Lleida, the Costa Brava/Daurada, and inland towns. Buying process, area and price benchmarks, taxes and fees, resident vs. non-resident financing, NIE and legal steps, and **live listing search** across Idealista, Fotocasa, and Habitaclia.

Out-of-region questions (property in Madrid, Valencia, etc.) are politely declined — depth on Catalonia is the point.

**This is general information, not legal, financial, or tax advice.** For binding decisions, use a licensed *gestor*, a property lawyer, and your own bank.

## Install

### Quick try (ephemeral)

From the repo root, launch one session with the plugin loaded:

```bash
claude --plugin-dir ./catalonia-real-estate
```

Good for a smoke test; the plugin is not loaded by plain `claude`.

### Persistent local install (development)

Register the repo as a local marketplace and enable the plugin globally. After this, plain `claude` loads it in every session.

1. The repo root ships a marketplace manifest at `.claude-plugin/marketplace.json` that declares this plugin alongside `catalonia-trip-advisor`.
2. Add the marketplace and enable the plugin in `~/.claude/settings.json`:

   ```json
   {
     "extraKnownMarketplaces": {
       "catalonia-local": {
         "source": {
           "source": "directory",
           "path": "/absolute/path/to/catalonia-trip-advisor"
         }
       }
     },
     "enabledPlugins": {
       "catalonia-real-estate@catalonia-local": true
     }
   }
   ```

   Point `path` at the **repo root** (the folder containing `.claude-plugin/marketplace.json`), not the inner plugin folder.

3. Restart Claude Code, or run `/reload-plugins` in an existing session.

After edits to skill files, `/reload-plugins` picks up changes without restarting — the marketplace points at your working tree.

### Marketplace

Once published, install via `/plugin install catalonia-real-estate` (or the equivalent your marketplace surfaces).

## Skills

| Skill ID                                 | Purpose                                                                                                                   |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `/catalonia-real-estate:advisor`         | The main advisor. Triggers on Catalonia property questions. Starts every turn with a buyer-profile gate.                  |
| `/catalonia-real-estate:interview`       | One-time buyer-profile interview. Runs automatically on first use; rerun explicitly with "update my Catalonia property preferences". |
| `/catalonia-real-estate:report`          | Saves the latest substantial advisor answer to `<repo-root>/exports/` as markdown or HTML, with a numbered Sources footer. Triggers on "save this as markdown", "export this", "give me an HTML file". |

You usually don't invoke them by name — just ask a Catalonia property question ("buy a flat in Gràcia under 400k", "can a non-resident get a mortgage", "what taxes do I pay buying a resale home") and the advisor fires.

## How it works

1. **Buyer-profile gate.** Every advisor turn first locates the repo root (the directory containing both `CLAUDE.md` and `.claude-plugin/marketplace.json`) and checks `profile/catalonia-realestate-profile.md` there. If it's missing, the interview skill runs once (seven short questions, one at a time), saves your preferences (intent, budget, financing, legal status, location, property, timeline, language), and hands back to the advisor.
2. **Stable vs. volatile facts.**
   - **Stable** (what a tax or step *is*, area character, the general process) → explained in plain language with a disclaimer.
   - **Volatile** (tax rates, mortgage rates, LTV caps, fees, and every listing) → **fetched live** via WebSearch/WebFetch and cited with the fetch date. Never quoted from memory.
3. **Live listing search.** The advisor builds profile-scoped queries (area, budget, type), searches Idealista/Fotocasa/Habitaclia, and summarizes a handful of representative listings with price, location, link, and a profile-fit note — flagging that listings are time-sensitive and links may expire.
4. **Profile-shaped answers.** Intent, budget, financing status, and location filter the advice and scope the search — applied silently, never echoed back at you.

**Disclaimers:** every answer touching process, money, tax, or legal status carries a not-legal/financial/tax disclaimer; listing answers carry a time-sensitivity caveat. These are built into the skills, not optional.

## Profile location

`<repo-root>/profile/catalonia-realestate-profile.md` — plain markdown, edit by hand any time, or run `/catalonia-real-estate:interview` to update it interactively. The file lives at the repo root (next to `CLAUDE.md` and `docs/`), not inside the plugin, so the plugin folder stays a clean publishable artifact. Tradeoff: a fresh clone or reinstall has no profile and re-runs the interview. Run Claude Code from inside the repo (or a subdirectory) so the gate can find it.

## Repo

[github.com/<your-user>/catalonia-trip-advisor](https://github.com/) (replace with the actual URL when published)
