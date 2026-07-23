---
name: resume-tailor
description: >
  Produce a compiled one-page PDF resume from the resume vault's golden-standard LaTeX template,
  curated either to a specific job posting or to a generalized target. Use this skill when the
  user says "tailor my resume for [posting/company]", "make a resume for the LANL job", "generate
  a resume PDF", "apply to [posting] - build the resume", "create a generalized resume",
  "build me a resume for [role type / themes]", "resume for ML engineer roles", or names themes
  to curate toward ("a resume emphasizing HPC and computer vision"). Two modes: POSTING mode
  targets one job-descriptions posting page (uses its Match Report, Gap Analysis, and Tailoring
  Notes); GENERALIZED mode targets a job-type archetype page or user-named focus themes (uses the
  archetype's "Resume Points for This Role Type"). Different from wiki-resume-bullets (which
  extracts an exhaustive bullet bank but produces no document) - this skill selects a curated
  subset from that bank, fills the LaTeX template, compiles it, and verifies the 1-page fit.
---

# Resume Tailor

Turn career evidence into a submission-ready PDF: select bullets from the vault's bullet bank,
fill the golden-standard template's section files, compile with Tectonic, and iterate until the
result passes the 1-page/no-overflow gates. Layout never changes — only content does.

## Before You Start

**Resolve config** — follow the Config Resolution Protocol in `llm-wiki/SKILL.md` (inline `@name`
override → walk up CWD for `.env` → `~/.obsidian-wiki/config` → prompt setup). This gives you
`OBSIDIAN_VAULT_PATH`. The vault must contain `resume/template/` (the golden-standard template —
`resume.cls`, `sections/`, `TEMPLATE-SPEC.md`, `build.ps1`) and a `job-descriptions/` category.
If `resume/template/` is missing, stop and tell the user the template must be set up first.

**Read `resume/template/TEMPLATE-SPEC.md` in full before selecting any content.** It is the
contract: per-section purpose, hard word/character budgets, the page-budget line-unit table, and
the ATS rules. Never fit content by editing `resume.cls` — trim content instead.

**Toolchain**: Tectonic at `%LOCALAPPDATA%\tectonic\tectonic.exe` (not on PATH). If missing,
install the latest `x86_64-pc-windows-msvc` zip from the tectonic-typesetting GitHub releases
into that directory (winget does not carry it).

## Write scope

This skill writes ONLY:
- `$VAULT/resume/tailored/<slug>/` (the output directory)
- `$VAULT/resume/bullet-bank.md` (regenerating a stale bank)
- one `## Resume` index line in `$VAULT/index.md`
- one `RESUME_TAILOR` line appended to `$VAULT/log.md`
- in POSTING mode: one link line under the posting page's `## Related` section

Never edit experience/skills/concepts pages, the template directory, or anything else.

## Step 1: Resolve the target and mode

**POSTING mode** — the user names a specific job/company ("the LANL job", "Buzz Solutions"):
match against `job-descriptions/*/` posting pages (check `index.md`'s Job Descriptions section
for the list with match percentages). If ambiguous, list candidates with their `match_overall`
and ask. Slug = the posting page's basename.

**GENERALIZED mode** — the user asks for a general resume, names a role type, or names themes:
- Role type ("ML engineer resume") → the archetype page `job-descriptions/<role-type>.md`.
- Free-form themes ("emphasize HPC and computer vision") → no archetype; the named themes are
  the curation targets. Record them verbatim in the curation notes.
- Plain "generalized resume" with no target → curate for the user's best-fit archetype (the
  index marks one as "best-fitting archetype") and say so; offer to redo against another.
Slug = `general-<role-type>` or `general-<theme-slug>`.

## Step 2: Ensure the generic evidence bank exists and is fresh

**Two-layer design (user decision, 2026-07-10):** the generic bank is the JD-agnostic *evidence
layer* — regenerated only when the wiki gains material, never per posting. The per-posting
*curated layer* (Step 3.5) is remade fresh for every job description.

The bank is `$VAULT/resume/bullet-bank.md` (frontmatter: `generated`, `pages_read`, `bullets`,
`flagged`). It is stale if `log.md` shows an `INGEST` or `RESTRUCTURE` entry newer than its
`generated` timestamp. If missing or stale, regenerate it by executing Steps 2–6 of
`~/.claude/skills/wiki-resume-bullets/SKILL.md` (full 2-hop traversal of every experience page,
per-page raw claim lists, formula tagging, honesty gate, completeness self-check) and persisting
the result to `resume/bullet-bank.md` — that skill itself stays read-only; the write is owned
here. Keep each bullet's formula tag, source `[[wikilinks]]`, and any `[UNVALIDATED]` /
`[PRELIMINARY]` flag in the bank.

## Step 3: Read the curation evidence

- The bullet bank (always).
- POSTING mode: the posting page in full — especially `## Requirements Scored`, `## Match
  Report`, `## Gap Analysis`, and `## Tailoring Notes` (verbatim JD phrases to mirror) — plus
  the parent archetype page's `## Resume Points for This Role Type`.
- GENERALIZED mode: the archetype page's `## Resume Points for This Role Type` and `## Keywords`
  material if an archetype was resolved; otherwise just the user's named themes.
- `education/` page's `## Resume-Ready Facts` for the EDUCATION section.

## Step 3.5: Build the per-JD curated bullet set — ALWAYS fresh, never reused

Write `$VAULT/resume/tailored/<slug>/curated-bullets.md`: the ENTIRE eligible bank re-expressed
for this specific target. This file is remade for every posting — never copy another posting's
curated set.

For every S1-S3 bank bullet:
1. **Re-draft its prose in the posting's vocabulary** — mirror Tailoring Notes phrases and exact
   JD keywords (ATS matches literal strings). Same fact, this JD's language.
2. **Map it to the scored requirement(s) it answers** (from the posting's Requirements Scored
   table; in generalized mode, to the user's named themes).
3. **Rank by relevance × substance tier**, then mark status: `SELECTED` (on the page),
   `FILL QUEUE` (ranked next-in candidates, roughly 2x what the page can hold), or `BENCH`
   (eligible but not competitive for this target).
4. **Apply the honesty gate and Gap Analysis exclusions at this layer**: S4/flagged bullets and
   anything conflicting with a scored ❌ go in an `EXCLUDED` section with the reason — they never
   enter the queue.
5. **Substance selects the bullet; vocabulary-mirroring only rewrites it (user rule, 2026-07-14).**
   ATS keyword coverage is mandatory, not optional — ATS engines and human screeners both need the
   posting's literal terms present, so ranking must never sacrifice a requirement going unmatched.
   But satisfy that by **rewriting the strongest available bullet into the JD's vocabulary**, never
   by picking a weaker bullet just because its default wording already happens to contain the JD's
   words. A bullet's un-redrafted phrasing is not evidence of its relevance — every bullet gets
   redrafted at Step 1 regardless, so "it already sounds like the JD" is worth nothing when ranking.
   See Step 4 rule 15 for what counts as "strongest."

All fourteen curation rules (Step 4 list) apply to the re-drafting here. The .tex sections in
Step 6 are then built from this file's SELECTED entries, and the fill/trim loops in Step 7 draw
from its FILL QUEUE in rank order.

## Step 4: Select and curate under budget

Fill each template section within TEMPLATE-SPEC.md's budgets (bullets ≤ 2 rendered lines each,
2-4 per role, skills lines never wrap, etc.):

1. Map the target's requirements/themes to bank themes; prefer bullets whose wording mirrors the
   posting's Tailoring Notes phrases (ATS parsers match literal strings).
2. **Maturity filter (user rule, 2026-07-08):** select only claims describing work that was
   truly matured and completed to some extent — shipped releases, delivered artifacts, finished
   experiments with final measured results. Exclude mid-investigation work, pivots without a
   delivered endpoint, "projected" outcomes, and anything the source frames as in-progress —
   even when unflagged, if you couldn't defend it in an interview as *finished*, it stays out.
3. Pick bullets per role, strongest and most target-relevant first (budget minimum 2 per role;
   the most recent role absorbs extras under the fill-the-page rule in Step 7). Rewrite bank
   bullets to fit character budgets without dropping the number/scale. Benchmark/evaluation
   bullets should land on the top measured result when a defensible number exists — "produced a
   ranking" is half a claim; "top macro F1 0.77" finishes it (use the replicated/verified figure,
   not one the bank flags as provisional).
4. **Honesty constrains the claim, not the sentence (user rule, 2026-07-09):** wiki scope
   qualifiers — verified windows, sample sizes, provenance markers — determine which numbers you
   may state; they never appear in the bullet itself. "56 tickets in 6 weeks" is the claim;
   "screenshot-verified window" is the citation, and citations live in curation-notes.md. If a
   number is only true *with* a hedge printed beside it, choose a different number rather than
   printing the hedge.
5. **Lead with the accomplishment, not the process:** one idea per bullet — what changed and why
   it mattered. Verification/QA methodology ("tracking functional parity", "validating
   pane-by-pane") is interview evidence, not bullet content, unless the posting explicitly asks
   for testing/validation skills (then frame it as the accomplishment itself).
6. **Read-aloud test:** if a phrase only makes sense to someone who has read the wiki ("verified
   window", "non-SME", "pane-by-pane"), translate it to plain language or cut it. Research
   jargon counts: say "fully reproducible experiments", not "fixed-seed reproducibility";
   describe consolidation as "standalone scripts → one universal pipeline", not notebook counts.
7. **Number consistency across adjacent bullets (user rule, 2026-07-09):** when related bullets
   carry different true counts of the same thing (e.g. 22 surveyed / 19 pipelined / 18
   benchmarked), state the count ONCE in the bullet where it's strongest and truest, and write
   the neighbors without competing numbers — never let 18/19/22 variants collide on the page,
   and never round different facts to one fake number.
8. **Time windows and outcome nouns (user rule, 2026-07-09):** include a time window only when
   speed itself is the achievement ("shipped in 3 weeks under deadline"); volume and scope
   metrics stand alone — "56 tickets in 6 weeks" becomes "50+ reports". Prefer outcome nouns
   (reports, releases, models) over internal process nouns (tickets, sprints, weekly updates) —
   the reader cares about what exists now, not the tracker it moved through.
9. **Thread grouping (user rule, 2026-07-09):** within a role, group bullets by narrative
   thread (e.g. product/application work together, ML/research work together), ordered so the
   target's highest-priority theme comes first; within a thread, lead with the shipped outcome,
   then the how. Exception: the single strongest bullet stays #1 overall even if it breaks the
   grouping — recruiters weight the first bullet heavily.
10. **In-progress work inside completed bullets:** genuinely prototyped-but-unadopted work (e.g.
   CI automation tested in a fork) may appear only as a clause inside a completed-work bullet,
   explicitly labeled ("prototyped X") — never as the bullet's headline claim.
11. **Selective bolding:** carry the bank's bolded span into the .tex as `\textbf{...}` — at most
    1-2 spans per bullet, on the load-bearing metric or an exact JD keyword; bullets with nothing
    remarkable get none (restraint is what makes the anchors work). ATS parsers strip formatting,
    so bolding is reader-only; bold Garamond runs slightly wider, so recheck wraps after adding it.
12. SKILLS lines: lead with the target's exact keyword terms the user actually has evidence for.
13. Record a priority ranking (1 = cut last) for every selected bullet — the trim and fill loops
    use it. Also queue 2-3 next-priority matured bullets as fill candidates.
14. Date ranges use a plain hyphen `-`, never en/em dashes (user preference, recorded in
    TEMPLATE-SPEC.md).
15. **Substance beats literal phrase-matching (user rule, 2026-07-14).** When several bank bullets
    could answer the same requirement, rank by what the bullet actually demonstrates, not by how
    closely its wording already resembles the JD:
    - **Positive-discovery bullets outrank error-correction bullets at equal or even lower tier.**
      A bullet whose story is "I found and fixed my own mistake" (a seeding bug, a split bug, an
      inflated benchmark later caught) reads as remedial regardless of its S-tier — it tells the
      reader something was wrong until you noticed, not that you built or discovered something.
      Prefer a same-tier or one-tier-lower bullet that shows a positive result, comparison, or
      finding instead, unless no other evidence exists for that requirement.
    - **Analytical/statistical rigor bullets are easy to under-rank and shouldn't be.** Hypothesis
      testing, ablation/parameter sweeps, feature analysis, and other bullets that demonstrate
      methodological rigor routinely lose a keyword-matching pass because their default wording
      doesn't share literal terms with the JD — check for these explicitly before defaulting to
      whichever bullet already "sounds like" the requirement.
    - **Process/administrative bullets (data reformatting, file-format conversion, editorial
      trimming) are usually the weakest evidence for a requirement, even when their wording is a
      close literal match.** "Converted data into a standard format" and "condensed a document to
      N pages" describe tasks, not outcomes — prefer a bullet with a measured result or a genuine
      finding, then redraft it to hit the same keyword.
    - This does not relax the honesty gate or the ATS ambition — it changes *which* bullet gets
      redrafted to hit a keyword, never whether the keyword gets hit. Run the existing "Coverage
      check" (end of Step 3.5's curated-bullets.md) to confirm every scored requirement still has
      a SELECTED/FILL QUEUE bullet mapped to it after re-ranking by substance.

## Step 5: The honesty gate — hard requirement, do not skip

Apply `wiki-resume-bullets` Step 5 to every selected bullet, plus one tailoring-specific rule:
a bullet must never imply experience the target's Gap Analysis scored as missing (e.g. if the
posting scores "containerization ❌" or "LLM/RAG ❌", no bullet may read as production Docker or
LLM experience). Bullets flagged `[UNVALIDATED]`/`[PRELIMINARY]`, or sourced from `^[inferred]`/
`^[ambiguous]` claims, are either excluded or rephrased as "designed / prototyped / benchmarked"
— never as delivered production outcomes. Log every exclusion or rephrase in the curation notes.
When unsure whether a claim is hedged, treat it as hedged.

## Step 6: Write the output directory

Create `$VAULT/resume/tailored/<slug>/` containing:
- `resume.tex` — copy of the template's `master-template.tex` (rename only; adjust nothing but
  the `\ATSGlyphs` comment if the user asked for ATS glyphs).
- `sections/*.tex` — copies of the template's section files with ONLY the macro arguments and
  `\item` lines replaced. Keep every budget comment header intact. Header section uses the
  canonical contact block in `resume/template/TEMPLATE-SPEC.md` — NOT the reference docx, whose
  phone number is a placeholder.
- **Escape LaTeX specials in all content text**: `%` → `\%`, `&` → `\&`, `#` → `\#`, `$` → `\$`,
  `_` → `\_`. A raw `%` silently comments out the rest of the bullet — it compiles clean and
  passes every gate while truncating the text, so grep the sections for unescaped specials
  before compiling.

## Step 7: Compile and verify

Copy `resume.cls`, `fonts/`, and `build.ps1` from `resume/template/` into the job directory
(fresh copy every compile — the template stays the single source of truth), then run
`./build.ps1 resume` there. Gates: exactly 1 page, zero `Overfull \hbox`, zero missing glyphs.

Iterate on failure: 2 pages → cut the lowest-priority bullet and recompile; overfull line →
shorten that specific line's text. Never touch `resume.cls`, never shrink fonts or spacing.

**Fill the page (user rule, 2026-07-08):** a passing build is not done if it leaves dead space
at the bottom. Read the PDF; if content ends well above the bottom margin, add the next queued
fill-candidate bullet (matured work only, Step 4) to the MOST RECENT role and recompile. Repeat
until adding one more bullet overflows to 2 pages, then remove that last bullet — the calibrated
maximum. The finished page ends near the bottom margin. Cap the whole compile loop at ~8
iterations, then present the tension to the user. Finish with a visual Read of the PDF.

## Step 8: Curation notes + wiki citizenship

Write `<slug>/curation-notes.md` with frontmatter (`category: resume`, tags per
`_meta/taxonomy.md`, `summary`, `created`, and `relationships:` → the posting/archetype page
`derived_from`, `[[resume/bullet-bank]]` `uses`) and body: bullets chosen per section with source
wikilinks and priority ranking, the honesty-gate audit (every exclusion/rephrase and why), which
JD phrases were mirrored, and compile stats. Then:
- add the output under `## Resume` in `index.md` (create the section after Job Descriptions if
  absent), mirroring the index's existing line style;
- POSTING mode: append `- [[resume/tailored/<slug>/curation-notes]] — tailored resume (YYYY-MM-DD)`
  under the posting page's `## Related`;
- append to `log.md`:
  `- [TIMESTAMP] RESUME_TAILOR target="<posting or archetype/themes>" mode=<posting|general> curated_bullets=N bullets_selected=N flagged_excluded=N compiles=N pages=1 overfull=0`

## Answer format

> **Resume built:** `<path to resume.pdf>` — 1 page, N compile iterations.
>
> [Table: section → bullets chosen (with source wikilinks) → target requirement/theme each maps to]
>
> **Honesty audit:** [what was excluded or rephrased and why; UNVALIDATED/PRELIMINARY claims that
> stayed out]
>
> **Gaps to be ready for:** [requirements the target lists that no honest bullet covers — flag
> for cover letter / interview, per the posting's Gap Analysis]
