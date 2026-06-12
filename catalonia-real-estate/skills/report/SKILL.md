---
name: report
description: Use after the real-estate advisor produces a substantial answer (area comparison, buying-process walkthrough, financing/tax breakdown, or a curated listing shortlist) or whenever the user asks to save the answer as a document — phrases like "save this as markdown", "export this", "write me a doc", "give me an HTML file". Asks the user to pick markdown or HTML, then writes a clean, source-cited document to `<repo-root>/exports/`.
---

# Catalonia Real Estate Advisor — Report

You write the most recent substantial advisor answer to a standalone, source-cited document on disk. The advisor already enforced fact discipline and added disclaimers; you do not re-fetch anything and you do not re-search listings. Render only what is already in the conversation.

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

Do not ask any other follow-ups.

## Step 3 — Identify the source answer

Look at the conversation history and find the most recent substantial answer the advisor produced. "Substantial" means:

- Multiple paragraphs, a section structure, or a listing shortlist, AND
- At least one source citation (a `[<url>, fetched YYYY-MM-DD]` web/listing citation).

If the latest answer is a one-liner with no citations (e.g., "Yes, you need an NIE to buy."), tell the user in one line: "The previous answer had no sources to cite, so I won't write a document for it." Then stop.

Otherwise, use that answer as your raw material. Do NOT invent content. Do NOT re-fetch any URL. Do NOT re-run any listing search.

## Step 4 — Preserve the advisor's disclaimer

The advisor's answer carries a not-legal/financial/tax disclaimer and (for listings) a time-sensitivity caveat. **Carry these into the document** — render the disclaimer as a short italic line near the top (under the "at a glance" block) and keep any listing time-sensitivity note with the listings. Do not drop them; they are mandatory for this plugin's subject matter.

## Step 5 — Derive the title and slug

Read the topic of the answer. Produce:

- **Title (H1):** a short noun phrase. Examples: "Buying in Gràcia — price benchmarks", "Non-resident mortgage & tax overview", "2-bed flat shortlist — Girona under €350k".
- **Slug:** apply these steps in order, exactly once each:
  1. Lowercase the title.
  2. Fold non-ASCII characters to their nearest ASCII equivalent (`è` → `e`, `ç` → `c`, `ñ` → `n`, `—` → `-`, `–` → `-`). For characters with no obvious Latin fold (CJK, Cyrillic, emoji, etc.), drop them.
  3. Replace any whitespace run with a single `-`.
  4. Strip every character that is not `[a-z0-9-]` (this also removes `.`, `/`, `\`, quotes, `€`, etc.).
  5. Collapse runs of `-` into a single `-`.
  6. Trim leading and trailing `-`.
  7. Cap length at 60 characters. If the slug is longer than 60, truncate to 60 and then back up to the previous `-` so the slug ends on a word boundary; if there is no `-` in the first 60 characters, hard-truncate at 60. After this step, re-apply the trim from rule 6 to drop any trailing `-`.
  8. If the result is empty after all that, fall back to `report-YYYY-MM-DD` using today's date.

  The final slug MUST match `^[a-z0-9-]+$`. If it doesn't, use the fallback. Example: `Buying in Gràcia — price benchmarks` → `buying-in-gracia-price-benchmarks`.

## Step 6 — Restructure into document shape

Build the body in this order:

1. **H1 title** (handled by the renderer in Step 8).
2. **"At a glance" block** (1–3 lines). Derive it from the advisor's answer itself (the answer already reflects the buyer profile). Do NOT open the profile file; the answer is the source of truth. For non-shortlist docs, a 1–2 line lede framing the topic is enough.
3. **Disclaimer line** (italic, one line) carried from Step 4.
4. **Body sections.** Mirror the advisor's structure: area/topic sections (`## Eixample — price benchmarks`), or a listing shortlist where each listing is a sub-block with price, location, features, link, and the profile-fit note. **Every listing sub-block MUST include a clickable URL to that listing's page rendered inline in the body** — a markdown link the user can click (e.g. `[View listing](https://www.idealista.com/en/inmueble/12345678/)`), not only a number in the Sources footer. This is non-negotiable: the whole point of the document is that the user can navigate to each property. If the advisor's answer showed a listing without a usable URL, do not fabricate one — but flag it ("link not available") rather than presenting the property as navigable. The Sources footer (Step 7) is in addition to, not a replacement for, the inline link. Reuse the advisor's wording where it reads well; tighten chattier passages.
5. **Inline source markers.** Every cited fact or listing gets a superscript marker numbered in citation order. In markdown: `[¹]`, `[²]`, `[³]` using Unicode superscripts. In HTML, wrap each marker in an anchor that jumps to the matching Sources entry: `<sup><a href="#src1">1</a></sup>`. The same source reused later keeps its first number and the same `href`.

## Step 7 — Build the Sources footer

Collect every citation from the source answer. Deduplicate (same URL → one entry). Number in citation order — the order in which the source's marker first appears in the body.

Format each entry (all citations in this plugin are web/listing URLs):

- Markdown: `1. [<label>](<url>) — fetched YYYY-MM-DD`
- HTML: `<li id="src1"><a href="<url>"><label></a> — fetched YYYY-MM-DD</li>`
- `<url>` is the URL exactly as the advisor cited it. If — and only if — the citation contains a bare domain rather than a full URL, prefix `https://` to make the link work, and treat that as a degraded case (the advisor citation discipline requires a full URL).
- `<label>` is a short human-readable name derived from the source's host or path (e.g. `https://www.idealista.com/en/inmueble/12345678/` → "idealista.com", `https://atc.gencat.cat/en/` → "atc.gencat.cat"). Do NOT invent a marketing-style label. If nothing reasonable comes to mind, fall back to the URL's host.

The `id="srcN"` numbers must match the inline superscript numbers built in Step 6, so a click on superscript `2` in the body scrolls to `<li id="src2">` in the footer.

## Step 8 — Render in the chosen format

### If markdown

Build the full document as a single string following the layout in `assets/style-notes.md`:

````
# {Title}

*Generated {YYYY-MM-DD} · catalonia-real-estate*

{at-a-glance block}

*{disclaimer line}*

## {Section 1}

{body with [¹], [²] markers}

## {Section 2}

…

## Sources

1. [idealista.com](https://www.idealista.com/en/inmueble/12345678/) — fetched 2026-06-12
2. [atc.gencat.cat](https://atc.gencat.cat/en/) — fetched 2026-06-12
````

### If HTML

Read the template at `${CLAUDE_SKILL_DIR}/assets/template.html` (or the equivalent relative path `assets/template.html`). Substitute the four placeholders:

- `{{TITLE}}` → the plain-text H1 title (no HTML).
- `{{DATE}}` → today's date in `YYYY-MM-DD`.
- `{{BODY}}` → the body sections converted to HTML: `## X` → `<h2>X</h2>`, the disclaimer as `<p><em>…</em></p>`, paragraphs wrapped in `<p>`, lists as `<ul><li>`, superscript markers as anchored links `<sup><a href="#srcN">N</a></sup>`.
- `{{SOURCES}}` → an `<ol>` of `<li>` entries built in Step 7.

**Markdown elements not covered above** — convert as expected: `**bold**` → `<strong>`, `*italic*` → `<em>`, `` `code` `` → `<code>`, `[label](url)` → `<a href="url">label</a>`, `### Heading` → `<h3>`, pipe tables → `<table>` with `<thead>`/`<tbody>` rows, blockquotes → `<blockquote>`, fenced code → `<pre><code>`. Preserve the literal `[¹]`, `[²]` superscript markers as anchored links — do NOT let the `[ ]` be parsed as link brackets, and do NOT emit a bare `<sup>1</sup>` without the anchor. HTML-escape `<`, `>`, `&` in the title and any body text that isn't itself markup.

## Step 9 — Write the file

Filename: `YYYY-MM-DD-<slug>.<ext>` where `<ext>` is `md` or `html` and the date is today.

Full path: `<PROJECT_ROOT>/exports/<filename>`.

If a file with that exact name already exists, append `-2`, `-3`, etc. before the extension until you find a free name. Use Bash `test -e` to check.

Use the Write tool to create the file.

## Step 10 — Report the path

Reply with exactly one line:

````
Saved to exports/<filename> — open it from your editor or browser.
````

Nothing else.

## What you must NOT do

- Do not re-fetch any URL or re-run any listing search — the doc matches what the advisor just showed.
- Do not invent citations the advisor did not make. If the advisor cited three listings and one tax page, the Sources footer has four entries — no more, no fewer.
- Do not drop the not-legal/financial/tax disclaimer or the listing time-sensitivity caveat.
- Do not render a listing without a clickable inline URL to its listing page. Every property in the document must be navigable; if the advisor had no link for it, flag "link not available" rather than presenting it as clickable.
- Do not write to any path outside `<PROJECT_ROOT>/exports/`. The slug pipeline in Step 5 strips `.`, `/`, `\`, `€`, and every other non-`[a-z0-9-]` character — if you're building a filename with those, you skipped the slug pipeline; go back and apply it.
- Do not ask follow-up questions after Step 2.
- Do not export an answer that has zero citations — that's the gate in Step 3.
