---
name: Collect funds with a virtual account
description: Create a programmable virtual account (virtual IBAN or stablecoin address) that auto-converts and settles incoming deposits, then reconcile via webhooks.
api: openapi/due-openapi-original.yml
operations: [post_v1-virtual-accounts, get_v1-virtual-accounts-key, get_v1-virtual-accounts, post_v1-webhook-endpoints]
---

# Collect funds with a virtual account

Issue static receiving details for an account so deposits are automatically routed, converted, and settled into a target currency or rail.

## Auth
`Authorization: Bearer <api_key>` with `Due-Account-Id: <acct_...>`. The account must have passed KYC/KYB (some rails require endorsements, e.g. USD/EUR/GBP/AED virtual accounts).

## Steps
1. Create the virtual account with `post_v1-virtual-accounts`, specifying the collection currency/rail and the settlement target (currency + destination). The response holds the receiving details (e.g. virtual IBAN or deposit address) keyed by `key`.
2. Render/return the receiving details with `get_v1-virtual-accounts-key`; list all with `get_v1-virtual-accounts`.
3. Register a webhook with `post_v1-webhook-endpoints` subscribing to transfer events so you are notified when deposits arrive and settle.
4. On each `transfer.status_changed` event, fetch the object and reconcile against your ledger.

## Rules
- Some channels gate virtual-account creation behind KYC endorsements — handle `err_endorsement_required` by completing the relevant endorsement flow.
- Micro-deposit verification: check received micro-deposits by filtering transactions for the virtual account id.
- Verify webhook signatures (Ed25519, `X-Webhook-Signature`) against the endpoint's stored public key before trusting a payload.
- In sandbox, simulate an incoming deposit with `POST /dev/merge/v1/payin`.
