# Kaseya

Kaseya is a Miami-based provider of IT and security management software for managed
service providers (MSPs) and internal IT teams, delivering its portfolio through the
Kaseya 365 and IT Complete platforms.

- Website: https://www.kaseya.com/
- Helpdesk / documentation: https://helpdesk.kaseya.com/hc/en-gb
- Status: https://status.kaseya.com/
- Trust center: https://www.kaseya.com/trust-center/

## APIs profiled

| API | Contract | Operations |
|---|---|---|
| Kaseya BMS API 2.0 | OpenAPI 3.0.1 | 435 |
| Datto Autotask PSA REST API | Swagger 2.0 | 3,009 |
| Datto RMM API v2 | OpenAPI 3.1.0 | 65 |
| IT Glue API | docs only (JSON:API) | — |
| Kaseya VSA 9 REST API | docs only (tenant-hosted) | — |
| Kaseya VSA 10 API | docs only (tenant-hosted) | — |
| myITprocess API | Swagger UI, spec auth-gated | — |

## Artifacts

`openapi/` `overlays/` `authentication/` `conventions/` `errors/` `lifecycle/`
`changelog/` `conformance/` `data-model/` `rate-limits/` `asyncapi/` `mcp/` `skills/`
`packages/` `security/` `well-known/` `llms/`

No A2A agent card, no AsyncAPI document, no first-party MCP server and no
`/.well-known/` discovery document was found on any Kaseya host — those artifacts are
deliberately absent rather than stubbed.
