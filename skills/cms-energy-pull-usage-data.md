---
name: cms-energy-pull-usage-data
description: >-
  Pull authorized Consumers Energy (CMS Energy) customer energy data — meters, bills and interval
  usage — as JSON, or as Green Button ESPI 1.1 Atom XML, once a customer authorization exists.
api: Consumers Energy Green Button Connect My Data
spec: openapi/cms-energy-meters-api-openapi.yml
base: https://utilityapi.com/api/v2
auth: bearer token (ESPI access_token for the Green Button surface)
operations:
  - listMeters
  - getMeter
  - listBills
  - listIntervals
  - getEspiBatch
---

# Pull Consumers Energy usage data

Prerequisite: a live customer authorization. See `cms-energy-greenbutton-authorization`.

Two projections of the same data. Choose one and stay on it.

- **JSON** (`/api/v2`) if you want ordinary objects.
- **Green Button ESPI 1.1 Atom XML** (`/DataCustodian/espi/1_1/resource`) if you already ingest
  the standard. This is what makes the program portable across utilities.

## JSON path

1. `listMeters` — `GET /meters` (filter with `?meters=NNN`)
2. `getMeter` — `GET /meters/{id}`
3. `listBills` — `GET /bills?meters=NNN`
4. `listIntervals` — `GET /intervals?meters=NNN`

Watch `bill_count` / `bill_coverage` and `interval_count` / `interval_coverage` on the meter to
know what has actually been collected. Data streams in as it is collected — a meter can serve
bills while its historical-collection job is still pending, so do not wait for "done".

## Green Button path

`getEspiBatch` —
`GET /DataCustodian/espi/1_1/resource/Batch/Subscription/{subscriptionId}/UsagePoint/{usagePointId}`

Returns `application/atom+xml`. Related ESPI resources sit on the same tree:

- `/Subscription/{auth_uid}/UsagePoint` and `/UsagePoint/{meter_uid}`
- `.../MeterReading` and `.../MeterReading/{reading_uid}/IntervalBlock`
- `.../UsageSummary` (the ESPI name for bills)
- `.../ElectricPowerQualitySummary`
- `/Batch/Subscription/{auth_uid}` for everything under one authorization
- `/Batch/RetailCustomer/{auth_uid}` for account details

The batch endpoints take an ESPI **`access_token`** (per authorization). The bulk endpoints and
the authorization list take a **`client_access_token`**. Registration endpoints take a
**`registration_access_token`**. Sending the wrong class returns 403, not 401.

Persistent UUIDs are supported, so you can ingest only what is new across authorizations rather
than re-reading everything.

## Rules that will bite you

- **1 per minute.** Bill and interval downloads are rate-guided to once per minute; everything
  else to once per second. 429 carries `Retry-After`. See
  `rate-limits/cms-energy-rate-limits.yml`.
- **202 is not a failure.** A request too large to answer synchronously returns 202 with an empty
  body and a `Retry-After`. Wait it out and re-request; do not treat it as an error and do not
  hammer.
- **Timezones are deliberately mixed.** `created`, `revoked` and similar platform timestamps are
  UTC. Timestamps parsed off bills and intervals carry the **utility's local timezone** so you
  can convert them correctly. Do not normalize to UTC before reading the offset.
- **Bulk URLs expire in days.** Save the payload on your side when a notification hands you a
  `/Bulk/{bulk_uid}` link; you may not be able to download it again.
- **Revocation deletes what has been collected.** Do not treat this API as your archive of
  record if the customer's authorization is short.
- **Errors are not RFC 9457.** `{"error", "error_description", "url"}`. New `error` values will
  appear; handle unknown ones.
