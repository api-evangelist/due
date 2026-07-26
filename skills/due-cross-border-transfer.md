---
name: Quote and send a cross-border transfer
description: Price a cross-border / on-off-ramp transfer, create a recipient, initiate the transfer against the quote, and track it to settlement.
api: openapi/due-openapi-original.yml
operations: [get_v1-channels, post_v1-recipients, post_v1-transfers-quote, post_v1-transfers, get_v1-transfers-id]
---

# Quote and send a cross-border transfer

Move money across borders — fiat↔stablecoin on/off-ramps, fiat↔fiat, or stablecoin swaps — for a verified account.

## Auth
`Authorization: Bearer <api_key>` plus `Due-Account-Id: <acct_...>` to scope the sending account. The account must have passed KYC/KYB.

## Steps
1. (Optional) List available rails with `get_v1-channels` to pick a source/destination channel and confirm country/schema support.
2. Create the payout destination with `post_v1-recipients`, supplying the fields required by the destination's payment schema (bank/mobile/crypto — fetch schema field definitions from the object reference). The response `id` is prefixed `rcp_`.
3. Get a price with `post_v1-transfers-quote`, passing source and destination currencies and rails. The quote is time-bounded (`expiresAt`).
4. Create the transfer with `post_v1-transfers` against the returned quote and the recipient. The response `id` is prefixed `tf_`.
5. Track status with `get_v1-transfers-id` or the `transfer.status_changed` webhook (e.g. `payment_processed`).

## Rules
- Always create against a fresh quote; quotes expire. Re-quote on `err_quote_limit_*` errors and adjust the amount.
- Transfer intents are de-duplicated by client `reference` — reusing a reference with different details returns `err_transfer_intent_has_different_id`.
- Handle policy/compliance rejections: `err_policy_breached`, `err_aml_risk_score_hit`, `err_insufficient_balance`. See errors/due-error-codes.yml.
- In sandbox, simulate the incoming leg with `POST /dev/merge/v1/payin` to drive status updates and webhooks.
