---
title: Markdown export layout cheat sheet
topic: meta
last_verified: 2026-05-17
---

# Markdown export layout

Use this as the exact layout for `.md` exports produced by the `report` skill.

```
# {Title}

*Generated {YYYY-MM-DD} · catalonia-trip-advisor*

{1–3 line "at a glance" block — plain prose or italics, no heading}

## {First section heading — e.g., "Day 1 — El Born & Gothic Quarter"}

{body, with superscript markers like [¹] next to each cited claim}

## {Second section heading}

…

## Sources

1. [tmb.cat](https://www.tmb.cat/en/barcelona-fares-metro-bus) — fetched 2026-05-17
2. [Catalonia Advisor knowledge base — transportation/metro](../catalonia-trip-advisor/skills/advisor/research/transportation/metro.md)
3. …
```

Rules:

- Sources are numbered in citation order and deduplicated.
- Web URLs become markdown links `[label](url) — fetched YYYY-MM-DD`. `url` is the full URL the advisor cited (the report skill's Step 6 covers the legacy domain-only fallback). `label` is a short human-readable name derived from the URL's host or path (e.g. "tmb.cat", "Aerobus · barcelona-tourist-guide.com"); do not invent a marketing label.
- Internal sources (research files) become markdown links back into the plugin: `[Catalonia Advisor knowledge base — <path>](../catalonia-trip-advisor/skills/advisor/research/<path>.md)`. The `<path>` strips the leading `research/` and the trailing `.md` (so `research/transportation/metro.md` becomes `transportation/metro`). The relative href resolves both in a local editor and when the file is viewed on GitHub.
- No trailing blank line after the sources list.
- Inline superscript markers use the Unicode characters `¹ ² ³ ⁴ ⁵ ⁶ ⁷ ⁸ ⁹ ⁰` in square brackets, e.g. `[¹]`. For numbers above 9, combine: `[¹⁰]`.
