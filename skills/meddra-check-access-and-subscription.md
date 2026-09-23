---
name: meddra-check-access-and-subscription
description: Confirm the MedDRA API is up, that your token works, and that your organisation's or a business partner's MedDRA subscription is valid before relying on either.
api: meddra:meddra-api
base_url: https://mapisbx.meddra.org
operations:
  - GET /api/status
  - GET /api/rel
  - GET /api/lang/{langt}
  - POST /api/sv
generated: '2026-09-17'
method: generated
source: openapi/meddra-api-openapi.yml
---

# Check MedDRA API access and subscription status

## Steps

1. **Is the API up?** `GET /api/status` — the one operation that needs **no** authentication. A 200 with
   `"API is Running"` means the service is reachable; a documented `400` means it reached the service but the
   database connection failed. Use this to distinguish "MedDRA is down" from "my token is wrong" before you
   raise anything with the help desk.

2. **Does my token work?** `GET /api/rel` with `Authorization: Bearer <token>`. A `200` returns the release
   list. A `401` with `WWW-Authenticate: Bearer` means the token is missing, expired or lacks the `meddraapi`
   scope — get a new one from `https://mid.meddra.org/connect/authorize`.

3. **What can I ask for?** `GET /api/lang/{langt}` lists the supported translations. Combine with the
   release list from step 2: not every language exists in every version, and an unknown pair is a documented
   `400` ("Language/Version Does Not exist").

4. **Is a subscription valid?** `POST /api/sv` confirms your own subscription status or that of a business
   partner. This is the operation to use before sharing MedDRA-coded data with a partner organisation — the
   MedDRA licence is per-subscriber, and this is how the MSSO lets you check.

## Rules

- Treat `GET /api/status` as a health check, not a status page — MedDRA publishes no status page, so this
  endpoint is the only availability signal there is.
- Do not use `POST /api/sv` as a lookup for organisations you have no business relationship with.
- MSSO subscriptions run for twelve months and are renewed annually; a partner that was valid last year is
  not necessarily valid today.
