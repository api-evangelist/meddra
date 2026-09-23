---
name: meddra-version-impact
description: Work out what a new MedDRA release changes for a specific set of coded terms, before upgrading a safety database to it.
api: meddra:meddra-api
base_url: https://mapisbx.meddra.org
operations:
  - GET /api/rel
  - POST /api/di
  - POST /api/vr
  - GET /api/hist/{code}/{htype}/{lang}/{rsview}
  - POST /api/hier
generated: '2026-09-17'
method: generated
source: openapi/meddra-api-openapi.yml
---

# Assess the impact of a MedDRA version upgrade

MedDRA changes twice a year. Terms are added, demoted, made non-current and moved in the hierarchy. Before a
safety database adopts a new version, someone has to answer: *what does this do to the codes we already
hold?* That is exactly what these operations are for.

## Steps

1. **Establish the from/to versions.** `GET /api/rel` lists supported versions and language combinations.
   The impact question is always between two specific releases.

2. **Run the data impact report.** `POST /api/di` with the list of terms or codes you hold, plus the two
   versions. It returns the changes between those versions for exactly those terms. Note the `force` flag —
   leave it `false` so validation errors surface instead of being run past.

3. **Run the version report.** `POST /api/vr` for the broader version-level view across releases.

4. **Explain any individual change.** `GET /api/hist/{code}/{htype}/{lang}/{rsview}` returns the change
   history of one term. `GET /api/hist/{term}/{lang}/{rsview}` finds the terms that can feed it.

5. **Re-derive the hierarchy where it moved.** `POST /api/hier` with the affected terms returns the
   hierarchy for the new version, with `hlt`/`hlgt` flags controlling which level names come back.

## Rules

- Do not upgrade a production safety database on an impact report you have not read. The whole point of the
  report is that some of your existing codes stop being current.
- Report results as "from version X to version Y" — an impact statement without both versions is meaningless.
- The API is read-only: nothing here changes your data. The upgrade itself is your own system's job, and
  this API gives you no way to undo it.
- `429` means throttled; back off. Impact reports over long term lists are the most likely calls to hit it.
