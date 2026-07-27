---
name: Subscribe to Leap webhook platform events
description: Create, test, update and retire subscriptions on Leap's general webhook platform so a partner gets Connect-session, meter and enrollment events instead of polling.
api: openapi/leapfrog-power-webhooks-openapi.yml
generated: '2026-07-27'
method: generated
source: openapi/leapfrog-power-webhooks-openapi.yml, https://developer.leap.energy/docs/webhook-setup, https://developer.leap.energy/docs/retry-mechanism
operations:
  - listWebhooks
  - createWebhook
  - updateWebhook
  - deleteWebhook
  - testWebhook
---

# Subscribe to Leap webhook platform events

The general webhook platform (GA since November 2025, path `/v1.1/webhooks`) covers Connect-session
and meter/enrollment events. Dispatch notifications are **not** here — they are configured through the
Dispatch API (see `skills/leapfrog-power-process-dispatch.md`).

## Before you start

- Stand up an endpoint that accepts a JSON POST on TCP 443, 8443 or 8843 with TLS 1.2 or 1.3 and
  returns any 2xx within 10 seconds. `https://webhook.site` is the receiver Leap's own docs recommend
  for a first pass.
- Auth on the management endpoints: `Authorization: Bearer <API_KEY>`, environment-scoped.

## Event types

- `connect_session.updated` — `CREATED` / `UPDATED` / `CLOSED` / `COMPLETED`
- `connect_session.authorization_updated` — `INITIATED` / `COMPLETED` / `FAILED`, with `utility_name`
- `meter.created`
- `meter.enrollment.global-status.updated`
- `meter.enrollment.required-actions.updated`
- `meter.enrollment.participation-status.updated`
- `meter.enrollment.group.updated`

Every delivery uses the same envelope: `{webhook_event_id, payload: {webhook_event_type, ...}}`.

## Steps

1. **Inventory what exists.** `GET /v1.1/webhooks` (`listWebhooks`) — this endpoint is unpaginated.
2. **Create the subscription.** `POST /v1.1/webhooks` (`createWebhook`) with the receiver URL, any
   custom headers, and one or more event types. Include your own auth header in `headers[]`; Leap
   publishes no HMAC signing, so that header is the authenticity check.
3. **Test it.** `POST /v1.1/webhooks/{webhook_id}/test` (`testWebhook`) sends a test event to the
   configured `receiver_url` — do not wait for a real event to validate the plumbing.
4. **Update carefully.** `PUT /v1.1/webhooks/{webhook_id}` (`updateWebhook`) **overwrites all existing
   settings** for that webhook id. Read the current record first and send the full desired state.
5. **Retire.** `DELETE /v1.1/webhooks/{webhook_id}` (`deleteWebhook`).

## Delivery semantics

- Retry ladder: immediate, 1m, 2m, 4m, 8m, 16m, 29m, 1h, 2h, 4h — 10 attempts over 8 hours, then Leap
  stops. (Dispatch webhooks are different: every 1 minute for up to 50 minutes.)
- Calls time out at 10 seconds. A 2xx returned at 11 seconds is still counted as a failure and retried.
- Duplicates are expected when an acknowledgement is lost — de-duplicate on `webhook_event_id`.
- Full event schemas: `asyncapi/leapfrog-power-events-asyncapi.yml`.

## Notes

- Subscriptions can also be managed in the Partner Portal at
  `/account?settings=webhooks` in either environment.
- Leap recommends these events **instead of** polling the Connect API, and as the recommended
  migration off the deprecated Meters API v1 (sunset 2026-10-31).
- Errors on these endpoints use a third envelope, `{error, message}`.
