---
name: Quote, preview, and execute a managed transfer
description: Move crypto from a connected account using Mesh's managed transfer flow — discover tokens/networks, quote fees, preview, then execute with user approval, and confirm via webhook.
api: openapi/mesh-integration-api-openapi.json
operations:
  - "GET /api/v1/transfers/managed/tokens"
  - "POST /api/v1/transfers/managed/quote"
  - "POST /api/v1/transfers/managed/preview"
  - "POST /api/v1/transfers/managed/execute"
---

# Quote, preview, and execute a managed transfer

Mesh's managed transfer API moves assets out of a connected account to a destination address, with a quote/preview/execute lifecycle.

## Auth
- `X-Client-Id` + `X-Client-Secret` headers. Base URL `https://integration-api.meshconnect.com`.

## Steps
1. `GET /api/v1/transfers/managed/tokens` (and `.../networks`) to confirm the destination `symbol`/`networkId` are supported.
2. `POST /api/v1/transfers/managed/quote` to get fees and min/max amounts across funding sources (crypto balance, cash, ACH/debit). Quotes for fiat funding are currently Coinbase-only.
3. `POST /api/v1/transfers/managed/preview` to validate the transfer and surface the exact amounts before committing.
4. `POST /api/v1/transfers/managed/execute` to run it. Transfers require end-user approval/MFA, which Mesh brokers.
5. Confirm settlement on the **transfer-status webhook** (`Succeeded` carries `TxHash`; multi-step bridges/swaps populate `TransferSteps`). See `asyncapi/mesh-transfers-webhooks.yml`.

## Rules & gotchas
- Respect exchange minimum-withdrawal amounts — surface `minAmount`/`minAmountInFiat` from the quote so a transfer is not rejected.
- Save `RefundAddress` from the webhook for reverse money movement / refunds.
- The `Pending` webhook is not guaranteed in sandbox; do not build logic that depends on receiving it during testing.
- This flow moves funds — treat `execute` as a safety-critical, human-approved action; never auto-execute without explicit user consent inside Link.
