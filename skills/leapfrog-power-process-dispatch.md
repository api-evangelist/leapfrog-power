---
name: Receive and process Leap grid dispatch events
description: Register a dispatch webhook receiver, send yourself test notifications, process meter and market-group dispatch timeslots safely, and fall back to polling.
api: openapi/leapfrog-power-dispatch-openapi.yml
generated: '2026-07-27'
method: generated
source: openapi/leapfrog-power-dispatch-openapi.yml, https://developer.leap.energy/docs/dispatch-automation-v2, https://developer.leap.energy/docs/webhook-push-notifications-v2
operations:
  - createOrUpdateMeterDispatchWebhook
  - getMeterDispatchWebhook
  - deleteMeterDispatchWebhook
  - triggerTestMeterDispatchNotification
  - triggerCommunicationTestMeterDispatch
  - createOrUpdateGroupDispatchWebhook
  - getGroupDispatchWebhook
  - deleteGroupDispatchWebhook
  - triggerTestGroupDispatchNotification
  - searchMeterDispatches
  - searchGroupDispatches
---

# Receive and process Leap grid dispatch events

Dispatch is the moment the platform earns. Push notifications are the recommended path because
real-time programs give minimal advance notice. Dispatch webhooks are configured **separately** from
the general webhook platform — different endpoints, different retry policy.

## Before you start

- Base URL for this API includes the version: `https://api.leap.energy/v2`.
- Auth: `Authorization: Bearer <API_KEY>`, environment-scoped.
- Your receiver must be HTTPS on port 443 (8443/8843 also accepted for the general platform), TLS 1.2
  or 1.3, and must return any **2xx within 10 seconds**.
- **Writing a webhook URL redirects live grid traffic.** `createOrUpdateMeterDispatchWebhook`
  overwrites the single meter-level receiver for the whole partner account. Read the current value
  first and confirm before overwriting.

## Steps

1. **Read the current receiver.** `GET /dispatch/meter/webhook` (`getMeterDispatchWebhook`) and, if
   you dispatch groups, `GET /dispatch/group/webhook` (`getGroupDispatchWebhook`).
2. **Register or update it.** `PUT /dispatch/meter/webhook`
   (`createOrUpdateMeterDispatchWebhook`) / `PUT /dispatch/group/webhook`
   (`createOrUpdateGroupDispatchWebhook`) with `{url, headers[]}`. Include an auth header of your own
   (e.g. `x-api-key`) in `headers[]` — Leap publishes no signature scheme, so that header is your only
   authenticity check. Header names arrive lower-cased.
3. **Send yourself a test.** `POST /dispatch/meter/webhook/integration_test`
   (`triggerTestMeterDispatchNotification`) or `POST /dispatch/group/webhook/integration_test`
   (`triggerTestGroupDispatchNotification`). You define the meter/group ids and the full `timeslots`
   array, so you can simulate multi-timeslot, multi-meter events exactly as a real event arrives.
   Test notifications travel over the same service and source IPs as real events; the only difference
   is `integration_test_notification: true`.
4. **Test the comms path only.** `POST /dispatch/meter/communication_test`
   (`triggerCommunicationTestMeterDispatch`) when you want to prove connectivity without a dispatch
   payload.
5. **Process the notification.** Acknowledge with 2xx *before* scheduling the events downstream.
   Retries run every minute for up to 50 minutes, so a slow acknowledgement produces duplicates —
   de-duplicate on `meter_event_id` (or `market_group_event_id`).
6. **Honour the timeslot fields.** `energy_kw` is the awarded curtailment/export in kW; **`null` means
   dispatch the maximum you can manage**. `nomination_kw` is the committed amount. Also read
   `cancelled`, `priority`, `is_voluntary`, `performance_compensation_cap`
   (`up-to-site-load` / `grid-exports-allowed`), `dispatch_event_types` and `programs`.
7. **Poll as a backstop.** `POST /dispatch/meter/search` (`searchMeterDispatches`) and
   `POST /dispatch/group/search` (`searchGroupDispatches`) return the same events on demand; use them
   for reconciliation after an outage of your receiver.
8. **Tear down.** `DELETE /dispatch/meter/webhook` / `DELETE /dispatch/group/webhook` remove the
   receiver — after which you receive nothing until it is set again.

## Notes

- Group dispatch requires your group membership to be current. Subscribe to
  `meter.enrollment.group.updated` on the general webhook platform.
- `partner_reference` is present on all dispatch notifications, meter and group level, so you can map
  a Leap `meter_id` to your own device id without a lookup.
- Errors here use a different envelope from the rest of Leap:
  `{code, message, error_code}` with codes like `DISPATCH_INVALID_INPUT` and
  `DISPATCH_NO_ACCESS_TO_RESOURCE` — see `errors/leapfrog-power-error-codes.yml`.
