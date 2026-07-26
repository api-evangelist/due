---
name: Onboard a customer account with KYC/KYB
description: Create a Due account for a business or individual, run KYC/KYB verification, and confirm the account is active before transacting.
api: openapi/due-openapi-original.yml
operations: [post_v1-accounts, get_v1-account-categories, post_v1-kyc, get_v1-kyc, get_v1-accounts-accountid]
---

# Onboard a customer account with KYC/KYB

Use this skill to register an end-user (business or individual) on Due and take them through verification so they can send, receive, hold, and convert funds.

## Auth
All calls use `Authorization: Bearer <api_key>`. Use the sandbox host `https://api.sandbox.due.network` for testing and `https://api.due.network` in production. Per-account operations are scoped by the `Due-Account-Id` header.

## Steps
1. (Optional) Fetch the allowed categories with `get_v1-account-categories` — lists differ for `business` vs `individual`.
2. Create the account with `post_v1-accounts`. Required fields: `type` (`business`|`individual`), `name`, `email`, `country` (ISO 3166-1 alpha-2), `category`. The response returns `id` (prefixed `acct_`), a `kyc.link`, and a `tos.link`.
3. Start verification with `post_v1-kyc` (or send the user to the hosted `kyc.link`). Retrieve the standard KYC/KYB link/status with `get_v1-kyc`.
4. Poll `get_v1-accounts-accountid` or subscribe to the `bp.kyc.status_changed` webhook. KYC status values: `pending`, `passed`, `resubmission_required`, `failed`.
5. Only once `kyc.status = passed` and ToS is `accepted` can the account transact.

## Rules
- Store the returned `account.id` — every wallet, transfer, and virtual account references it.
- Prefer webhooks over polling for status changes; verify webhook signatures (Ed25519, `X-Webhook-Signature`).
- Errors return `{ error_code, message, http_code }`; handle `err_kyc_not_passed` before attempting transfers. See errors/due-error-codes.yml.
- In sandbox, approve the applicant with `POST /dev/sumsub/applicants/{accountId}/approve` to simulate a completed KYC.
