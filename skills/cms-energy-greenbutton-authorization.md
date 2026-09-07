---
name: cms-energy-greenbutton-authorization
description: >-
  Obtain and manage a Consumers Energy (CMS Energy) customer's Green Button Connect My Data
  authorization — register as a third party, run the Green Button OAuth flow with the
  CONSUMERSENERGY utility id, verify the authorization, and revoke it. Covers sandbox test
  scenarios.
api: Consumers Energy Green Button Connect My Data
spec: openapi/cms-energy-authorizations-api-openapi.yml
base: https://utilityapi.com/api/v2
auth: bearer token + per-customer OAuth 2.0 authorization code
operations:
  - listAuthorizations
  - getAuthorization
  - deleteAuthorization
---

# Get a Consumers Energy customer's data authorization

Consumers Energy runs Green Button Connect My Data on the UtilityAPI EE/DER Engagement Platform.
Consumers Energy's own Green Button host names that platform as its API documentation and its
third-party dashboard, so the base host below is the correct one for this program.

## 0. Register first — this is a human gate

Register at <https://greenbutton.consumersenergy.com/third-party/register>. You are assigned a
`client_id` and you start in **sandbox mode**: real Consumers Energy customers cannot authorize
you. Consumers Energy reviews the registration and emails you when you can switch to Live in
settings. There is no self-serve path past this. Utility id is `CONSUMERSENERGY`.

## 1. Send the customer to the authorization URL

Green Button OAuth is the OAuth 2.0 authorization code grant. Your base authorization URL,
`client_id` and editable `redirect_uri` list live in your settings after registration.

Parameters: `response_type=code` (always), `client_id`, `redirect_uri` (optional — defaults to
the customer's receipt), `scope` (optional), `state` (optional tracking string).

Customer authentication defaults to Consumers Energy single sign-on. Add `prompt=on` if you do
not want an existing session reused.

## 2. Exercise it against test accounts first

Append the authentication option to the scope string: `auth-{type}[-{value}]`.

```
scope=FB%3D4_16_51%3BAdditionalScope%3Dauth-test-test_commercial
```

| Test account | Scenario |
|---|---|
| `test_residential` | 1 account, 2 services (electric and gas) |
| `test_commercial` | 2 accounts, 6 services (electric and gas) |
| `test_empty` | No eligible accounts — the error path |

`auth-sso` is the default and is what you want for real customers. Test credentials still work
when set as scope parameters, so you can automate against them.

## 3. Confirm the authorization landed

`listAuthorizations` — `GET /authorizations`
`getAuthorization` — `GET /authorizations/{id}`

Send the token as `Authorization: Bearer <token>` (or `?access_token=`). Object ids are opaque
UID strings — they can look numeric; do not parse them as integers.

Do not poll for it. Subscribe to `authorization_created`, `authorization_update_started`,
`authorization_update_finished_successful` and `authorization_update_finished_errored` — see
`asyncapi/cms-energy-webhooks.yml`. Webhook delivery is retried hourly for 72 hours and is
**not ordered**, so reconcile by event `uid` and `ts`.

## 4. Revoking — read this before you call it

`deleteAuthorization` — `DELETE /authorizations/{id}`
(Green Button equivalent: `DELETE /Authorization/{auth_uid}`)

Revocation **deletes all collected data — bills and intervals included — and the access
credentials**. There is no undo and no restore window. Regaining access requires sending the
customer a brand-new authorization form. Treat this as human-in-the-loop: confirm with the
account holder before calling it.

Expiry is the other terminal state: `authorization_expiring_soon` fires 24 hours out, then
`authorization_expired`, after which credentials are deleted and you must send a re-authorization
form.

The customer can revoke at any time from
<https://greenbutton.consumersenergy.com/my-authorizations>. Revocation only stops future
access — data you already downloaded is yours to delete, and the utility tells customers to
contact you directly about it.

## Rules that will bite you

- **No idempotency.** No `Idempotency-Key` header exists anywhere on this API. A retried
  `DELETE` after a timeout is not protected. Check state with `getAuthorization` before retrying
  a write.
- **404 does not mean "does not exist".** It also means "you cannot see it".
- **403 means wrong token class**, not wrong permissions — the Green Button API has three token
  classes and each endpoint family accepts exactly one.
- **Rate:** 1 request per second for everything except bill and interval downloads. See
  `rate-limits/cms-energy-rate-limits.yml`.
