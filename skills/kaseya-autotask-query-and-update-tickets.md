---
name: Query and update Datto Autotask PSA tickets
description: Discover an Autotask entity's real fields, build a valid JSON filter,
  page through results with the server-supplied page URLs, and create or patch tickets
  and time entries — grounded in operationIds that exist in the published Autotask
  Swagger 2.0 document.
api: openapi/kaseya-autotask-psa-openapi-original.json
generated: '2026-08-01'
method: generated
operations:
  - Tickets_QueryEntityInformation
  - Tickets_QueryFieldDefinitions
  - Tickets_QueryUserDefinedFieldDefinitions
  - Tickets_Query
  - Tickets_QueryCount
  - Tickets_QueryItem
  - Tickets_CreateEntity
  - Tickets_PatchEntity
  - Companies_Query
  - Contacts_Query
  - TimeEntries_CreateEntity
---

# Query and update Autotask PSA tickets

## Before you start

- **Resolve the zone first.** Autotask is multi-homed across at least 14 hosts
  (`webservices1`–`webservices19.autotask.net`, covering pre-release, limited release,
  America East/West, UK, ANZ, German, EU1 and Spanish). The base URL is
  `https://<zone>/ATServicesRest/V1.0/`. Calling the wrong zone fails.
- **Authentication is three static headers on every request**: `UserName` (the
  API-only user's e-mail), `Secret` (its password) and `ApiIntegrationCode` (the
  integration tracking identifier). Optionally `ImpersonationResourceId`. These are
  *not* declared as securityDefinitions in the Swagger document — they are ordinary
  required header parameters on all 3,009 operations.
- The API-only user has **full system administrator access**. There are no scopes.
  Treat every write as privileged.
- **Rate limit: 10,000 requests per rolling hour per database, counting every
  integration on that tenant.** Latency is applied progressively as you approach it and
  exceeding it suspends API access for the whole tenant — including other vendors'
  integrations. Be a good citizen: page efficiently and use `Tickets_QueryCount` before
  pulling a large result set.
- **There is no idempotency key.** A retried `Tickets_CreateEntity` creates a second
  ticket.

## Steps

1. **Introspect the entity before you filter it.** Autotask installs are heavily
   customised. Call `Tickets_QueryFieldDefinitions`
   (`GET /V1.0/Tickets/entityInformation/fields`) and
   `Tickets_QueryUserDefinedFieldDefinitions`
   (`GET /V1.0/Tickets/entityInformation/userDefinedFields`) to learn which fields and
   picklist values this tenant actually has. `Tickets_QueryEntityInformation`
   (`GET /V1.0/Tickets/entityInformation`) reports which verbs the entity supports.

2. **Size the result set** — `Tickets_QueryCount`
   (`POST /V1.0/Tickets/query/count`) with the same filter you intend to run.

3. **Query** — `Tickets_Query` (`POST /V1.0/Tickets/query`). The body is a
   `QueryModel`:

   - `filter`: an array of `Filter` objects `{op, field, value, udf, items}`. Nest
     compound conditions through `items`. Set `udf` to true to target a user-defined
     field.
   - `includeFields`: array of field names to return (use it — ticket rows carry 76
     fields).
   - `maxRecords`: page size.

   Prefer the POST form over `Tickets_UrlParameterQuery`
   (`GET /V1.0/Tickets/query?search=...`), which URL-encodes the whole filter tree into
   a query string.

4. **Page by following the server's URLs.** The response is
   `{ "items": [...], "pageDetails": { "count", "requestCount", "prevPageUrl",
   "nextPageUrl" } }`. Follow `nextPageUrl` verbatim — do not construct offsets.

5. **Read one ticket** — `Tickets_QueryItem` (`GET /V1.0/Tickets/{id}`).

6. **Create** — `Tickets_CreateEntity` (`POST /V1.0/Tickets`). Resolve
   `companyID` with `Companies_Query` and `contactID` with `Contacts_Query` first;
   Autotask will reject unknown ids rather than create the parent.

7. **Update** — `Tickets_PatchEntity` (`PATCH /V1.0/Tickets`) for a partial change.
   `Tickets_UpdateEntity` (`PUT /V1.0/Tickets`) replaces the entity and will null
   fields you omit — prefer PATCH.

8. **Log time** — `TimeEntries_CreateEntity` (`POST /V1.0/TimeEntries`).

## Error handling

The contract documents only `200`, `401 Unauthorized` and `403 Forbidden`, with no
error schema. Anything else you receive is undocumented. Surface the raw body to the
operator rather than pattern-matching on it, and treat a sudden run of failures as a
possible threshold suspension.

## Do not

- Do not generate one tool per operation. The 3,009 operations are ~190 entities times
  the same 12-operation CRUD template; parameterise over the entity name instead.
- Do not run unbounded queries. `filter` is required and an over-broad one will eat the
  tenant's hourly threshold.
- Do not write to webhook configuration entities (`TicketWebhooks`, `CompanyWebhooks`,
  …) without explicit human confirmation — they change what other integrations receive.

## Related artifacts

- `conventions/kaseya-conventions.yml` — the filter grammar and paging model
- `rate-limits/kaseya-rate-limits.yml`
- `asyncapi/kaseya-webhooks.yml` — the webhook control plane
- `authentication/kaseya-authentication.yml`
