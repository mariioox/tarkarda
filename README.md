# Takarda: Decision Dossier, Repository

This repository is the third of the assignment's three submitted
deliverables. The decision dossier PDF and the one-page summary are
separate files handed in alongside it, not duplicated here. What this
repository holds is exactly what the brief asks a repository for: the API
contract as a specification file that validates, and the C4 diagrams as
their own files rather than as images pasted into a PDF.

**Author:** Ehimare Okosun
**Repository:** https://github.com/mariioox/tarkarda

## Contents

```
spec/
  openapi.json           The HTTP contract, OpenAPI 3.1.0, JSON. Validates
                         clean under both openapi-spec-validator and
                         Redocly CLI.
diagrams/
  context.svg             C4 system context: Takarda and everyone it
                         talks to.
  container.svg           C4 container diagram: the runnable pieces
                         inside Takarda.
  component-ammend.svg    C4 component diagram: the amendment and
                         versioning pipeline, the part most likely to be
                         got wrong.
```

## The diagrams

All three are C4 diagrams, drawn by hand in Excalidraw using the C4
model's own color convention: dark blue for a person, medium blue for a
software system in scope, grey for an external system, lighter blue for a
container or component. Each carries its own legend.

- **context.svg.** Shows the people and organizations the system talks to
  (candidate, school, employer, council officer, regulator) plus the two
  external systems it depends on (the payment partner and the USSD
  aggregator), and leaves everything inside Takarda as a single box.
- **container.svg.** Opens that box. Shows the APIs, the pre-computation
  pipeline, the reconciliation job, and the three data stores (cache,
  primary database, read replicas), including how the burst-window versus
  off-window read path splits between them.
- **component-ammend.svg.** Opens the one container most likely to be got
  wrong: getting the amendment pipeline wrong either breaks the fifty-year
  reproducibility requirement or silently fails to propagate a correction
  to verification. Shows the request validator, the version writer, the
  supersede marker, the amendment logger, and the cache notifier, each
  with what it writes and where.

## Validating the API contract

```
pip install openapi-spec-validator
python3 -c "from openapi_spec_validator import validate; from openapi_spec_validator.readers import read_from_filename; s,_=read_from_filename('spec/openapi.json'); validate(s); print('valid')"

npx @redocly/cli lint spec/openapi.json
```

Both pass clean. The spec targets OpenAPI 3.1.0 rather than 3.0, since the
employer association's authentication is genuine mutual TLS, and
`mutualTLS` is not a valid security scheme type under 3.0. Two endpoints
(`POST /pin-purchases`, `GET /pin-purchases/{attemptId}`) are explicitly
public, `security: []`, with a comment explaining why: a candidate has no
account or credential before their first purchase.

## Conventions reflected in this contract and these diagrams

- Money is an integer count of kobo (`amountKobo` in the PIN purchase
  schema).
- A result row is never updated in place. Amendments append a new
  version, and the current view is the row with `superseded_at IS NULL`
  (see `component-ammend.svg` and the `PATCH
  /results/{examNumber}/amendments` operation).
- The employer association authenticates by mutual TLS, not an API key,
  because it is a genuine contracted relationship, not high-volume
  traffic (see ADR-006 in the decision dossier).

## What else was submitted

The decision dossier (business and product requirements, the architecture
decision records, the quality-attribute ranking, the work breakdown
structure, and the write-up of how this contract and these diagrams fit
into the wider design) and the one-page summary of the five most
consequential decisions were submitted as separate PDF files, per the
brief's own "what you submit" list. This repository is only the third
item on that list.
