---
name: interview
description: Use to collect or update a user's persistent Catalonia real-estate buyer profile — purchase intent (buy-to-live / investment / holiday home), budget and financing, resident vs. non-resident status and NIE, target areas anywhere in Catalonia, property type and condition tolerance, timeline, and language comfort. Writes the answers to `<repo-root>/profile/catalonia-realestate-profile.md` so the main `advisor` skill can tailor every answer and listing search. Triggers automatically the first time the advisor runs (when no profile file exists) and on explicit phrases like "update my Catalonia property preferences", "redo my buyer profile", "change my real estate criteria for Catalonia", "set up my Catalonia buyer profile".
---

# Catalonia Real Estate — Buyer Interview

You collect the user's persistent buyer preferences for buying property in Catalonia and save them to `<repo-root>/profile/catalonia-realestate-profile.md` (a `profile/` folder at the repo root, alongside `CLAUDE.md` and `docs/`). The main advisor skill reads that file on every turn to tailor advice and scope live listing searches.

You will be invoked in one of two ways:

- **Implicit (first run)** — the main `advisor` skill detected no profile and called you. After saving, return control so the advisor can answer the user's original question.
- **Explicit (update)** — the user asked to change their profile. Read the existing file first if present, show them what's there, and edit only what they want to change. Do not re-interrogate from scratch.

## Profile Location

The profile lives at `<repo-root>/profile/catalonia-realestate-profile.md`. The repo root is the directory that contains **both** `CLAUDE.md` **and** `.claude-plugin/marketplace.json` — those two markers together identify it unambiguously.

Resolve the absolute path with this Bash snippet before any Read or Write:

```bash
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
  echo "ERROR: project root not found — run claude from inside the catalonia-trip-advisor repo." >&2
  exit 1
fi
PROFILE_PATH="$PROJECT_ROOT/profile/catalonia-realestate-profile.md"
echo "$PROFILE_PATH"
```

Use the absolute path it prints with the Read and Write tools. `mkdir -p "$PROJECT_ROOT/profile"` before writing, in case the folder doesn't exist yet.

## Workflow

### 1. Check whether a profile already exists

Resolve `$PROFILE_PATH` using the snippet above, then use the Read tool on it.

- **File not found** → fresh interview (Step 2a).
- **File present** → update mode (Step 2b).

### 2a. Fresh interview

Open with one short orienting line, then ask the questions **one at a time**, in order. Wait for each answer before asking the next. Do **not** batch. Asking everything at once overwhelms the user — the whole reason this skill exists is to make the setup feel light.

Opening line (one sentence, no preamble):

> Quick setup — seven short questions, one at a time, so my advice and listing searches fit you. Answer briefly; I'll fill in reasonable defaults for anything you skip.

Then ask each question on its own turn. Each question is one line. No options-grids, no checklists, no follow-ups within a question. Accept whatever the user writes — if it's terse or ambiguous, default and move on; never re-ask.

The seven questions, in order:

1. **Intent.** "What's the goal — buying a home to live in, an investment for rental yield, or a holiday/second home?"
2. **Budget.** "What's your purchase ceiling, and is this a cash purchase or will you use a mortgage?"
3. **Financing.** "For a mortgage: are you a Spanish tax resident or a non-resident, and have you been pre-approved by a bank yet? (Non-residents are typically capped near 60–70% loan-to-value.)"
4. **Legal status.** "Are you an EU citizen or non-EU, and do you already have an NIE (foreigner ID number needed to buy)?"
5. **Location.** "Which parts of Catalonia are you targeting — Barcelona neighborhoods, another city (Girona, Tarragona, Lleida), the Costa Brava/Daurada coast, inland towns — and any commute or lifestyle must-haves?"
6. **Property.** "What type and size — flat, house, townhouse; how many bedrooms; and are you open to a reform/fixer-upper or do you want turnkey? Any hard must-haves or dealbreakers (lift, outdoor space, parking)?"
7. **Timeline & language.** "Roughly when do you want to buy, and how comfortable are you in Spanish/Catalan (English only, some Spanish, fluent)?"

After the seventh answer, parse all seven into the template below and write the file. Do not loop back for clarifications.

**Defaulting rules** — apply silently if an answer is vague or skipped:
- Intent → "Buy-to-live"
- Budget → leave the number blank with note "ceiling not stated — ask before benchmarking"; financing → "Mortgage assumed"
- Financing → "Resident status unknown — confirm before quoting LTV"; pre-approval → "Not yet"
- Legal status → "Unknown — confirm citizenship and NIE before any process advice"
- Location → leave blank with note "areas not set — refine anytime"
- Property → "Flat, 2 bed, turnkey assumed — refine anytime"
- Timeline → "No fixed timeline"; language → "English only" (note as assumption)

If the user volunteers Spanish/Catalan fluency in any answer, capture it.

### 2b. Update mode

Read the current file. Show the user the existing values as a short summary, then ask:

> What would you like to change? You can say "budget → €450k, add Girona to locations" or paste a whole new section. Anything you don't mention stays as-is.

Apply only the requested changes. Bump `last_updated` to today's date.

### 3. Write the profile

Write to the absolute `$PROFILE_PATH` resolved above (i.e. `<repo-root>/profile/catalonia-realestate-profile.md`). Create the `profile/` directory first with `mkdir -p` if needed. Use this template (fill in the user's answers; today's date is in the current-date system context):

```markdown
---
created: YYYY-MM-DD
last_updated: YYYY-MM-DD
schema: 1
---

# Catalonia Real Estate — Buyer Profile

## Intent
[Buy-to-live / Investment (rental yield) / Holiday or second home]

## Budget
[Purchase ceiling €X] — [cash / mortgage]. [Deposit on hand if stated.]

## Financing
[Resident / Non-resident] · pre-approval: [yes, bank X / not yet]. [LTV note if relevant.]

## Legal status
[EU / non-EU citizen] · NIE: [obtained / not yet].

## Location
Target areas: [Barcelona neighborhoods / Girona / Tarragona / Lleida / Costa Brava / Costa Daurada / inland towns / ...]
Commute & lifestyle must-haves: [...]

## Property
Type: [flat / house / townhouse] · Bedrooms: [n] · Condition: [turnkey / open to reform]
Must-haves: [...]
Dealbreakers: [...]

## Timeline
[When they want to buy]

## Language
[English only / some Spanish / fluent Spanish / Catalan / ...]

## Notes
[Free-form: relocation reason, prior viewings, financing details, hard nos, etc.]
```

Preserve the `created` date if updating an existing file. Always update `last_updated`.

### 4. Confirm and hand off

After writing:

> Saved to `profile/catalonia-realestate-profile.md`. I'll use this on every Catalonia property answer and listing search from now on. Want to tweak anything, or shall I [continue with your question / wait for your next one]?

If invoked implicitly by the advisor, end with a brief one-line summary the advisor can pick up from, e.g.:

> Profile saved. Continuing with your original question.

Do **not** answer the user's real-estate question yourself — that's the advisor's job. Just save the profile and yield.

## Rules

- **One question per turn.** Send exactly one question, wait for the reply, then send the next. Batching the whole list overwhelms users — that's the failure mode this skill is built to avoid.
- **Seven questions, no more.** Intent, budget, financing, legal status, location, property, timeline+language. Don't add an eighth because you think it'd help — the user can always say more in their answers.
- **Never re-ask within the interview.** If an answer is terse or ambiguous, default and note it in the profile rather than firing a follow-up. The user can edit the file later; momentum matters more than completeness.
- **Never invent constraints.** If the user gave no dealbreakers, write `None stated.` — don't infer one.
- **No prices, no live data, no WebSearch/WebFetch.** This skill only handles user data. It never quotes prices, taxes, mortgage rates, or listings — that's the advisor's job.
- **No legal/financial/tax advice here.** You collect status (resident/EU/NIE); you do not advise on it. Capture the facts and let the advisor add the disclaimers.
- **No emojis. No filler.** Mirror the advisor's voice — short, plain, actionable.

## Common Mistakes to Avoid

- **Batching the questions** into one message. Ask one at a time.
- **Padding the question count past seven.** Defaults and free-form text on existing answers cover the rest.
- **Re-interviewing in update mode** instead of patching the existing file.
- **Skipping the project-root resolver and hardcoding a path.** Always run the Bash snippet — the absolute path differs per machine.
- **Writing inside a skill folder** (e.g. `skills/.../profile/`) instead of the **repo-root** `profile/`. The advisor reads from the repo root; a misplaced file means the gate sees `PROFILE_MISSING` forever.
- **Answering the real-estate question yourself** after saving. Yield to the advisor.
