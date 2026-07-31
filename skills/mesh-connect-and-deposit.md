---
name: Connect an account and take a crypto deposit
description: Mint a Link Token, launch Mesh Link so a user connects an exchange/wallet and deposits crypto to your address, then confirm settlement via webhook.
api: openapi/mesh-integration-api-openapi.json
operations:
  - "GET /api/v1/integrations"
  - "POST /api/v1/linktoken"
---

# Connect an account and take a crypto deposit

Use Mesh Link to let a user connect a crypto exchange or self-custody wallet and send a deposit to your platform. Mesh handles auth, MFA, and per-integration quirks.

## Auth
- Server-to-server calls use two API-key headers: `X-Client-Id` and `X-Client-Secret` (from the Mesh dashboard). Sandbox and production keys are separate.
- Base URL: `https://integration-api.meshconnect.com` (sandbox: `https://sandbox-integration-api.meshconnect.com`).

## Steps
1. (Optional) `GET /api/v1/integrations` to discover supported exchanges/wallets and their categories.
2. `POST /api/v1/linktoken` server-side to mint a **Link Token**. Set `transferType: deposit` and the destination(s) in `transferOptions.toAddresses` (each entry is a `networkId` + `symbol` **ticker** + address). Pass your own `userId` and `transactionId` so you can reconcile later. The token is single-use and expires in 10 minutes.
3. Hand the Link Token to the client SDK and call `openLink()` **on the connect-button click** (not page load, so it does not expire). The user picks an integration and authenticates inside Link.
4. Wait for the **transfer-status webhook** (see `asyncapi/mesh-transfers-webhooks.yml`) — do not credit the user on the SDK `onTransferFinished` callback alone. Credit only on `TransferStatus: Succeeded` (which carries the `TxHash`).

## Rules & gotchas
- Use the token **ticker** (e.g. `USDC`), never an ERC-20 contract address, for `symbol`.
- Add every host domain (including `localhost:PORT`) under Dashboard > Account > API keys > Access, and add `*.meshconnect.com` to your CSP `frame-src`.
- Verify webhooks: HMAC-SHA256 over the **raw body**, base64, compared to `X-Mesh-Signature-256`. Deduplicate on `EventId`. Return `200` in < 200ms and process async.
- CEX deposits can take minutes to 24h to reach `Succeeded`; build for the wait. See `errors/mesh-problem-types.yml` for the full failure catalog.
