# Takarda: Decision Dossier

Paper-only design assignment for the National Secondary Certificate Council.
Nothing here is an application. There is no database, no query, no code
that runs.

**Author:** Ehimare Okosun

## What this submission is made of

The brief asks for three separate things, so they're delivered as three
separate files, not one document wearing three hats:

1. **`takarda-one-page-summary.pdf`**, read first: the five most
   consequential decisions in two sentences each.
2. **`takarda-dossier.pdf`**, the full decision dossier: stakeholder
   conflicts, Parts A-E, the ADRs, the diagrams, everything else.
3. **This repository**, containing the API contract as a spec file that
   validates (`spec/openapi.json`) and the C4 diagrams as their own files
   (`diagrams/*.mmd`, `*.svg`), not pasted into the PDF as images only.

## Repository layout

```
docs/
  00-summary.md          One-page summary, five decisions, built separately
                         as takarda-one-page-summary.pdf, not part of the
                         dossier PDF
  00b-conflicts.md       Where the stakeholders conflict, and the one
                         impossible request, opens the dossier PDF
  01-brd.md              Business Requirements Document
  02-prd.md              Product Requirements Document
  03-frd.md              Functional Requirements Document
  04-wbs.md              Work Breakdown Structure
  05-qa-ranked.md        Quality attributes, ranked, with cost
  pattern.md             The platform architecture pattern (Part B)
  06-adrs.md             Architecture Decision Records (9, two one-way doors)
  07-tradeoffs.md        Trade-off register
  08-not-building.md     What is deliberately outside the first release
  13-diagrams-inline.md  C4 diagrams with embedded images + leaves-out notes
  09-part-c.md           The wire: protocols, TLS, contract, PIN fix
  10-part-d.md           Data model and the five predicted queries
  11-part-e.md           How the AI was used, and where it was checked
assets/
  univaciti-logo.png     Logo used on the cover page (extracted from the
                         reference submission, with transparency restored)
build/
  cover.html             Cover page markup (logo, title, submitted-by block)
  style.css              Full stylesheet, shared by the dossier and the
                         one-page summary
  full_dossier.md         Concatenated conflicts + Parts A-E (generated,
                         not hand-edited, excludes 00-summary.md)
  body.html / full.html   Intermediate HTML for the dossier (generated)
spec/
  openapi.json           HTTP contract, JSON, validates clean under both
                         openapi-spec-validator and Redocly CLI
diagrams/
  context.mmd / .svg / .png            C4 system context
  container.mmd / .svg / .png          C4 container diagram
  component-amend.mmd / .svg / .png    C4 component diagram, amendment pipeline
takarda.pdf                    The assignment brief
takarda-dossier.pdf             The full decision dossier (conflicts + Parts A-E)
takarda-one-page-summary.pdf    The one-page summary, read first, separate file
```

## Conventions used throughout

- Every number asserted is from the brief, computed from a stated
  assumption, or labelled an estimate with the reasoning shown.
- Money is an integer count of kobo, and the result-checker PIN is 350000.
- A result row is never updated in place. Amendments append a new version,
  and the current view is the row with `superseded_at IS NULL`.
- Cache-ahead pre-computation is scoped to the results-day burst window
  only, not treated as a permanent serving mechanism (see ADR-001 and Part
  E, Incident 3, for why this correction matters).
- USSD and SMS are designed as two distinct decisions (ADR-003a, ADR-003b),
  not one, because they are different protocols with different failure
  modes (see Part E, Incident 4).
- Every decision is defended against the alternatives it rejected, and
  every rejection names the specific property of *this* system that ruled
  it out.

## How the PDFs were assembled

**The dossier** (`takarda-dossier.pdf`). Markdown sections in `docs/` are
concatenated in reading order (stakeholder conflicts, Part A, Part B, C4
diagrams, Part C, Part D, Part E, `00-summary.md` deliberately excluded),
rendered to HTML with Pandoc (`--toc --number-sections`, images embedded),
then merged with a hand-built cover page (`build/cover.html`) carrying the
Univaciti logo and a matching stylesheet (`build/style.css`, heading colour
`#2E74B5` sampled directly from the reference submission's own PDF), then
rendered to PDF with `wkhtmltopdf`.

**The one-page summary** (`takarda-one-page-summary.pdf`). `docs/00-summary.md`
alone, rendered to HTML with Pandoc (no table of contents, no section
numbering, since it's a single flat page), styled with the same
`build/style.css` so it visually matches the dossier, then rendered to PDF
with `wkhtmltopdf`. It carries its own byline (title, author, date) since
it has no cover page of its own and travels as an independent file.

Diagrams are authored once as Mermaid source (`diagrams/*.mmd`) and
rendered to both `.svg` (the diagram-as-a-file deliverable) and `.png`
(embedded in the PDF, since the PDF pipeline used here doesn't rasterize
SVG directly).

Page numbers are added in a separate pass after `wkhtmltopdf` runs, since
the installed build has no header/footer support: a small script
(`reportlab` + `pypdf`) draws the number and merges it onto every page
except the cover, matching the reference submission's own convention of
leaving the cover unlabeled.

Three real formatting bugs were caught and fixed during assembly, not
assumed away because a command ran without error:
- Concatenated markdown files need a blank line at each join, without it,
  Pandoc silently folds the next file's top-level heading into the
  previous file's subsection tree. Caught by inspecting the rendered table
  of contents.
- A bold label line (e.g. `**Alternatives rejected.**`) immediately
  followed by a `- ` list with no blank line between them gets flattened
  into a run-on paragraph instead of a bullet list. Caught by inspecting
  the rendered ADR pages, then fixed across all nine ADRs at once with a
  script rather than by hand, to make sure none were missed.
- The Univaciti logo, extracted from the reference PDF, initially lost its
  transparency (a flat black box appeared behind it) because the base
  image and its soft mask are stored separately inside a PDF, and both had
  to be extracted and recombined to get a clean transparent PNG.
- The cover page's vertical centering looked fine in the browser but not
  in this wkhtmltopdf build: a `display:table` / `table-cell` centering
  trick was silently ignored, and even a plain `margin-top` only applied a
  fraction of the value set in CSS. Caught by rendering the cover to an
  image and measuring the actual pixel position, then fixing it with a
  margin value tuned against that measurement instead of trusted at face
  value.

## Validating the API contract

```
pip install openapi-spec-validator
python3 -c "from openapi_spec_validator import validate; from openapi_spec_validator.readers import read_from_filename; s,_=read_from_filename('spec/openapi.json'); validate(s); print('valid')"

npx @redocly/cli lint spec/openapi.json
```

Both pass clean as of the final build. Two real issues were caught and
fixed during this process (not suppressed): `mutualTLS` is not a valid
security scheme type under OpenAPI 3.0 (the spec was upgraded to 3.1.0,
which supports it, since the employers' association is genuinely
authenticated by client certificate, see ADR-006), and two endpoints
(`POST /pin-purchases`, `GET /pin-purchases/{attemptId}`) had no `security`
field at all, which the linter correctly flagged. Both were given an
explicit `security: []` with a comment explaining why they are
deliberately public at this point in the flow, rather than an oversight.
