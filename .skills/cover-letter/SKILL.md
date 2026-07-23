---
name: cover-letter
description: >
  Produce a compiled one-page PDF cover letter for a specific job posting, arguing from the
  posting's curated evidence rather than summarizing the resume. Use this skill when the user
  says "write a cover letter for [posting/company]", "cover letter for the LANL job", "generate
  the cover letter", or "apply to [posting] - I need the letter". Always POSTING-targeted (a
  generalized cover letter is a contradiction - if asked, explain and offer to target a posting).
  Different from resume-tailor (which selects bullets into the resume template) - this skill
  builds a 250-400 word argument from 2-3 of that posting's curated bullets, fills the
  coverletter.cls template, compiles it, and verifies the 1-page fit. Requires the posting's
  tailored resume + curated-bullets.md to exist first (offer to run resume-tailor if not).
---

# Cover Letter

Turn a posting's curated evidence into a one-page letter that makes ONE argument for an
interview. The letter selects the 2-3 strongest proofs, explains their relevance, and connects
them to the posting's top-ranked requirements — it never re-lists the resume.

## Before You Start

**Resolve config** — follow the Config Resolution Protocol in `llm-wiki/SKILL.md` (inline `@name`
override → walk up CWD for `.env` → `~/.obsidian-wiki/config` → prompt setup). This gives you
`OBSIDIAN_VAULT_PATH`.

**Read `resume/template/COVER-LETTER-SPEC.md` in full before drafting.** It is the contract:
budgets, the content checklist, the one-argument rule, style/banned-phrase lists, and accuracy
rules. Never fit content by editing `coverletter.cls` — cut words instead.

**Prerequisite chain:** the posting must already have `resume/tailored/<slug>/curated-bullets.md`
(the honesty-gated, requirement-mapped evidence layer this skill draws from). If missing, stop
and offer to run `resume-tailor` first — never draft a letter straight from the generic bullet
bank or the wiki.

**Toolchain**: Tectonic at `%LOCALAPPDATA%\tectonic\tectonic.exe` (not on PATH; `build.ps1`
knows the path).

## Write scope

This skill writes ONLY:
- `$VAULT/resume/tailored/<slug>/cover-letter.tex` (+ compiled `.pdf`/`.log`, and a fresh copy
  of `coverletter.cls` from the template)
- a `## Cover Letter` section appended to that posting's `curation-notes.md`
- one `COVER_LETTER` line appended to `$VAULT/log.md`

Never edit the template directory, posting pages, or wiki content pages.

**References data (related, but NOT this skill's job):** `$VAULT/resume/references/reference-entries.tex`
is the user-maintained data file (one `\ReferenceEntry` per person). Never invent, edit, or
reorder reference people — that file is user-owned. `resume/references/references.tex` compiles
it standalone (`./build.ps1 references` in that directory).

**Every cover letter attaches a references page by default (user decision, 2026-07-12):** end
the letter with `\newpage`, `\RefHeading{References}`, then
`\input{../../references/reference-entries.tex}` (relative path from
`tailored/<slug>/cover-letter.tex` up to `resume/references/`). `coverletter.cls` already defines
matching `\RefHeading`/`\ReferenceEntry` macros, so this reads identically to the standalone
sheet. This makes the letter a deliberate 2-page document — compile with
`./build.ps1 cover-letter -MaxPages 2` (add 1 to whatever page limit otherwise applies, e.g.
`-MaxPages 3` if a posting override from Step 2 already permits 2 letter pages). This is a
standing default, not something that needs a posting's permission.

## Step 1: Resolve the posting

Match the user's target against `job-descriptions/*/` posting pages (same fuzzy matching as
resume-tailor Step 1; list candidates with `match_overall` if ambiguous). Slug = the posting
page's basename. Confirm `resume/tailored/<slug>/` exists with `curated-bullets.md` and a
compiled resume — the letter is the second half of an application package, not a standalone.

## Step 2: Read the evidence

- The posting page in full — `## Requirements Scored` (the ranking is already done),
  `## Gap Analysis` (hard exclusions), `## Tailoring Notes` (verbatim JD phrases), and the
  company/mission prose at the top. **Also check for any application-instructions text on the
  page or in its source material for a stated cover letter length/content requirement** (e.g.
  "submit a comprehensive cover letter (2 pages or less) that addresses the job requirements,
  desired skills, and education eligibility"). Default is 1 page — only deviate when the posting
  says so; see COVER-LETTER-SPEC.md's "Posting override" and "Comprehensive mode" sections for
  exactly what changes (page limit, word budget, and whether full-requirement coverage replaces
  the default 2-3-requirement argument).
- `tailored/<slug>/curated-bullets.md` — SELECTED and FILL QUEUE are the candidate evidence;
  EXCLUDED stays excluded.
- `tailored/<slug>/curation-notes.md` — the resume's choices, so the letter complements rather
  than duplicates its emphasis.
- The company's `entities/` page if one exists — mission/team material for the
  company-specific-reason slot.
- `resume/template/TEMPLATE-SPEC.md` canonical contact block — the letter header must match the
  resume header exactly.

## Step 3: Rank what the letter must answer

From Requirements Scored, take the top of the ranking: (1) mandatory qualifications, (2)
responsibilities the JD repeats, (3) "highly preferred" items, (4) transferable support. The
letter addresses the top 2-3 ONLY — coverage of everything else is the resume's job. Note which
requirement the posting itself flags as the differentiator (e.g. a verbatim-strength score or an
institutional connection) — that usually seeds the argument.

## Step 4: State the argument, then select evidence

**One letter, one argument.** Write the thesis as one sentence FIRST (it goes in the curation
notes verbatim), e.g. "I already run this role's core loop, here is proof twice." Then select
2-3 experiences from SELECTED/FILL QUEUE that prove it, using the evidence hierarchy:

1. Direct experience performing the same work
2. Closely related experience with comparable tools/methods
3. Transferable experience demonstrating the same competency
4. Relevant coursework or independent projects

Prefer evidence carrying: a specific problem, an action the candidate personally performed,
named tools/methods, a measurable or observable result, and a clean connection to a top-ranked
requirement. Never use weak evidence just to touch a keyword; off-thesis evidence stays on the
resume. At least two of the chosen proofs must contain results (a number, or an observable
outcome like a shipped release when no honest number exists).

## Step 5: Draft under the checklist contract

Fill every slot of COVER-LETTER-SPEC.md's content checklist (opening: title + company + themes +
sourced company-specific reason; two complementary evidence paragraphs; optional third only when
it adds coverage; closing: interest + contribution + ask + thanks) — **but never instantiate the
checklist as sentence skeletons**. Formulaic scaffolding ("I used X to Y, resulting in Z. This
experience strengthened my ability to A...") is the top way letters get binned as
AI-template output; vary sentence and paragraph construction every letter.

Hard rules while drafting:
- 250-400 words, 3-5 short paragraphs, one page — UNLESS Step 2 found the posting stating its
  own limit (e.g. "2 pages or less"), in which case use that limit and its "Comprehensive mode"
  content requirements per COVER-LETTER-SPEC.md. Never expand beyond 1 page on your own
  initiative; the posting has to say so.
- **Company-specific reason must be sourced** from the posting page or the company's entity page.
  If the vault has nothing real to cite, flag the missing slot to the user — never improvise
  flattery.
- Accuracy: facts ONLY from curated-bullets.md and the posting/company pages. Never invent
  metrics/tools/titles/dates; never imply experience the Gap Analysis scores ❌ (this letter's
  wording included — e.g. no "containers", no LLM/RAG implication when scored missing);
  distinguish direct from transferable experience explicitly.
- Style: active voice; JD terminology integrated naturally; banned-phrase list enforced; no
  adjectives doing a number's job; read-aloud test on every sentence.
- **No colons, no semicolons, no dashes in prose** (user rule, 2026-07-11; semicolons added
  2026-07-14): never use a colon or semicolon to introduce or join a clause — split into two
  plain sentences instead ("I bring the same rigor to analysis: I validated..." is wrong; "I bring
  the same rigor to analysis. I validated..." is right). Never use a hyphen or em/en dash as
  sentence punctuation. Rewrite compound modifiers into plain phrasing rather than hyphenating
  ("GPU-enabled" -> "environments that use GPUs", "one-click" -> "install with a single click",
  number ranges "19-36%" -> "19 to 36%"). Exception: literal identifiers that aren't stylistic
  choices — phone numbers, URLs, an org's actual designation (e.g. "A-4") — keep their hyphens.
  Cover-letter-only rule; unrelated to the resume's plain-hyphen convention.
- **Company-specific reason must be genuine, not a jargon name-drop** (user rule, 2026-07-14):
  sourcing the reason from the posting doesn't mean lifting its most specific internal program or
  division terminology just because the text is available. If the candidate couldn't explain the
  term in an interview (a named portfolio, a specific initiative, an internal acronym), it reads
  as reaching. Prefer a general, honest statement of wanting to contribute to the organization's
  mission, naming the org and the plain-language kind of work, not a specific program skimmed off
  the JD.
- **Warm closing register** (user rule, 2026-07-11): avoid "I would welcome the chance/
  opportunity to..." — prefer "It would be great to talk more about..." or similar.
- **No GPA-vs-minimum framing** (user rule, 2026-07-12): never write that a GPA "exceeds"/"is
  above"/"is well above" a posting's stated minimum, and don't restate the GPA number to make
  that comparison — applying already implies the candidate clears the posting's bars, so the
  comparison reads as filler. In comprehensive mode, cover education with relevant **completed
  coursework** instead (name specific classes mapping to the posting's listed areas, per the
  education page's Relevance Map). GPA and threshold framing stay on the resume/transcript only.
- **No cute or gimmicky rhetorical devices** (user rule, 2026-07-14): avoid tidy contrast
  constructions used for effect ("X, not a checkbox" / "X, not a formality") and cheeky metaphors.
  State interest and fit plainly. If a sentence's contrast exists only to sound clever rather than
  to convey information, cut the contrast and state the fact.
- The letter must not repeat the resume's exact bullet sentences — same facts, prose register.

## Step 6: Write and compile

Write `tailored/<slug>/cover-letter.tex` using ONLY `coverletter.cls` macros (`\LetterSender`,
`\LetterDate`, `\LetterSalutation`, body paragraphs, `\LetterClosing`; `\LetterRecipient` only
when a real named person is known — never a bare "Hiring Manager / Company" block). Letterhead =
top-left sender block at body size (the resume's banner belongs to the resume alone; contact
data still matches TEMPLATE-SPEC's canonical block). Escape LaTeX
specials (`\%` `\&` `\#` `\$` `\_`) — a raw `%` silently truncates the paragraph. Copy
`coverletter.cls` fresh from `resume/template/` (fonts/ and build.ps1 are already in the
directory from the resume build), then run `./build.ps1 cover-letter` — or, when Step 2 found a
posting-stated page limit N, `./build.ps1 cover-letter -MaxPages N`.

Gates: pages within limit (1 by default, or the posting's stated N) · 0 overfull · 0 missing
glyphs · word count within budget (250-400 for 1 page, scaled ~250-400/page for a posting
override — count it, the compiler won't). Iterate by cutting words, never by touching the
class — an override raises the ceiling, it doesn't invite padding to fill it.

**Fill the page** (user rule, 2026-07-14): a passing compile is not done if it leaves a visible
paragraph of dead space at the bottom of page 1. Read the PDF after compiling — if content ends
well above the bottom margin, add the optional evidence paragraph 3 (per COVER-LETTER-SPEC.md)
using real unused material from `curated-bullets.md`, never by padding existing sentences.
Recompile; trim instead of overflowing the page/word ceiling. Finish with a visual Read of the PDF.

## Step 7: Validate, record, log

Run COVER-LETTER-SPEC.md's checklist plus these before declaring done: correct title/company
everywhere; zero content from any other application; top mandatory requirements addressed; every
claim traceable to curated-bullets.md; ≥2 proofs with results; opening is company-specific;
closing asks for the conversation; dates/status accurate.

Then append to `curation-notes.md` a `## Cover Letter` section: the thesis sentence, the
evidence chosen (with curated-bullets #s and source wikilinks), the sourced company-specific
reason and where it came from, exclusions honored, word count, and compile stats. Append to
`log.md`:
`- [TIMESTAMP] COVER_LETTER target="<posting>" words=N paragraphs=N evidence=N compiles=N pages=1 overfull=0`

## Answer format

> **Cover letter built:** `<path to cover-letter.pdf>` — 1 page, N words, N compile iterations.
>
> **The argument:** [thesis sentence]
>
> [Table: paragraph → evidence used (curated-bullets #) → requirement it answers]
>
> **Honesty audit:** [gaps not overstated, exclusions honored, any flagged missing slots]
