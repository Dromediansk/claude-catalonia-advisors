# Retrieving Idealista (and Fotocasa/Habitaclia) listing data

Research notes for the `catalonia-real-estate` plugin. Captured 2026-06-12.

## The problem

The advisor's `report`/listing search relies on `WebSearch` + `WebFetch`. Plain `WebFetch`
against Idealista listing pages **fails** — Idealista sits behind **DataDome** anti-bot
protection and serves a challenge page to headless clients instead of the listings.
Fotocasa and Habitaclia use similar protection to varying degrees.

So the fix is not a "better fetch" — it's a different data channel. There are three.

The current skill behavior (fall back to "browse the portal directly" + a portal search
URL, and never invent listings) is the **correct** response when no real data channel is
wired up. Everything below is about adding a real channel.

## Channel 1 — Official Idealista API

- **Cost:** free.
- **Access:** gated. Apply at <https://developers.idealista.com/access-request>, describe
  the project, wait for manual approval. Favors real businesses over personal tools.
- **Shape:** OAuth2 (client id + secret → bearer token), returns clean JSON for search.
- **Limit:** historically a very small monthly quota (~100 requests/month); the exact
  current free quota is not published — you learn it on approval.
- **ToS:** this is the sanctioned, lowest-risk path.
- **Community clients / walkthroughs:**
  - <https://github.com/yagueto/idealista-api> (Python client)
  - <https://medium.com/@guilhermedatt/calling-the-idealista-api-using-python-a39a843cf5cc>

**Verdict:** cleanest on terms-of-service and free, but slow to set up and tightly
rate-limited. Best if you get accepted and only need light, occasional lookups.

## Channel 2 — Third-party scraper APIs

These bypass DataDome for you and return JSON; no Idealista approval needed. Most are paid
per result, but several have free trials or recurring free allowances.

| Service | Free offer | Link |
|---|---|---|
| Apify Idealista scraper | $5/month credits (see Channel 3) | <https://apify.com/lukass/idealista-scraper/api> |
| ZenRows | trial credits | <https://docs.zenrows.com/scraper-apis/get-started/idealista-property> |
| Piloterr | 50 free requests | <https://www.piloterr.com/library/idealista-search> |
| Oxylabs | up to 2,000 results, no card | <https://oxylabs.io/products/scraper-api/real-estate/idealista> |
| ScrapingBee | free signup credits | <https://www.scrapingbee.com/scrapers/idealista-api/> |
| APIDojo (RapidAPI) | freemium tier | <https://rapidapi.com/apidojo/api/idealista2> |

**ToS note:** third-party scrapers operate in a greyer area than the official API. Fine for
personal/research use; reconsider for anything commercial.

## Channel 3 — MCP servers (best fit for Claude Code)

Apify hosts ready-to-use **MCP servers** wrapping its Idealista/Fotocasa scrapers. They
handle DataDome and return structured JSON (price, location, rooms, agent, energy rating,
28+ fields). This is the channel that drops most cleanly into this plugin — the advisor
skill would call an MCP tool instead of `WebFetch`.

- Idealista.com — <https://apify.com/lukass/idealista-scraper/api/mcp>
- Idealista Property Search & Discovery — <https://apify.com/axlymxp/idealista-property-search/api/mcp>
- Spain/Portugal multi-portal (Idealista + Fotocasa + Milanuncios) — <https://apify.com/friendly_keelboat/spain-portugal-real-estate/api/mcp>
- Fotocasa — <https://apify.com/igolaizola/fotocasa-scraper/api/mcp>

Connect via `mcp.apify.com` with an Apify account + API token (Apify Console →
Integrations).

## Free options, ranked

The question that matters most: **what is free and stays free?**

1. **Apify free plan + its Idealista MCP — most practical.**
   $0/month, **no credit card**, **$5 in platform credits every month**
   (<https://apify.com/pricing>, <https://use-apify.com/docs/what-is-apify/apify-free-plan>).
   The Idealista scrapers are pay-per-result, so $5 buys a limited-but-real number of
   listings monthly (an actor at ~$4/1,000 results → ~1,250 listings before the budget
   stops). Credits don't roll over; runs halt when the $5 is spent. For casual personal
   use ("show me 5 flats in Girona") you'd essentially never hit the cap.

2. **Official Idealista API — cleanest on ToS, if approved.**
   Free but gated, tiny quota. See Channel 1.

3. **One-time free trials — good for testing, not sustainable.**
   Oxylabs (2,000 results), Piloterr (50 requests), ScrapingBee (signup credits).
   These expire; not a standing solution.

4. **Self-hosted scraper — $0 in fees, high effort.**
   Playwright/stealth + your own proxies. No service fee, but you must defeat DataDome
   yourself: fragile, breaks often, grey-area ToS. Not worth it for this plugin.

**Recommendation:** if you want something free *and* easy to wire into the plugin, use the
**Apify free plan + its Idealista MCP**. If terms-of-service cleanliness matters more than
setup speed, pursue the **official API** and live with the quota.

## Validation (2026-06-12) — channel confirmed

We wired the Apify free plan + MCP and tested two actors against the plugin's real
requirements (sale not rent, anywhere in Catalonia, a clickable per-listing URL).

**Winner — [`lukass/idealista-scraper`](https://apify.com/lukass/idealista-scraper/api/mcp).**
A `sale` query for `district: "Girona"` returned 5 real Idealista listings in ~12s, each with
a direct `https://www.idealista.com/inmueble/<id>/` URL plus price, rooms, baths, size,
address, typology, photos, and agent contact. This is the channel to wire. Notes:

- `operation` supports `sale` (default) — not rental-only.
- `district` is **free-text** (e.g. "Girona", "Tarragona"), so it covers all of Catalonia, not a fixed city list. `startUrl` also accepts a full Idealista search URL.
- `proxy` is **required** (`{"useApifyProxy": true, "apifyProxyGroups": ["RESIDENTIAL"]}` works).
- **No `maxPrice`/`minPrice` parameter, and price-filtered `startUrl`s don't work.** We tested it: a plain-location `startUrl` (`/venta-viviendas/girona-girona/`) and the `district` param both crawl fine, but the same URL with a `con-precio-hasta_300000` segment failed twice (`0 succeeded, 1 failed`, 0 items) — even though that's valid Idealista syntax. This actor can't crawl custom filtered search URLs (see its [Custom search URL issue](https://apify.com/lukass/idealista-scraper/issues/custom-search-url-fXl80Sqf2JVdYuEFk)). **Scope the area via `district` and enforce the budget ceiling by post-filtering returned items** (fetch a generous `maxItems` since results aren't price-sorted).
- The `bedrooms`/`bathrooms`/`condition`/`features` filters expect **arrays of objects, not strings** — non-obvious, and we didn't reverse-engineer the object shape. Post-filtering returned items on these fields is the reliable path.
- Idealista-only. Fotocasa/Habitaclia stay on the `WebSearch`/portal-URL fallback.
- Cost across the whole validation session: ~0.025 compute units — negligible against the $5/mo.

**Rejected — [`friendly_keelboat/spain-portugal-real-estate`](https://apify.com/friendly_keelboat/spain-portugal-real-estate/api/mcp).**
Despite the multi-portal billing, its input schema scrapes **rental** listings only (prices
came back as monthly rents, URLs were `/alquiler/`), its `city` enum is limited to five
Spanish cities (Barcelona the only Catalan one — no Girona/Tarragona/Lleida), and its
Idealista source returned **0** items (Fotocasa carried the run). Wrong product type and
wrong geography for a buying advisor. Do not use it for this plugin.

**Registration used (local scope, token kept out of the repo):**

```bash
claude mcp add --transport http --scope local apify-idealista \
  "https://mcp.apify.com/?actors=lukass/idealista-scraper" \
  --header "Authorization: Bearer ${APIFY_TOKEN}"
```

A newly pinned actor only loads on a fresh Claude Code session start — restart after adding.

## If you decide to wire one up

The advisor skill currently lists Idealista/Fotocasa/Habitaclia as `WebSearch`/`WebFetch`
portals (`catalonia-real-estate/skills/advisor/SKILL.md`, "Portals & authoritative sources"
and "Live listing search"). Wiring a channel means:

1. Register the MCP server (or API) so the tool is available in the session.
2. Update the advisor's "Live listing search" procedure to call that tool instead of
   `WebFetch` for Idealista, keeping `WebSearch`/`WebFetch` as the fallback.
3. Keep every existing guardrail intact: profile-scoped queries, the mandatory clickable
   direct URL per listing, the time-sensitivity caveat, and the no-invented-listings rule.

## Sources

- <https://developers.idealista.com/access-request>
- <https://apify.com/pricing>
- <https://use-apify.com/docs/what-is-apify/apify-free-plan>
- <https://apify.com/lukass/idealista-scraper/api>
- <https://apify.com/lukass/idealista-scraper/api/mcp>
- <https://oxylabs.io/products/scraper-api/real-estate/idealista>
- <https://www.piloterr.com/library/idealista-search>
- <https://docs.zenrows.com/scraper-apis/get-started/idealista-property>
- <https://rapidapi.com/apidojo/api/idealista2>
