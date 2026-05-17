---
name: report
description: Use after the advisor produces a substantial answer (itinerary, transport plan, day-trip breakdown) or whenever the user asks to save the answer as a document — phrases like "save this as markdown", "export this", "write me a doc", "give me an HTML file". Asks the user to pick markdown or HTML, then writes a clean, source-cited document to `<repo-root>/exports/`.
---

# Catalonia Trip Advisor — Report

You write the most recent substantial advisor answer to a standalone, source-cited document on disk. The advisor already enforces stable/volatile fact discipline; you do not re-fetch anything. Render only what is already in the conversation.

## Step 1 — Resolve the repo root and prepare `exports/`

Run this Bash command. It walks up from `$PWD` until it finds a directory containing both `CLAUDE.md` and `.claude-plugin/marketplace.json`, then creates `exports/` there.

````bash
DIR="$PWD"
PROJECT_ROOT=""
while [ "$DIR" != "/" ]; do
  if [ -f "$DIR/CLAUDE.md" ] && [ -f "$DIR/.claude-plugin/marketplace.json" ]; then
    PROJECT_ROOT="$DIR"
    break
  fi
  DIR="$(dirname "$DIR")"
done
if [ -z "$PROJECT_ROOT" ]; then
  echo "PROJECT_ROOT_NOT_FOUND"
else
  mkdir -p "$PROJECT_ROOT/exports"
  echo "PROJECT_ROOT:$PROJECT_ROOT"
fi
````

If the output is `PROJECT_ROOT_NOT_FOUND`, tell the user in one line: "I need to be run from inside the catalonia-trip-advisor repo — try `cd` into that folder and ask again." Then stop. Do not improvise an answer or write to any other path.

Otherwise, remember the absolute path after `PROJECT_ROOT:` for later steps.

## Step 2 — Determine the format

First, check the user's most recent message that triggered this skill. If it already names a format, use it and skip the question:

- Mentions of `markdown`, `md`, `.md`, "as markdown", "as md" → markdown.
- Mentions of `html`, `.html`, "as html", "html file" → HTML.

If — and only if — the request is ambiguous (e.g. plain "save this", "export this", "write me a doc"), ask exactly one short question, no preamble: **"Markdown or HTML?"** Wait for the reply, accepting the same tokens as above. If anything else, ask once more.

Do not ask any other follow-ups (no "what should the title be", no "where should I save it").

## Step 3 — Identify the source answer

Look at the conversation history and find the most recent substantial answer the advisor produced. "Substantial" means:

- Multiple paragraphs or a day-by-day structure, AND
- At least one source citation (either `[research/...]` or `[domain.tld, fetched YYYY-MM-DD]`).

If the latest answer is a one-liner with no citations (e.g., "Yes, the tap water is safe."), tell the user in one line: "The previous answer had no sources to cite, so I won't write a document for it." Then stop.

Otherwise, use that answer as your raw material. Do NOT invent content. Do NOT re-fetch any URL. Do NOT re-read any research file.

## Step 4 — Derive the title and slug

Read the topic of the answer. Produce:

- **Title (H1):** a short noun phrase. Examples: "Barcelona — 5-Day Itinerary", "Getting from BCN airport to the city", "Day trip to Montserrat".
- **Slug:** apply these steps in order, exactly once each:
  1. Lowercase the title.
  2. Fold non-ASCII characters to their nearest ASCII equivalent (`è` → `e`, `ç` → `c`, `ñ` → `n`, `—` → `-`, `–` → `-`). For characters with no obvious Latin fold (CJK, Cyrillic, emoji, etc.), drop them.
  3. Replace any whitespace run with a single `-`.
  4. Strip every character that is not `[a-z0-9-]` (this also removes `.`, `/`, `\`, quotes, etc.).
  5. Collapse runs of `-` into a single `-`.
  6. Trim leading and trailing `-`.
  7. Cap length at 60 characters (do not split a word — just truncate).
  8. If the result is empty after all that (e.g. the title was entirely non-foldable characters), fall back to `report-YYYY-MM-DD` using today's date.

  The final slug MUST match `^[a-z0-9-]+$`. If it doesn't, use the fallback. Example: `Barcelona — 5-Day Itinerary` → `barcelona-5-day-itinerary`.

## Step 5 — Restructure into document shape

Build the body in this order:

1. **H1 title** (markdown `# Title` or HTML `<h1>Title</h1>` — handled by the renderer in Step 7).
2. **"At a glance" block** (1–3 lines). Derive it from the advisor's answer itself — the advisor already filtered through the profile, so the answer text contains what you need (trip length, base, focus). Do NOT open the profile file; the answer is the source of truth here. For non-itinerary docs, a 1–2 line lede framing the topic is enough.
3. **Body sections.** Mirror the advisor's structure: day-by-day (`## Day 1 — ...`) for itineraries, topic sections (`## Transport from BCN airport`) otherwise. Reuse the advisor's wording where it reads well as a document; tighten the chattier passages.
4. **Inline source markers.** Every cited fact gets a superscript marker numbered in citation order. In markdown: `[¹]`, `[²]`, `[³]` using Unicode superscripts. In HTML, wrap each marker in an anchor that jumps to the matching Sources entry: `<sup><a href="#src1">1</a></sup>`, `<sup><a href="#src2">2</a></sup>`, and so on. The same source reused later in the document keeps its first number and the same `href`.

## Step 6 — Build the Sources footer

Collect every citation from the source answer. Deduplicate (same URL → one entry; same research path → one entry). Number in citation order — the order in which the source's marker first appears in the body.

Format each entry:

- **Web source (had a URL and `fetched YYYY-MM-DD`):**
  - Markdown: `1. [<label>](<url>) — fetched YYYY-MM-DD`
  - HTML: `<li id="src1"><a href="<url>"><label></a> — fetched YYYY-MM-DD</li>`
  - `<url>` is the URL exactly as the advisor cited it. If — and only if — the citation contains a bare domain rather than a full URL, prefix `https://` to make the link work, and treat that as a degraded case (the advisor citation discipline now requires a full URL).
  - `<label>` is a short human-readable name derived from the source's host or path (e.g. `https://www.tmb.cat/...` → "tmb.cat", `https://www.barcelona-tourist-guide.com/en/airport/aerobus.html` → "Aerobus · barcelona-tourist-guide.com"). Do NOT invent a marketing-style label that isn't supported by the URL. If nothing reasonable comes to mind, fall back to the URL's host (`tmb.cat`).
- **Internal source (a `research/...` path):**
  - Markdown: `1. [Catalonia Advisor knowledge base — <path>](../catalonia-trip-advisor/skills/advisor/research/<path>.md)`
  - HTML: `<li id="src2"><a href="../catalonia-trip-advisor/skills/advisor/research/<path>.md">Catalonia Advisor knowledge base — <path></a></li>`
  - `<path>` is the citation with the leading `research/` and the trailing `.md` stripped (e.g., `research/transportation/metro.md` → `transportation/metro`). The `href` is a relative path from `exports/` back to the actual research file in the plugin — so a reader can click straight through to the underlying corpus. It resolves in any browser opened against the file, and renders the markdown if the repo is viewed on GitHub. Both inline `<sup>` and the footer entry are clickable.

The `id="srcN"` numbers must match the inline superscript numbers built in Step 5, so a click on superscript `2` in the body scrolls to `<li id="src2">` in the footer.

## Step 7 — Render in the chosen format

### If markdown

Build the full document as a single string following the layout in `assets/style-notes.md`:

````
# {Title}

*Generated {YYYY-MM-DD} · catalonia-trip-advisor*

{at-a-glance block}

## {Section 1}

{body with [¹], [²] markers}

## {Section 2}

…

## Sources

1. [TMB official fares](https://www.tmb.cat/...) — fetched 2026-05-17
2. Catalonia Advisor knowledge base — transportation/metro
````

### If HTML

Read the template at `${CLAUDE_SKILL_DIR}/assets/template.html` (or the equivalent relative path `assets/template.html`). Substitute the four placeholders:

- `{{TITLE}}` → the plain-text H1 title (no HTML).
- `{{DATE}}` → today's date in `YYYY-MM-DD`.
- `{{BODY}}` → the body sections converted to HTML: `## X` → `<h2>X</h2>`, paragraphs wrapped in `<p>`, lists as `<ul><li>`, superscript markers as `<sup>N</sup>`.
- `{{SOURCES}}` → an `<ol>` of `<li>` entries built in Step 6.

**Markdown elements not covered above** — convert as expected: `**bold**` → `<strong>`, `*italic*` → `<em>`, `` `code` `` → `<code>`, `[label](url)` → `<a href="url">label</a>`, `### Heading` → `<h3>`, pipe tables → `<table>` with `<thead>`/`<tbody>` rows, blockquotes → `<blockquote>`, fenced code → `<pre><code>`. Preserve the literal `[¹]`, `[²]` superscript markers as `<sup>1</sup>`, `<sup>2</sup>` — do NOT let the `[ ]` be parsed as link brackets. HTML-escape `<`, `>`, `&` in the title and any body text that isn't itself markup.

## Step 8 — Write the file

Filename: `YYYY-MM-DD-<slug>.<ext>` where `<ext>` is `md` or `html` and the date is today.

Full path: `<PROJECT_ROOT>/exports/<filename>`.

If a file with that exact name already exists, append `-2`, `-3`, etc. before the extension (e.g., `2026-05-17-barcelona-5-day-itinerary-2.md`) until you find a free name. Use Bash `test -e` to check.

Use the Write tool to create the file.

## Step 9 — Report the path

Reply with exactly one line:

````
Saved to exports/<filename> — open it from your editor or browser.
````

Nothing else. No summary of what's in the file, no offer of follow-ups. The file speaks for itself.

## What you must NOT do

- Do not re-fetch any URL — even if a price feels stale, the contract is that the doc matches what the advisor just showed.
- Do not re-read research files for fresh material — work only from the latest advisor answer.
- Do not invent citations the advisor did not make. If the advisor cited two URLs and one research file, the Sources footer has three entries — no more, no fewer.
- Do not write to any path outside `<PROJECT_ROOT>/exports/`. The slug pipeline in Step 4 strips `.`, `/`, `\`, and every other non-`[a-z0-9-]` character — if you ever find yourself building a filename that contains those, you've skipped the slug pipeline; go back and apply it.
- Do not ask follow-up questions after Step 2.
- Do not export an answer that has zero citations — that's the gate in Step 3.
