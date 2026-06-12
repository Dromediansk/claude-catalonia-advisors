---
title: Markdown export layout cheat sheet
topic: meta
last_verified: 2026-06-12
---

# Markdown export layout

Use this as the exact layout for `.md` exports produced by the `report` skill.

```
# {Title}

*Generated {YYYY-MM-DD} · catalonia-real-estate*

{1–3 line "at a glance" block — plain prose or italics, no heading}

## {First section heading — e.g., "Eixample — price benchmarks"}

{body, with superscript markers like [¹] next to each cited claim}

## {Second section heading}

…

## Sources

1. [idealista.com](https://www.idealista.com/en/inmueble/12345678/) — fetched 2026-06-12
2. [atc.gencat.cat](https://atc.gencat.cat/en/) — fetched 2026-06-12
3. …
```

Rules:

- Sources are numbered in citation order and deduplicated.
- Web URLs become markdown links `[label](url) — fetched YYYY-MM-DD`. `url` is the full URL the advisor cited. `label` is a short human-readable name derived from the URL's host or path (e.g. "idealista.com", "atc.gencat.cat"); do not invent a marketing label.
- Listing links keep their full deep URL so the reader can open the exact property (which may already be gone — that is expected for time-sensitive listings).
- No trailing blank line after the sources list.
- Inline superscript markers use the Unicode characters `¹ ² ³ ⁴ ⁵ ⁶ ⁷ ⁸ ⁹ ⁰` in square brackets, e.g. `[¹]`. For numbers above 9, combine: `[¹⁰]`.
