---
name: Suggest and manage Leap market nominations
description: Offer capacity into a market by suggesting nominations per meter or in bulk, track Leap's review, and reconcile the accepted nominations that drive program enrollment.
api: openapi/leapfrog-power-nominations-openapi.yml
generated: '2026-07-27'
method: generated
source: openapi/leapfrog-power-nominations-openapi.yml, https://developer.leap.energy/docs/bidding-introduction
operations:
  - getMeterNominationSuggestions
  - postMeterNominationSuggestions
  - postNominationSuggestions
  - deleteMeterNominationSuggestion
  - searchNominationSuggestions
  - searchNominations
---

# Suggest and manage Leap market nominations

A nomination is a commitment in kW made to a market for an asset. Partners do not set nominations
directly — they *suggest* them, and Leap reviews each suggestion and may approve, modify or deny it.
This is a revenue-consequential write path; treat every POST as a market action.

## Before you start

- Auth: `Authorization: Bearer <API_KEY>`, environment-scoped.
- No idempotency key exists. A retried suggestion POST is a second suggestion.
- All datetimes are UTC.
- Responses on the search endpoints honour the `Accept` header — `application/json` or `text/csv`.
  Anything else returns **415**.

## Steps

1. **Read what Leap suggests first.** `GET /v2/meters/{meter_id}/nominations/suggestions`
   (`getMeterNominationSuggestions`) returns suggested nomination details for that meter for each
   applicable program and time period. Start from these rather than inventing values.
2. **Suggest for one meter.** `POST /v2/meters/{meter_id}/nominations/suggestions`
   (`postMeterNominationSuggestions`).
3. **Suggest in bulk.** `POST /v2/meters/nominations/suggestions` (`postNominationSuggestions`) for
   many meters at once. The array is capped at 10,000 items per request.
4. **Understand overlap semantics.** A newer suggestion that completely eclipses an earlier one
   deletes the earlier nomination. Partial overlaps are **not** automatically adjusted or
   de-duplicated — you must clean those up yourself.
5. **Clean up.** `DELETE /v2/meters/{meter_id}/nominations/suggestions/{nomination_suggestion_id}`
   (`deleteMeterNominationSuggestion`). A suggestion can only be deleted while it is still in
   `RECEIVED` status; once Leap has reviewed it, it cannot be deleted. Deleting stale suggestions
   speeds up Leap's review of the ones that matter.
6. **Track review outcomes.** `POST /v2/meters/nominations/suggestions/search`
   (`searchNominationSuggestions`) across all meters — published examples show statuses `RECEIVED`
   and `MODIFIED`, with `created_at` and `created_by`.
7. **Reconcile the real commitments.** `POST /v2/meters/nominations/search` (`searchNominations`)
   returns the nominations in kW that will actually be used for program enrollment, filtered by meter
   ids and a date range, with a `source` of `PARTNER` or `LEAP`.

## Downstream

The accepted `nomination_kw` shows up again on every dispatch timeslot beside the awarded
`energy_kw`, so a dispatch handler can compare committed against awarded capacity — see
`skills/leapfrog-power-process-dispatch.md`.

## Notes

- Bids (the Pay-as-Bid auction path, `submitBid` / `searchBids` / `searchStandingBids` in the API
  reference) are a separate surface from nomination suggestions and are not covered by the eight
  harvested OpenAPI definitions. Initial bids must be placed at least 24 hours before the start of the
  trading day, and a bid is cancelled by submitting an empty curve.
- Errors use the `{title, status, details}` envelope; 404 on unknown meter or suggestion ids.
