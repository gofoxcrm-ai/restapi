# Gofox Public REST API

EngageBay-style **REST API** for Gofox CRM — create integrations on top of contacts, companies, leads, deals, and tasks using HTTPS + JSON.

> Sibling products: [Tracking Code API](https://github.com/gofoxcrm-ai/trackingcodeapi) · [Webhooks](https://github.com/gofoxcrm-ai/webhooks) · [SSO](https://github.com/gofoxcrm-ai/sso)  
> Marketing overview pattern: [EngageBay API](https://www.engagebay.com/api)

---

## Status

| Area | Status |
|------|--------|
| Auth (API keys `gfk_…`) | ✅ Live in `gofox-server` |
| Contacts / Companies / Leads / Deals / Tasks | ✅ Live under `/api/v1/rest` |
| OpenAPI / SDKs | 🚧 Placeholders below |
| Screenshots / video walkthrough | 🚧 Placeholders below |

---

## Base URL

```
Production: https://api.gofox.io/api/v1/rest
Local:      http://localhost:4000/api/v1/rest
```

Tenant UI APIs remain under `/api/v1/tenant/*` (session JWT). This repo documents the **public integration REST** surface.

---

## Authentication

1. In Gofox: **Account Settings → API & tracking code** → create an API key.
2. Include scopes:
   - `crm:read` — list & get
   - `crm:write` — create & update (also allows read)
3. Call with either header:

```http
Authorization: Bearer gfk_xxxxxxxx
```

or

```http
x-gofox-api-key: gfk_xxxxxxxx
```

> The raw key is shown **once** at creation. Store it in your secrets manager — never in frontend code or git.

<!-- SCREENSHOT: docs/assets/api-keys-create.png
     Placeholder — add screenshot of Account Settings → API keys create dialog showing crm:read / crm:write scopes.
-->

![API keys UI (placeholder)](docs/assets/api-keys-create.png)

---

## Quick start

```bash
export API=https://api.gofox.io/api/v1/rest
export KEY=gfk_your_key_here

# Ping
curl -s "$API/" -H "Authorization: Bearer $KEY" | jq

# List contacts
curl -s "$API/contacts?page=1&limit=20" -H "Authorization: Bearer $KEY" | jq

# Create contact
curl -s -X POST "$API/contacts" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{"firstName":"Ada","lastName":"Lovelace","email":"ada@example.com"}' | jq
```

<!-- VIDEO: docs/assets/rest-api-quickstart.mp4
     Placeholder — 60–90s screencast: create key → curl list contacts → create contact in UI.
-->

[Video walkthrough (placeholder)](docs/assets/rest-api-quickstart.mp4)

---

## Resources

All responses are JSON: `{ "data": … }`. Errors: `{ "error": { "code": "…", "message": "…" } }`.

### Meta

| Method | Path | Scope | Description |
|--------|------|-------|-------------|
| `GET` | `/` | any valid key | Ping, org id, scopes, resource list |

### Contacts

| Method | Path | Scope |
|--------|------|-------|
| `GET` | `/contacts` | `crm:read` or `crm:write` |
| `GET` | `/contacts/:contactId` | `crm:read` or `crm:write` |
| `POST` | `/contacts` | `crm:write` |
| `PATCH` | `/contacts/:contactId` | `crm:write` |

**Create body (minimal):**

```json
{
  "firstName": "Ada",
  "lastName": "Lovelace",
  "email": "ada@example.com"
}
```

### Companies

| Method | Path | Scope |
|--------|------|-------|
| `GET` | `/companies` | `crm:read` or `crm:write` |
| `GET` | `/companies/:companyId` | `crm:read` or `crm:write` |
| `POST` | `/companies` | `crm:write` |
| `PATCH` | `/companies/:companyId` | `crm:write` |

### Leads

| Method | Path | Scope |
|--------|------|-------|
| `GET` | `/leads` | `crm:read`, `crm:write`, or `leads:read` |
| `GET` | `/leads/:leadId` | same |
| `POST` | `/leads` | `crm:write` |
| `PATCH` | `/leads/:leadId` | `crm:write` |

### Deals

| Method | Path | Scope |
|--------|------|-------|
| `GET` | `/deals` | `crm:read` or `crm:write` |
| `GET` | `/deals/:dealId` | `crm:read` or `crm:write` |
| `POST` | `/deals` | `crm:write` |
| `PATCH` | `/deals/:dealId` | `crm:write` |

### Tasks

| Method | Path | Scope |
|--------|------|-------|
| `GET` | `/tasks` | `crm:read` or `crm:write` |
| `GET` | `/tasks/:taskId` | `crm:read` or `crm:write` |
| `POST` | `/tasks` | `crm:write` |
| `PATCH` | `/tasks/:taskId` | `crm:write` |

---

## Pagination & filters

List endpoints accept the same query params as the tenant UI APIs (e.g. `page`, `limit`, `search`, pipeline/stage filters where applicable). Invalid query params return `400` with a Zod validation message.

---

## Rate limits

Default: **120 requests / minute / API key** on `/api/v1/rest`. Exceeding returns `429` with `RATE_LIMITED`.

---

## Adding new endpoints

Implementation lives in the Gofox monolith:

1. Add route in `gofox-server/src/modules/public-rest/public-rest.routes.ts`
2. Reuse existing `*.service.ts` domain functions (do not duplicate CRM logic)
3. Guard with `requireCrmRead` / `requireCrmWrite` (or `requireAnyApiKeyScope`)
4. Document the row in this README **in the same PR**
5. Ship — new routes are live as soon as the server deploys

Plan entitlement: API keys require plan feature **`api_access`** (Prime+).

---

## Related

- Lead ingest (forms / Zapier-style): `POST /api/v1/public/leads/ingest` — see also `ingest:write` scope
- Outbound events: [webhooks](https://github.com/gofoxcrm-ai/webhooks)
- Visitor tracking: [trackingcodeapi](https://github.com/gofoxcrm-ai/trackingcodeapi)

---

## Media placeholders

Place files under `docs/assets/` (gitignored binaries optional — link externally if large):

| File | Purpose |
|------|---------|
| `docs/assets/api-keys-create.png` | Create API key UI |
| `docs/assets/rest-contacts-list.png` | Example JSON / Postman |
| `docs/assets/rest-api-quickstart.mp4` | Quickstart video |

---

## License

Documentation © Gofox. API access subject to your Gofox subscription and Terms of Service.
