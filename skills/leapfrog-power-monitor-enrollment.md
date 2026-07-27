---
name: Monitor Leap meter enrollment and clear required actions
description: Keep a partner's meter inventory in sync with Leap — read meter and enrollment state, detect ineligible meters, and work the required-actions queue that blocks program participation.
api: openapi/leapfrog-power-meter-enrollment-openapi.yml
generated: '2026-07-27'
method: generated
source: openapi/leapfrog-power-meter-enrollment-openapi.yml, openapi/leapfrog-power-meter-details-openapi.yml, https://developer.leap.energy/docs/sync-meter-inventory-v2
operations:
  - getMeterEnrollment
  - searchMeterEnrollments
  - getMeterDetails
  - searchMeterDetails
---

# Monitor Leap meter enrollment and clear required actions

A meter only earns when it is enrolled and eligible. This skill keeps the partner's picture of that
state current and turns Leap's blocking items into work.

## Before you start

- Auth: `Authorization: Bearer <API_KEY>`, environment-scoped (staging key on production → 403).
- Prefer webhook events over polling. Leap's own guidance is that the meter events replace
  inventory polling — see `asyncapi/leapfrog-power-events-asyncapi.yml`.

## Steps

1. **Full sync (initial load or reconciliation).** `POST /v2/meters/enrollments/search`
   (`searchMeterEnrollments`) with a filter body. Page with `page_token` / `page_size` and follow
   `next_page_token` until it is empty. The response carries current enrollment status, participation
   preferences, associated programs and required actions per meter.
2. **Attach the inventory detail.** `POST /v2/meters/search` (`searchMeterDetails`) returns customer,
   utility, site and device information for the same meters. Join on `meter_id`, or on your own
   `partner_reference` if you are writing into your own store.
3. **Single-meter drill-down.** `GET /v2/meters/{meter_id}/enrollments` (`getMeterEnrollment`) and
   `GET /v2/meters/{meter_id}` (`getMeterDetails`) for one meter — use these when handling a webhook
   event, not the search endpoints.
4. **Work the required actions.** When `global_enrollment_status` is `INELIGIBLE`, read
   `required_actions.partner[]`. Each entry carries `action_type`, `action_description`, `action_code`
   (e.g. `SERV_0005` — missing configuration fields), `program_identifier`, `delivery_period` and an
   `enrollment_deadline_date_time`. Treat the deadline as hard: past it, the meter misses the delivery
   period.
5. **Re-register after a fix.** For CAISO utility meters that were ineligible and have since been
   fixed (conflicting program removed, smart meter installed), submit a market-registration refresh —
   `postRefreshMeterMarketRegistrationRequest` in the API reference. Registration can take up to two
   weeks in CAISO.
6. **Stay in sync incrementally.** Subscribe to `meter.enrollment.global-status.updated`,
   `meter.enrollment.required-actions.updated`, `meter.enrollment.participation-status.updated` and
   `meter.enrollment.group.updated`. A required-actions event with an empty `required_actions` object
   means everything is resolved.

## Notes

- Participation state is `PARTICIPATE`, `IDLE` or `DISENROLL`, date-ranged. Idle periods respect the
  meter's local timezone; almost every other datetime in the Leap API is UTC.
- If your platform dispatches pre-configured groups, `meter.enrollment.group.updated` is mandatory
  reading — dispatching a stale group hurts performance and revenue.
- Error envelope: `{title, status, details}`; 404 on an unknown `meter_id`.
