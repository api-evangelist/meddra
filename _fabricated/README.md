# Quarantined fabricated artifacts (2026-09-17)

Everything in this directory was **fabricated** — it was never published by MedDRA / the MSSO.

The OpenAPI files under `openapi/` (and the `_original/meddra-terminology-openapi.yml` they were split
from, which the provenance manifest wrongly recorded as `harvested`) declare
`servers: [https://api.meddra.example.com/v1]` — an RFC 2606 reserved example domain that has no DNS
record. The host does not exist and never did; the operations, schemas and `X-API-Key` security scheme
were authored by an earlier API Evangelist pass, not harvested. The Postman/OpenCollection files under
`collections/` were derived from those specs, so they inherit the defect.

MedDRA's **real** API contract was found on 2026-09-17 at
`https://mapisbx.meddra.org/swagger/v1/swagger.json` (HTTP 200, OpenAPI 3.0.1, 16 operations,
`info.contact.email: wadmin@meddra.org`) and is now saved verbatim at
`openapi/_original/meddra-api-openapi.json`. Nothing in this directory should be scored, published,
re-derived from, or restored.
