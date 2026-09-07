---
name: cms-energy-check-outages
description: >-
  Read current Consumers Energy (CMS Energy) electric outages in Michigan — by county, by city,
  or as individual outage jobs with cause and estimated restoration — from the utility's public
  ArcGIS services. No key, no account, no registration.
api: Consumers Energy Outage Map ArcGIS REST API
spec: openapi/cms-energy-outage-map-api-openapi.yml
base: https://www.consumersenergy.com/arcgispublic/rest
auth: none
operations:
  - listServices
  - describeServiceDashboardMapService
  - describeServiceDashboardLayer
  - queryServiceDashboardLayer
  - describeOutageMapService
  - describeOutageMapLayer
  - queryOutageMapLayer
---

# Check Consumers Energy outages

Consumers Energy publishes its outage map data as an anonymous Esri ArcGIS REST service on its
own domain. Nothing here requires a key.

## 1. Confirm the services are up

`listServices` — `GET /services?f=json`

Expect `CEOutageMap` (MapServer), `ServiceDashboard` (FeatureServer and MapServer). If the body
comes back as HTML you asked for the default format; always pass `f=json`.

## 2. Pick the right layer

`describeServiceDashboardMapService` — `GET /services/ServiceDashboard/MapServer?f=json`

| Layer | Contents |
|---|---|
| 0 | CustomersInterruptedPercent |
| 1 | NumberOfInterruptions |
| 2 | NumberOfWireDowns |
| 3 | CE_County_Outages |
| 4 | CE_City_Outages |

`describeOutageMapService` — `GET /services/CEOutageMap/MapServer?f=json`

| Layer | Contents |
|---|---|
| 0 | Service Territory |
| 1, 2, 5 | Number of Customers Affected (the outage jobs) |
| 3, 4, 6 | Percentage of Customers Affected |

Call `describeServiceDashboardLayer` / `describeOutageMapLayer` before your first query on a new
layer and read `fields[]` — field names are not stable across layers and are not documented
anywhere else.

## 3. Query by county

`queryServiceDashboardLayer` — `GET /services/ServiceDashboard/MapServer/3/query`

```
?where=1%3D1&outFields=*&returnGeometry=false&f=json
```

Attributes: `COUNTY_NAME`, `COUNTY_ID`, `CUSTOMER_COUNT`, `OUTAGE_COUNT`, `PCT_CUSTOMERS_OUT`,
`CRITICAL_COUNT`, `PRIORITY_COUNT`, `LAST_UPDATED`.

Always set `returnGeometry=false` unless you are drawing a map — the polygons dominate the
payload. Use `outFields` to project only what you need.

To ask how bad it is right now without pulling rows, use `returnCountOnly=true`.

## 4. Query individual outage jobs

`queryOutageMapLayer` — `GET /services/CEOutageMap/MapServer/1/query`

Attributes include `JOB_ID`, `CUSTOMER_COUNT`, `CAUSE_CD`, `CAUSE_DESC`, `OUTAGE_TIME`,
`EST_TIME_RESTORATION`, `EST_TIME_ETR`, `CREW_ASSIGNED`.

Layers 1, 2 and 5 all carry "Number of Customers Affected" at different aggregation levels — read
`GROUP_LEVEL` before you sum anything, and never add totals across those three layers.

## Rules that will bite you

- **Cap of 1000.** `maxRecordCount` is 1000 per response on every service here. Page with
  `resultRecordCount` and `resultOffset`; do not assume a single call returned everything.
- **HTTP 200 is not success.** ArcGIS returns `{"error":{"code","message","details"}}` under a
  200 for several failure classes. Parse the body. See
  `errors/cms-energy-problem-types.yml`.
- **Dates are epoch milliseconds** in ArcGIS attribute output, not the ISO 8601 the Green Button
  side uses.
- **Do not crawl this.** `robots.txt` disallows `/arcgispublic/*`. No request-rate limit is
  published, which is not permission — poll on a sane interval.
- **This is outage data, not API status.** If the outage map is empty, the grid is healthy; it
  says nothing about whether the API is healthy.
