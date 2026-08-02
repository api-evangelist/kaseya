---
name: Take a Datto RMM alert from open to remediated
description: Pull open Datto RMM alerts, locate the affected device and site, read its
  audit and patch state, run a remediation component as a quick job, read the job's
  stdout/stderr and resolve the alert — pacing against the live request-rate endpoint.
api: openapi/kaseya-datto-rmm-openapi-original.json
generated: '2026-08-01'
method: generated
operations:
  - getUserAccountOpenAlerts
  - getAlert
  - getByUid
  - getSite
  - getDeviceAudit
  - getDevicePatches
  - createQuickJob
  - getJobResults
  - getStdOut
  - getStdErr
  - resolveAlert
  - get
---

# Datto RMM: alert to remediation

## Before you start

- Base URL is regional: `https://<region>-api.centrastage.net/api` where region is one
  of `pinotage`, `merlot`, `concord`, `vidal`, `zinfandel`, `syrah`. Use the tenant's
  region.
- Authenticate with OAuth 2.0: exchange the API Key and API Secret Key at
  `POST /auth/oauth/token` for an access token. Tokens last 100 hours; an expired token
  returns `401`.
- **Pace yourself against the real limits**: 600 read and 100 write requests per rolling
  60 seconds. Exceeding returns `429`; persistent violation returns `403` **and blocks
  your IP for five minutes**. Because a `403` is also the ordinary authorization
  failure, do not interpret a `403` as "no permission" without checking rate state.
- `get` (`GET /v2/system/request_rate`) returns the live `RateStatusResponse` — sliding
  window size, account read/write limits, current counts and a per-operation write
  limit/count map. Call it when you are about to run a burst. This is the only runtime
  rate-limit signal anywhere in the Kaseya portfolio.
- **There is no idempotency key.** Datto RMM does return `409` on concurrent write
  ("Request aborted due to concurrent write access to this record"), so on `409`
  re-read the record and re-apply — but a retried `createQuickJob` after a timeout can
  still run the component twice.

## Steps

1. **List open alerts** — `getUserAccountOpenAlerts`
   (`GET /v2/account/alerts/open`). Page with `page` and `max`; call
   `getPaginationConfigurations` (`GET /v2/system/pagination`) if you need the server's
   configured page bounds.

2. **Read one alert** — `getAlert` (`GET /v2/alert/{alertUid}`). The alert carries a
   typed context object (disk usage, antivirus, event log, endpoint security, patch,
   SNMP, …) — read it rather than parsing the alert message text.

3. **Locate the device and site** — `getByUid` (`GET /v3/device/{deviceUid}`) and
   `getSite` (`GET /v2/site/{siteUid}`). Prefer the `/v3` device operations; the `/v2`
   equivalents (`getByUid_1`, `getSiteDevices_1`) are the older shape.

4. **Establish state before acting** — `getDeviceAudit`
   (`GET /v2/audit/device/{deviceUid}`) for hardware/OS facts and `getDevicePatches`
   (`GET /v2/device/{deviceUid}/patches`) for patch state. For non-generic devices use
   `getPrinterAudit` or `getEsxiHostAudit`; calling the wrong one returns `400` with
   "not of class …".

5. **Remediate — HUMAN CONFIRMATION REQUIRED.** `createQuickJob`
   (`PUT /v2/device/{deviceUid}/quickjob`) executes a component on a live managed
   endpoint. This is the highest-consequence operation in the Kaseya portfolio. Never
   run it autonomously: present the device, the site, the component and its variables
   to a human and require explicit approval.

6. **Read the result** — `getJobResults`
   (`GET /v2/job/{jobUid}/results/{deviceUid}`), then `getStdOut` and `getStdErr` for
   the component's actual output. Do not resolve the alert until the job output
   confirms the fix.

7. **Resolve** — `resolveAlert` (`POST /v2/alert/{alertUid}/resolve`).

## Do not

- Do not call `muteAlert` or `unmuteAlert` — their own summaries state that alerts can
  no longer be muted as of release 8.9.0, even though the spec does not flag them
  deprecated.
- Do not call `resetApiKeys` (`POST /v2/user/resetApiKeys`) — it rotates the
  authenticated user's API credentials and will break every other integration using
  them.
- Do not call `moveDevice` (`PUT /v2/device/{deviceUid}/site/{siteUid}`) without human
  confirmation; it re-parents a device between customers.

## Related artifacts

- `rate-limits/kaseya-rate-limits.yml` — the request_rate introspection endpoint
- `errors/kaseya-problem-types.yml` — the ErrorResponse shape and the 400/403/409 cases
- `conventions/kaseya-conventions.yml`
- `authentication/kaseya-authentication.yml`
