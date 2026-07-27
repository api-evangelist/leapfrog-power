# Leap (leapfrog-power)

Leap (Leapfrog Power, Inc.) is a San Francisco based energy software company whose primary domain leap.ac 301-redirects to www.leap.energy. Leap sits in the private, unmandated layer of the United States energy stack — between distributed energy resources and the wholesale markets — letting technology brands build and scale virtual power plants without owning market access. Its software-only platform aggregates residential and commercial battery storage, smart thermostats, heat pumps, HVAC and EV charging, registers those assets into CAISO, NYISO, PJM and utility demand response programs, and settles the revenue back to the partner. Leap publishes a genuinely open developer portal at developer.leap.energy with eight anonymously downloadable OpenAPI definitions covering meter creation, enrollment and idle periods, meter details, market nominations, dispatch, webhooks, interval data upload and revenue and analytics. Its API posture is open documentation over a closed door — every specification and guide can be read without an account, but no key can be self-issued — keys are created only inside a Leap-provisioned partner account and prospective partners are told to contact an account manager or partners@leap.ac. No data-sharing mandate applies to Leap. It is not a utility, not a retailer and not a designated data holder anywhere; it is a downstream recipient of consumer data that California's investor-owned utilities are compelled to share, integrating PG&E, SCE and SDG&E through their Share My Data authorization flow. Leap's documentation never names Green Button, ESPI, OpenADR, IEEE 2030.5, OCPP or CIM. Consumer usage and settlement data are reachable through the API but only for a partner's own consented, enrolled meters, and Leap publishes no open grid or market data at all.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/leapfrog-power/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/leapfrog-power/refs/heads/main/apis.yml)

## Tags

- Energy
- United States
- Electricity
- Grid
- Demand Response
- DER
- Virtual Power Plant
- Energy Markets
- Storage Flexibility
- EV Charging
- Smart Metering

## Timestamps

- **Created:** 2026-07-27
- **Modified:** 2026-07-27

## APIs

### Leap Create Meters API

Create or update meters on the Leap platform individually or in bulk, accepting CSV or JSON input and returning a job ID, plus endpoints to list meter upload jobs, check job status and manage provisional assets. OpenAPI 3.0.2, 7 operations.

- **Human URL:** [https://developer.leap.energy/reference/createmeterbatchjob](https://developer.leap.energy/reference/createmeterbatchjob)
- **Base URL:** `https://api.leap.energy`

#### Tags

- Create Meters
- Provisional Assets

#### Properties

- [OpenAPI](openapi/leapfrog-power-create-meters-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Documentation](https://developer.leap.energy/docs/partner-created-meters)
- [API Reference](https://developer.leap.energy/reference/createmeterbatchjob)

### Leap Meter Enrollment API

Retrieve and search enrollment information for meters, including current enrollment status, participation preferences, associated programs and required actions, alongside idle period, disenrollment and market participation management. OpenAPI 3.0.2.

- **Human URL:** [https://developer.leap.energy/reference/getmeterenrollment](https://developer.leap.energy/reference/getmeterenrollment)
- **Base URL:** `https://api.leap.energy`

#### Tags

- Meter Enrollment

#### Properties

- [OpenAPI](openapi/leapfrog-power-meter-enrollment-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Documentation](https://developer.leap.energy/docs/sync-meter-inventory-v2)
- [API Reference](https://developer.leap.energy/reference/getmeterenrollment)

### Leap Meter Details API

Get and search meter details such as customer, utility, site and device information across a partner's meter inventory, with optional filtering by request parameters. OpenAPI 3.0.2.

- **Human URL:** [https://developer.leap.energy/reference/getmeterdetails](https://developer.leap.energy/reference/getmeterdetails)
- **Base URL:** `https://api.leap.energy`

#### Tags

- Meter Details

#### Properties

- [OpenAPI](openapi/leapfrog-power-meter-details-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Documentation](https://developer.leap.energy/docs/utility-meters-vs-devices)
- [API Reference](https://developer.leap.energy/reference/getmeterdetails)

### Leap Meter Nomination API

Suggest, retrieve and delete nomination suggestions for individual meters or in bulk for each applicable program and time period. Suggestions are reviewed by Leap before becoming actual market nominations. OpenAPI 3.1.0.

- **Human URL:** [https://developer.leap.energy/reference/getmeternominationsuggestions](https://developer.leap.energy/reference/getmeternominationsuggestions)
- **Base URL:** `https://api.leap.energy`

#### Tags

- Nominations

#### Properties

- [OpenAPI](openapi/leapfrog-power-nominations-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Documentation](https://developer.leap.energy/docs/bidding-introduction)
- [API Reference](https://developer.leap.energy/reference/getmeternominationsuggestions)

### Leap Dispatch API

Receive and search grid dispatch instructions at meter and group level through Leap Dispatch API V2, including webhook URL management for meter and group dispatches. OpenAPI 3.0.3, served from the /v2 path.

- **Human URL:** [https://developer.leap.energy/reference/dispatch-event](https://developer.leap.energy/reference/dispatch-event)
- **Base URL:** `https://api.leap.energy/v2`

#### Tags

- Meter Dispatches
- Group Dispatches

#### Properties

- [OpenAPI](openapi/leapfrog-power-dispatch-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Documentation](https://developer.leap.energy/docs/dispatch-automation-v2)
- [API Reference](https://developer.leap.energy/reference/dispatch-event)

### Leap Webhook Subscription API

List, create, update, delete and test webhooks that deliver Leap event notifications such as meter, connect and dispatch events to a partner receiver URL. OpenAPI 3.0.1.

- **Human URL:** [https://developer.leap.energy/reference/listwebhooks-2](https://developer.leap.energy/reference/listwebhooks-2)
- **Base URL:** `https://api.leap.energy`

#### Tags

- Webhooks

#### Properties

- [OpenAPI](openapi/leapfrog-power-webhooks-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Documentation](https://developer.leap.energy/docs/webhook-setup)
- [API Reference](https://developer.leap.energy/reference/listwebhooks-2)

### Leap Revenue and Analytics API

Retrieve settlement and performance data — monthly revenue reports, annual revenue data, revenue report versions, event performance and unresponsive meter reporting — for a partner's enrolled fleet. OpenAPI 3.0.3.

- **Human URL:** [https://developer.leap.energy/reference/getperiodicreports](https://developer.leap.energy/reference/getperiodicreports)
- **Base URL:** `https://api.leap.energy`

#### Tags

- Revenue

#### Properties

- [OpenAPI](openapi/leapfrog-power-revenue-analytics-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Documentation](https://developer.leap.energy/docs/revenue-settlement-data)
- [API Reference](https://developer.leap.energy/reference/getperiodicreports)

### Leap Interval Data Upload API

Submit and monitor partner-supplied interval meter data — upload statuses, data validation errors, aggregated intervals and meter-level intervals — for meters where Leap does not receive utility data directly. OpenAPI 3.0.1, published as JSON.

- **Human URL:** [https://developer.leap.energy/reference/listintervaldatauploads](https://developer.leap.energy/reference/listintervaldatauploads)
- **Base URL:** `https://api.leap.energy`

#### Tags

- Interval Data Upload

#### Properties

- [OpenAPI](openapi/leapfrog-power-interval-data-upload-openapi.json) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Documentation](https://developer.leap.energy/docs/event-performance-interval-data)
- [Documentation](https://developer.leap.energy/docs/upload-interval-data-sftp)

## Common Properties

- [Website](https://www.leap.energy/)
- [DeveloperPortal](https://developer.leap.energy/)
- [Documentation](https://developer.leap.energy/docs/home)
- [APIReference](https://developer.leap.energy/reference/)
- [GettingStarted](https://developer.leap.energy/docs/getting-started)
- [Authentication](https://developer.leap.energy/docs/api-key-authentication)
- [ChangeLog](https://developer.leap.energy/changelog)
- [LLMsTxt](https://developer.leap.energy/llms.txt)
- [SignUp](https://partner.leap.energy/)
- [Support](https://support.leap.energy/support/solutions)
- [StatusPage](https://status.leap.energy/)
- [Blog](https://www.leap.energy/blog)
- [TermsOfService](https://www.leap.energy/terms-of-service)
- [PrivacyPolicy](https://www.leap.energy/privacy-policy)

## Maintainers

- Kin Lane — kin@apievangelist.com
