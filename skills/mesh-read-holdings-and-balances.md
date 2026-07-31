---
name: Read a connected account's holdings and balances
description: Fetch real-time crypto holdings, fiat balances, and an aggregated portfolio view for an account a user has connected through Mesh.
api: openapi/mesh-integration-api-openapi.json
operations:
  - "POST /api/v1/holdings/get"
  - "POST /api/v1/balance/get"
  - "GET /api/v1/holdings/portfolio"
---

# Read a connected account's holdings and balances

Once a user has connected an account (see the connect-and-deposit skill), read their positions and balances.

## Auth
- `X-Client-Id` + `X-Client-Secret` headers. Base URL `https://integration-api.meshconnect.com`.
- Read requests carry the connected account's auth token / broker context in the POST body.

## Steps
1. `POST /api/v1/holdings/get` — crypto holdings for the connected account (real-time call to the underlying integration).
2. `POST /api/v1/balance/get` — real-time fiat balances for the account.
3. `GET /api/v1/holdings/portfolio` — the aggregated portfolio with market values (use `POST /api/v1/holdings/value` when you also need total value and performance).

## Rules & gotchas
- Holdings/balance reads hit the live integration, so they can be slower and are rate-sensitive; cache where the UX allows.
- A `401`/`403` usually means the connection expired or the API key lacks access — reconnect the account or refresh the auth token (`POST /api/v1/token/refresh`).
- Error bodies are plain JSON (not RFC 9457). See `errors/mesh-problem-types.yml`.
