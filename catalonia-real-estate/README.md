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
       "catalonia-advisors": {
         "source": {
           "source": "directory",
           "path": "/absolute/path/to/catalonia-trip-advisor"
         }
       }
     },
     "enabledPlugins": {
       "catalonia-real-estate@catalonia-advisors": true
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
3. **Live listing search.** The advisor builds profile-scoped queries (area, budget, type), searches Idealista/Fotocasa/Habitaclia, and summarizes a handful of representative listings with price, location, link, and a profile-fit note — flagging that listings are time-sensitive and links may expire. For richer, structured Idealista results you can optionally wire a free MCP server — see [Live Idealista listings](#live-idealista-listings-optional) below.
4. **Profile-shaped answers.** Intent, budget, financing status, and location filter the advice and scope the search — applied silently, never echoed back at you.

**Disclaimers:** every answer touching process, money, tax, or legal status carries a not-legal/financial/tax disclaimer; listing answers carry a time-sensitivity caveat. These are built into the skills, not optional.

## Live Idealista listings (optional)

Out of the box, the advisor finds listings with `WebSearch` and links you to portal search pages. That works — but **Idealista**, the largest Catalan portal, sits behind anti-bot protection that blocks direct fetching, so the advisor can't always pull structured per-listing detail from it.

To get real, structured Idealista **sale** listings (price, rooms, size, and a clickable link per property), you can wire the free [Apify](https://apify.com/) MCP server. This is **optional** — the plugin works without it — and affects only Idealista; Fotocasa and Habitaclia always use the search fallback.

1. Create a free Apify account at [console.apify.com/sign-up](https://console.apify.com/sign-up) (no credit card).
2. Copy your API token from **Settings → Integrations → API tokens**.
3. Store the token in an environment variable instead of pasting it into any Claude config. Add this to your shell profile (`~/.zshrc`, `~/.bashrc`, etc.) so it persists across sessions:

   ```bash
   export APIFY_TOKEN="apify_api_xxxxxxxxxxxxxxxxxxxx"
   ```

   Then reload your shell (`source ~/.zshrc`) or open a new terminal so the variable is set.
4. From the repo root, register the Idealista scraper as a local MCP server. Reference the variable in the header — **use single quotes** so your shell does **not** expand `${APIFY_TOKEN}` before Claude stores it:

   ```bash
   claude mcp add --transport http --scope local apify-idealista \
     'https://mcp.apify.com/?actors=lukass/idealista-scraper' \
     --header 'Authorization: Bearer ${APIFY_TOKEN}'
   ```

   This writes only the literal string `${APIFY_TOKEN}` into your config — **not the token itself**. Claude Code expands it from the environment each time the server loads, so your secret never lands in `claude.json` (or any committed file). `--scope local` additionally keeps the entry in your personal project config rather than the shared repo.
5. **Restart Claude Code** (a newly added MCP server only loads on a fresh session, and it must inherit the `APIFY_TOKEN` variable from your environment), then confirm with `claude mcp list` that `apify-idealista` shows `✔ Connected`.

That's it — the advisor automatically prefers the MCP tool for Idealista when it's present and silently falls back to `WebSearch` when it isn't. The Apify free plan includes **$5/month of credits**, which covers casual use comfortably (a typical search costs a fraction of a cent).

Why this setup: plain `WebFetch` is blocked by Idealista's anti-bot protection (DataDome), so the scraper actor is the only reliable way to pull structured per-listing detail. Known limitation: the actor doesn't filter by price server-side, so the advisor narrows results client-side after fetching.

## Profile location

`<repo-root>/profile/catalonia-realestate-profile.md` — plain markdown, edit by hand any time, or run `/catalonia-real-estate:interview` to update it interactively. The file lives at the repo root (next to `CLAUDE.md`), not inside the plugin, so the plugin folder stays a clean publishable artifact. Tradeoff: a fresh clone or reinstall has no profile and re-runs the interview. Run Claude Code from inside the repo (or a subdirectory) so the gate can find it.

## Repo

[github.com/Dromediansk/claude-catalonia-advisors](https://github.com/Dromediansk/claude-catalonia-advisors)
