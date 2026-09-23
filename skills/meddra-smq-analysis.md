---
name: meddra-smq-analysis
description: Run a Standardised MedDRA Query analysis over a list of terms or codes to find which safety topics they belong to, choosing broad, narrow or algorithmic scope.
api: meddra:meddra-api
base_url: https://mapisbx.meddra.org
operations:
  - GET /api/rel
  - POST /api/gt
  - POST /api/smqa
  - POST /api/detail
generated: '2026-09-17'
method: generated
source: openapi/meddra-api-openapi.yml
---

# Run an SMQ analysis

A Standardised MedDRA Query groups terms that together describe one safety topic. SMQ analysis answers the
question a safety reviewer actually asks: *given this list of coded events, which safety topics are in play?*

## Steps

1. **Pin version and language.** `GET /api/rel`. Carry `ver` and `lang` into every body below.

2. **List the SMQs available.** `POST /api/gt` returns the top-level terms for a language and version —
   with the SMQ browser view it returns the full SMQ list and synonym-group information. Use it to resolve
   SMQ names to codes rather than assuming a code.

3. **Run the analysis.** `POST /api/smqa` with an `smqaDTO` body:
   - `lang`, `ver` — the pinned dictionary.
   - `search` — **`1` broad, `2` narrow, `3` algorithmic**. This choice changes the answer materially. Narrow
     favours specificity, broad favours sensitivity, algorithmic applies the SMQ's own combination rules.
     State which one you used whenever you report a result.
   - `filters` — an SMQ code list to restrict the analysis.
   - `force` — `false` runs data validation first and returns only the errors if any exist; `true` runs the
     analysis regardless. **Prefer `false`**: a silent validation error is how a wrong safety signal gets
     reported.

4. **Explain any hit.** `POST /api/detail` on a term code returns the SMQs that term belongs to, which is the
   evidence for why the analysis matched it.

## Rules

- Never mix scopes in one result set. Broad and narrow results are not comparable.
- Never run with `force: true` unless a human has seen the validation errors and decided to proceed.
- Re-run after every dictionary release: SMQ membership changes between versions.
- `429` means throttled — back off exponentially; no `Retry-After` is published.
