---
name: Upload partner interval data to Leap
description: Submit interval meter data for meters where Leap does not receive utility data directly, then monitor ingestion jobs and fix row-level validation errors.
api: openapi/leapfrog-power-interval-data-upload-openapi.json
generated: '2026-07-27'
method: generated
source: openapi/leapfrog-power-interval-data-upload-openapi.json, https://developer.leap.energy/docs/event-performance-interval-data, https://developer.leap.energy/docs/upload-interval-data-sftp
operations:
  - uploadIntervalData
  - listIntervalDataUploads
  - listIntervalDataUploadErrors
  - intervalDataFilePreview
  - intervalDataPresignedUrl
  - searchMissingIntervalData
---

# Upload partner interval data to Leap

Where Leap does not receive interval data from the utility, the partner supplies it. Settlement and
performance depend on this data being complete and correctly aligned, so ingestion is validated hard
and reports errors per row and column.

## Before you start

- Auth: `Authorization: Bearer <API_KEY>`, environment-scoped. Note that this definition declares the
  bearer scheme but no top-level `security` requirement — the endpoints are protected regardless.
- Intervals are 15 minutes. `interval_start_time_utc` and `interval_end_time_utc` must align to
  15-minute boundaries, must be UTC, and must not be in the future.
- Energy columns: supply `energy_net_kwh`, or the pair `energy_generated_kwh` + `energy_consumed_kwh`,
  or all of them — but never conflicting values.

## Steps

1. **Find the gaps first.** `POST /v1.1/missing-interval-data/search` (`searchMissingIntervalData`)
   returns the meters and periods Leap is missing, so you upload only what is needed.
2. **Upload.** `POST /v1.1/interval-data` (`uploadIntervalData`). Alternatively drop the file on
   Leap's SFTP server — both paths land in the same ingestion pipeline and appear in the same job
   list.
3. **Watch the job.** `GET /v1.1/jobs/interval-data-uploads` (`listIntervalDataUploads`) lists uploaded
   files regardless of whether they came through SFTP or the Partner Portal, with status.
4. **Read the errors.** `GET /v1.1/jobs/interval-data-uploads/{job_id}`
   (`listIntervalDataUploadErrors`) returns a per-column `error_summary[]` of `{column, code, count}`
   and the individual `errors[]` of `{line, column, code, message}`, plus an
   `error_csv_file_path` for the full reject file.
5. **Fix by code.** The codes are precise — `MTR_0003` (meter_id does not point to a known meter),
   `STM_0003` (start time not aligned to a 15-minute boundary or in the future), `DTR_0003` (range is
   not 15 minutes), `NRG_0001` (no valid set of energy columns), `NRG_0003` (energy columns conflict),
   `REG_0003` (region is not a valid state). Full list:
   `errors/leapfrog-power-error-codes.yml`.
6. **Inspect the source.** `GET /v1.1/jobs/interval-data-uploads/file-preview/{job_id}`
   (`intervalDataFilePreview`) shows what Leap parsed;
   `GET /v1.1/jobs/interval-data-uploads/presigned-url/{job_id}` (`intervalDataPresignedUrl`) returns
   a **302** redirect to a presigned location for the file — follow the redirect rather than treating
   it as an error.
7. **Re-upload the corrected rows** and repeat from step 3 until the error summary is empty.

## Notes

- There is no idempotency key. Re-uploading the same file re-ingests it — reconcile on job id.
- Uploaded interval data feeds the event-performance and revenue surfaces; see
  `skills/leapfrog-power-revenue-reporting.md`.
- The 4xx/5xx bodies on this service are declared as untyped objects in the specification, so program
  against the status code plus the per-row error codes rather than a body shape.
