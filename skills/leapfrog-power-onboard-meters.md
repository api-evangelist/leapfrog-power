---
name: Onboard meters into the Leap platform
description: Create meters on Leap in bulk or individually, poll the resulting job to completion, and resolve per-row validation errors before the meters enter market registration.
api: openapi/leapfrog-power-create-meters-openapi.yml
generated: '2026-07-27'
method: generated
source: openapi/leapfrog-power-create-meters-openapi.yml, https://developer.leap.energy/docs/partner-created-meters
operations:
  - createMeterBatchJob
  - getMeterBatchJob
  - listMeterBatchJobs
  - listMeterBatchJobsFilters
  - createSingleMeter
  - listProvisionalAssets
  - listProvisionalAssetsFilters
---

# Onboard meters into the Leap platform

Use this when a partner has meters or devices to register with Leap so they can be enrolled into
market and utility demand-response programs. This is the "Direct Enroll" path — the alternative is
the customer-authorized Leap Connect flow, which produces meters without any API call.

## Before you start

- Base URL: `https://api.leap.energy` (production) or `https://api.staging.leap.energy` (staging).
- Auth: `Authorization: Bearer <API_KEY>`. Keys are environment-scoped; using a staging key against
  production returns **403**, not 401.
- There is **no idempotency key**. A retried `createMeterBatchJob` creates a second job. Always poll
  before retrying.
- Set `partner_reference` on every meter. It is the only join key back to your own systems, and it is
  echoed on meter details, webhook events and dispatch notifications.

## Steps

1. **Submit the batch.** `POST /v1.1/jobs/meters` (`createMeterBatchJob`). The endpoint accepts CSV or
   JSON. It returns a job id — treat the response as an acknowledgement, not a success.
2. **Poll the job.** `GET /v1.1/jobs/meters/{job_id}` (`getMeterBatchJob`) until it reaches a terminal
   status. The response carries the per-asset upload status and the meter ids created for successfully
   uploaded assets.
3. **Read the errors.** Failed rows come back as `CreateMeterErrorResponse` — `errors[]` of
   `{field, description, suggestions[]}` — and as `ValidationError` with `csv_row` (CSV uploads) or
   `json_index` (JSON payloads) plus `problem_field`, so you can point the fix at the exact input row.
   Fix and resubmit only the failed rows.
4. **Single meters.** For one-off registrations use `POST /v1.1/meters` (`createSingleMeter`) instead
   of a batch job. Same validation shape, synchronous response.
5. **Track history.** `GET /v1.1/jobs/meters` (`listMeterBatchJobs`) lists prior jobs; the filter
   values available for that listing come from `GET /v1.1/jobs/meter-job-filters`
   (`listMeterBatchJobsFilters`).
6. **Check provisional assets.** `GET /v1.1/provisional-assets` (`listProvisionalAssets`) shows assets
   submitted but not yet promoted to meters, with filters from `listProvisionalAssetsFilters`.
7. **Watch for the meter to appear.** Subscribe to the `meter.created` webhook event rather than
   polling — it fires for both partner-created and Connect-created meters and carries `meter_id`,
   `partner_reference`, `transmission_region` and `meter_type`.

## After creation

Creating a meter does not enroll it. Market registration is a Leap-side process that can take up to
two weeks in CAISO. Follow with the enrollment skill
(`skills/leapfrog-power-monitor-enrollment.md`) to watch `global_enrollment_status` and clear any
`required_actions` before the meter can earn.

## Conventions and errors

- Pagination on the list endpoints is cursor-based: send `page_token` and `page_size`, follow
  `next_page_token` until it is absent.
- Error envelope is `{title, status, details}` — not RFC 9457. See
  `errors/leapfrog-power-problem-types.yml`.
- Full cross-cutting rules: `conventions/leapfrog-power-conventions.yml`.
