---
name: Pull Leap revenue, settlement and performance data
description: Retrieve monthly and annual revenue for an enrolled fleet, aggregate it by meter, customer, utility, load type or market group, track report versions, and diagnose unresponsive meters.
api: openapi/leapfrog-power-revenue-analytics-openapi.yml
generated: '2026-07-27'
method: generated
source: openapi/leapfrog-power-revenue-analytics-openapi.yml, https://developer.leap.energy/docs/revenue-settlement-data
operations:
  - getPeriodicReports
  - getPeriodicReportVersions
  - getYearlyOverview
  - postMeterSearchPeriodicAggregationSearch
  - postCustomerSearchPeriodicAggregationSearch
  - postUtilitySearchPeriodicAggregationSearch
  - postLoadTypeSearchPeriodicAggregationSearch
  - postMarketGroupSearchPeriodicAggregationSearch
  - postMonthlyUnresponsiveMetersPerformance
---

# Pull Leap revenue, settlement and performance data

Settlement is the last stage of the meter lifecycle. This skill covers reading what a partner earned
and finding the meters that cost them money.

## Before you start

- Auth: `Authorization: Bearer <API_KEY>`, environment-scoped.
- Set `Accept` to `application/json` or `text/csv`. Any other value returns **415** with
  "Invalid value in the 'Accept' header".
- Revenue reporting is published for California and New York programs; New England and Texas partners
  receive settlement reports outside the API.

## Steps

1. **Monthly reports.** `GET /v1.1/revenue/periodic/reports` (`getPeriodicReports`) for a time frame.
   If you do not specify a version, the response is the **latest** version.
2. **Version discipline.** `GET /v1.1/revenue/periodic/versions` (`getPeriodicReportVersions`) lists
   every version for a report type and date range. Revenue restates — pin the version you booked and
   diff against later ones rather than silently overwriting your ledger.
3. **Annual view.** `GET /v1.1/revenue/yearly/{year}` (`getYearlyOverview`), optionally filtered by
   transmission region.
4. **Aggregate on the axis you need.** All five are POST searches over a designated month or months:
   - by meter — `postMeterSearchPeriodicAggregationSearch`
   - by customer — `postCustomerSearchPeriodicAggregationSearch`
   - by utility — `postUtilitySearchPeriodicAggregationSearch`
   - by load type — `postLoadTypeSearchPeriodicAggregationSearch`
   - by market group — `postMarketGroupSearchPeriodicAggregationSearch`
   Page with `page_token` / `page_size`, follow `next_page_token`.
5. **Find the leaks.** `POST /v1.1/performance/diagnosis/monthly/unresponsive/meters`
   (`postMonthlyUnresponsiveMetersPerformance`) returns the meters that did not respond in a given
   month. This endpoint left beta in July 2026 and is the intended basis for automated alerting.
6. **Go deeper on an event.** Meter-level baseline/dispatch/performance data and aggregated or
   meter-level intervals are available from the event-performance endpoints in the API reference
   (`listMeterLevelPerformance`, `listAggregatedIntervals`, `listMeterLevelIntervals`). Requests are
   capped at 1,000,000 data points — exceed it and you get a 400; reduce the day range or the meter
   count.

## Notes

- Interval data is gap-filled: when a utility does not send Leap some intervals, Leap predicts them
  from past data. Treat filled intervals accordingly in any settlement dispute.
- Errors use `{title, status, details[]}` where `details[]` is an array of `{message, description}` —
  slightly different from the other Leap services. See `errors/leapfrog-power-problem-types.yml`.
