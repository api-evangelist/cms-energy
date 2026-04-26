# CMS Energy (cms-energy)

CMS Energy is an energy holding company whose primary subsidiary is Consumers Energy, an electric and natural gas utility serving customers in Michigan. Consumers Energy participates in the Green Button Connect My Data (GBCMD) program, exposing customer-authorized energy usage data to third parties via OAuth 2.0 — typically brokered through UtilityAPI — for use in energy management, demand response, EV charging, solar, and sustainability applications.

**URL:** [https://raw.githubusercontent.com/api-evangelist/cms-energy/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/cms-energy/refs/heads/main/apis.yml)

## Scope

- **Type:** Index
- **x-type:** company
- **Access:** 3rd-Party

## Tags

- Electric
- Energy
- Green Button
- Michigan
- Natural Gas
- Utility

## Timestamps

- **Created:** 2026-03-21
- **Modified:** 2026-04-23

## APIs

### Consumers Energy Green Button Connect My Data API

Exposes customer-authorized electric and natural-gas usage data to registered third parties using the NAESB ESPI / Green Button standard. Authorization uses OAuth 2.0 with single sign-on or test-account scopes; data is delivered as Green Button XML or JSON via UtilityAPI's standardized endpoints (Meters, Bills, Intervals, Billing Summaries). Third parties register with Consumers Energy, are issued a client_id, and start in sandbox mode before being approved for live data.

**Human URL:** [https://utilityapi.com/docs/utilities/consumersenergy](https://utilityapi.com/docs/utilities/consumersenergy)

#### Tags

- Energy Usage, Green Button, OAuth2, Smart Meter, Utility

#### Features

- **Green Button Standard:** NAESB ESPI / Green Button XML for usage point and interval data.
- **Meters Endpoint:** Retrieve individual meter information for an authorized customer.
- **Bills Endpoint:** Retrieve bill history and charges.
- **Intervals Endpoint:** Retrieve granular interval-usage time series.
- **Billing Summaries:** Retrieve account-level summary data.
- **OAuth 2.0:** Customer authorization via OAuth 2.0 with SSO or test scopes.
- **Sandbox Test Accounts:** Built-in test residential and commercial scenarios for development.

#### Use Cases

- **Energy Efficiency:** Pull usage data into energy-efficiency and benchmarking apps.
- **Demand Response:** Power demand-response and load-management programs.
- **Solar and EV:** Inform solar sizing and EV-charging optimization with real usage.
- **Sustainability Reporting:** Aggregate usage into Scope 2 emissions and ESG reporting.

## Common Properties

- [Website](https://www.cmsenergy.com)
- [Consumers Energy](https://www.consumersenergy.com)
- [Consumers Energy on UtilityAPI](https://utilityapi.com/docs/utilities/consumersenergy)
- [About Consumers Energy](https://www.cmsenergy.com/about-cms-energy/consumers-energy/)
- [Contact Us](https://www.cmsenergy.com/contact-us/default.aspx)
- [Privacy Statement](https://www.cmsenergy.com/privacy-statement/default.aspx)
- [X](https://twitter.com/CMSEnergy)
- [LinkedIn](https://www.linkedin.com/company/cms-energy/)

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
