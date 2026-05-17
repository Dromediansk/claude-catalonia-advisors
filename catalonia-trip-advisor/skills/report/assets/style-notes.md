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

1. [TMB official fares](https://www.tmb.cat/...) — fetched 2026-05-17
2. Catalonia Advisor knowledge base — transportation/metro
3. …
```

Rules:

- Sources are numbered in citation order and deduplicated.
- Web URLs become markdown links `[label](url) — fetched YYYY-MM-DD`. The `label` is a short human-readable name (e.g., "TMB official fares"), not the bare domain.
- Internal sources (research files) appear as plain text `Catalonia Advisor knowledge base — <path>`. The `<path>` strips the leading `research/` and the trailing `.md` (so `research/transportation/metro.md` becomes `transportation/metro`). No hyperlink — there's no external URL to link to.
- No trailing blank line after the sources list.
- Inline superscript markers use the Unicode characters `¹ ² ³ ⁴ ⁵ ⁶ ⁷ ⁸ ⁹ ⁰` in square brackets, e.g. `[¹]`. For numbers above 9, combine: `[¹⁰]`.