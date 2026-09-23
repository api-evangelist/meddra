---
name: meddra-code-an-adverse-event
description: Take verbatim adverse-event text, find the matching MedDRA term, and read its code, currency and full SOC-to-LLT hierarchy for a stated dictionary version and language.
api: meddra:meddra-api
base_url: https://mapisbx.meddra.org
operations:
  - GET /api/rel
  - POST /api/search
  - POST /api/detail
  - POST /api/type
generated: '2026-09-17'
method: generated
source: openapi/meddra-api-openapi.yml
---

# Code an adverse event against MedDRA

MedDRA coding means mapping a clinician's verbatim text to a Lowest Level Term (LLT), then reading the
Preferred Term (PT) and the System Organ Class (SOC) it rolls up to. Every answer is version- and
language-specific, so pin those first.

## Before you start

- You need an OAuth 2.0 bearer token from `https://mid.meddra.org/connect/authorize`, scope `meddraapi`,
  against an active MSSO subscription. Send it as `Authorization: Bearer <token>` on every call below.
- This API is read-only. `POST` here means "send a query body", not "change something".

## Steps

1. **Pin the dictionary version and language.** `GET /api/rel` returns the supported release list, and
   `GET /api/lang/{langt}` returns the translations. Never hard-code a version: MedDRA ships a new one every
   March and September, and a code's currency can change between them. Carry the chosen `ver` and `lang` in
   every subsequent body.

2. **Search the verbatim text.** `POST /api/search` with a `SearchDTO` body. The required fields are
   `bview` (`SOC` or `SMQ` — use `SOC` for coding), `rsview`, `language`, `version` and `stype` (`1` for a
   regular search). `addlangs` returns additional translations alongside the result; `filters` narrows to
   specific SOC or SMQ codes. Expect matches at several hierarchy levels — coding normally selects an LLT.

3. **Read the term's detail.** `POST /api/detail` with a `MedTypeDTO` body carrying the 8-digit `code`.
   This returns the term name, its primary and secondary SOC assignment, whether an LLT is current, and the
   SMQs the term belongs to. **Check currency before using an LLT** — a non-current LLT must not be used to
   code new data.

4. **Walk the hierarchy.** `POST /api/type` with `MedTypeDTO` to get the parents, children or the full
   hierarchy of the term. Use it to confirm the PT and SOC you will report, not to guess them.

## Rules

- A MedDRA code without a version and language is not an answer. Record all three together.
- Do not cache a code's currency across dictionary versions; re-check after every release.
- MedDRA content is licensed. What you retrieve is governed by the MedDRA licence, not by the API's
  technical terms — do not redistribute retrieved terminology outside your subscription's scope.
- On `429`, back off: every operation declares a throttling limit. No `Retry-After` header is published, so
  use exponential backoff.
- On `400`, the contract gives you a status code and nothing machine-readable. The documented meanings are
  failed model validation, an unknown language/version combination, insufficient authorization, and invalid
  input — re-check `ver`/`lang` against `GET /api/rel` first, that is the most common cause.
