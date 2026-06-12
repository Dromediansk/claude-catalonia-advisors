# catalonia-trip-advisor

A Claude Code plugin that turns Claude into a travel advisor for tourists visiting **Catalonia**, with a focus on **Barcelona**. Itineraries, day trips (Montserrat, Girona, Costa Brava, Sitges, Tarragona), transport (metro, T-Casual, Hola Barcelona, Rodalies, RENFE, airport transfers), neighborhoods, food, scams, tipping, and Catalan culture.

Out-of-region questions (Madrid, Andalusia, etc.) are politely declined — depth on Catalonia is the point.

## Install

### Quick try (ephemeral)

From the repo root, launch one session with the plugin loaded:

```bash
claude --plugin-dir ./catalonia-trip-advisor
```

Good for a smoke test; the plugin is not loaded by plain `claude`.

### Persistent local install (development)

Register the repo as a local marketplace and enable the plugin globally. After this, plain `claude` loads it in every session.

1. The repo root already ships a marketplace manifest at `.claude-plugin/marketplace.json` that declares this plugin.
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
       "catalonia-trip-advisor@catalonia-local": true
     }
   }
   ```

   Point `path` at the **repo root** (the folder containing `.claude-plugin/marketplace.json`), not the inner plugin folder.

3. Restart Claude Code, or run `/reload-plugins` in an existing session.

After edits to skill or research files, `/reload-plugins` picks up changes without restarting — the marketplace points at your working tree.

### Marketplace

Once published to a marketplace, install via `/plugin install catalonia-trip-advisor` (or the equivalent your marketplace surfaces).

## Skills

| Skill ID                                 | Purpose                                                                                                                   |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `/catalonia-trip-advisor:advisor`        | The main advisor. Triggers on Catalonia/Barcelona questions. Starts every turn with a profile gate.                       |
| `/catalonia-trip-advisor:interview`      | One-time preferences interview. Runs automatically on first use; rerun explicitly with "update my Catalonia preferences". |
| `/catalonia-trip-advisor:report`         | Saves the latest substantial advisor answer to `<repo-root>/exports/` as markdown or HTML, with a numbered Sources footer. Triggers on "save this as markdown", "export this", "give me an HTML file". |

You usually don't invoke them by name — just ask a Catalonia question ("Barcelona for 5 days", "is the T-Casual worth it", "day trip to Montserrat") and the advisor fires.

## How it works

1. **Profile gate.** Every advisor turn first locates the repo root (the directory containing both `CLAUDE.md` and `.claude-plugin/marketplace.json`) and checks `profile/spain-trip-profile.md` there. If it's missing, the interview skill runs once (six short questions, one at a time), saves your preferences (group, pace, interests, budget, diet, mobility, language, base), and hands back to the advisor.
2. **Stable vs. volatile facts.**
   - **Stable** (neighborhood character, dishes, scam patterns, line maps) → cited from `skills/advisor/research/`.
   - **Volatile** (prices, fares, hours, schedules, festival dates) → **WebFetched live** from URLs in `research/sources.md` and cited with the fetch date. Never quoted from a cached file.
3. **Profile-shaped answers.** Budget filters venue tier, interests reorder itinerary anchors, diet filters food, mobility constrains routes, pace caps activities/day — applied silently, never echoed back at you.

## Profile location

`<repo-root>/profile/spain-trip-profile.md` — plain markdown, edit by hand any time, or run `/catalonia-trip-advisor:interview` to update it interactively. The file lives at the repo root (next to `CLAUDE.md`), not inside the plugin, so the plugin folder stays a clean publishable artifact. Tradeoff: a fresh clone or reinstall has no profile and re-runs the interview. Run Claude Code from inside the repo (or a subdirectory) so the gate can find it.

## Repo

[github.com/Dromediansk/trip-advisor-claude-marketplace](https://github.com/Dromediansk/trip-advisor-claude-marketplace)
