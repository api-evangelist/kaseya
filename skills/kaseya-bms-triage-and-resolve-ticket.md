---
name: Triage and resolve a Kaseya BMS service desk ticket
description: Find an open BMS ticket, read its activity and notes, assign it to a
  technician, log time against it and resolve it — using only operations that exist in
  the published BMS OpenAPI 3.0.1 document.
api: openapi/kaseya-bms-openapi-original.json
generated: '2026-08-01'
method: generated
operations:
  - SearchTickets
  - GetTicket
  - GetTicketActivities
  - GetTicketNotes
  - PostTicketNote
  - AssignTicket
  - PostTicketTimeEntry
  - ResolveTicket
  - GetTicketSLAInfo
---

# Triage and resolve a Kaseya BMS ticket

## Before you start

- Base URL is region-specific: `https://api.bms.kaseya.com` (US),
  `https://api.bmsemea.kaseya.com` (EMEA), `https://api.bmsapac.kaseya.com` (APAC),
  `https://api.vorexlogin.com` (Vorex). Use the one matching the tenant.
- Authenticate first: `POST /v2/security/authenticate` with the BMS username,
  password, company name and server URL. It returns a JWT plus a refresh token. Send
  the JWT as `Authorization: Bearer <token>` on every call — the spec applies
  `bearerAuth` at the document root, so all 435 operations require it.
- **Every BMS response is wrapped.** The body is
  `{ "Success": bool, "Error": ErrorInfo, "Result": ..., "TotalRecords": int }`.
  A failure can arrive with HTTP 200 and `Success: false` — always check `Success`
  before reading `Result`. The `Error` object is `{Code, Message, Details, StackStrace,
  SubErrors}` — note the vendor's spelling of `StackStrace`.
- Rate limit: 1500 requests per hour **per endpoint**. No rate-limit headers are
  returned, so throttle yourself.
- **There is no idempotency key.** Never blind-retry a POST. If a create times out,
  search for the record before re-sending.

## Steps

1. **Find candidate tickets** — `SearchTickets` (`POST /v2/servicedesk/tickets/search`).
   The body is `GetTicketsInputDto`: `PageSize`, `PageNumber`, `Sort`, `Exclude` and a
   typed `Filter` (`TicketFilterDto`). Read `TotalRecords` from the envelope to page.
   Use `GetTicketsListSummary` if you only need summary rows.

2. **Read the ticket** — `GetTicket` (`GET /v2/servicedesk/tickets/{ticketId}`).

3. **Read the history before acting** — `GetTicketActivities`
   (`GET /v2/servicedesk/tickets/{ticketId}/activities`) and `GetTicketNotes`
   (`GET /v2/servicedesk/tickets/{ticketId}/notes`). Check `GetTicketSLAInfo`
   (`GET /v2/servicedesk/tickets/{ticketId}/slainfo`) to see whether the ticket is
   near breach before you re-prioritise it.

4. **Record what you found** — `PostTicketNote`
   (`POST /v2/servicedesk/tickets/{ticketId}/notes`). Post the note *before* assigning,
   so the technician receives context with the assignment.

5. **Assign** — `AssignTicket`
   (`POST /v2/servicedesk/tickets/{ticketId}/assignticket`).

6. **Log time** — `PostTicketTimeEntry`
   (`POST /v2/servicedesk/tickets/{ticketId}/timelogs`). Time entries are billable
   records; do not create one speculatively and do not retry on an ambiguous failure —
   read back with `GetTicketTimeEntry` first.

7. **Resolve** — `ResolveTicket`
   (`POST /v2/servicedesk/tickets/{ticketId}/resolve`).

## Do not

- Do not call `DeleteTicket`, `DeleteTickets` or `MergeTickets` without explicit human
  confirmation — they are destructive and irreversible over the API.
- Do not treat HTTP 200 as success.
- Do not assume an error code vocabulary; BMS publishes no error-code registry, so
  surface `Error.Code` and `Error.Message` verbatim to the operator.

## Related artifacts

- `conventions/kaseya-conventions.yml` — envelope, pagination and filter grammar
- `errors/kaseya-problem-types.yml` — the ErrorInfo shape
- `rate-limits/kaseya-rate-limits.yml`
- `authentication/kaseya-authentication.yml`
