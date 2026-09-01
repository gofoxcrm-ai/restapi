# Gofox Public REST API

EngageBay-style **REST API** for Gofox CRM — build integrations with HTTPS + JSON using org API keys (`gfk_…`).

> Sibling products: [Tracking Code API](https://github.com/gofoxcrm-ai/trackingcodeapi) · [Webhooks](https://github.com/gofoxcrm-ai/webhooks) · [SSO](https://github.com/gofoxcrm-ai/sso)

**Phased docs:** see [PHASES.md](./PHASES.md) for the full checklist. This README is organized by implementation phase.

---

## Status overview

| Phase | Area | Status |
|-------|------|--------|
| 1 | Auth + CRM REST | ✅ Live |
| 2 | Content API + lead ingest | ✅ Live |
| 3 | Site helpers / OpenAPI / DELETE | 🚧 Partial |
| Media | Screenshots / video | 🚧 Placeholders in `docs/assets/` |

---

## Base URLs

```
CRM REST:     https://api.gofox.io/api/v1/rest
Content API:  https://api.gofox.io/api/v1/content
Public ingest:https://api.gofox.io/api/v1/public
Local:        http://localhost:4000/api/v1/...
```

Tenant UI APIs remain under `/api/v1/tenant/*` (session JWT). This repo documents **public integration** surfaces only.

---

# Phase 1 — CRM REST API

Must-have for external CRM integrations.

## Authentication

1. In Gofox: **Account Settings → API & tracking code** → create an API key  
   (also: **Marketing → API keys**, or Content Hub → **Website** for `content:read`)
2. Choose scopes (see table below). The raw key is shown **once**.
3. Call with either header:

```http
Authorization: Bearer gfk_xxxxxxxx
```

or

```http
X-Gofox-Api-Key: gfk_xxxxxxxx
```

### Scopes

| Scope | Purpose |
|-------|---------|
| `crm:read` | List & get CRM resources |
| `crm:write` | Create & update CRM resources (implies read) |
| `leads:read` | List & get leads (legacy; prefer `crm:read`) |
| `ingest:write` | Lead ingest endpoint (Phase 2) |
| `content:read` | Published Content Hub API (Phase 2) |
| `integrations:read` | Site plugin helpers (Phase 3) |

Plan entitlement: creating keys requires **`api_access`** (Prime+).

<!-- SCREENSHOT: docs/assets/api-keys-create.png -->
![API keys UI (placeholder)](docs/assets/api-keys-create.png)

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

[Video walkthrough (placeholder)](docs/assets/rest-api-quickstart.mp4)

## Response envelope

Success: `{ "data": … }`  
Errors: `{ "error": { "code": "…", "message": "…" } }`

## Resources

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

> There is **no** public REST `DELETE` for these resources yet (Phase 3).

## Pagination & filters

List endpoints accept the same query params as the tenant UI APIs (`page`, `limit`, `search`, and resource-specific filters). Invalid params return `400` with a Zod validation message.

## Rate limits

**120 requests / minute / client IP** on `/api/v1/rest`. Exceeding returns `429` with `RATE_LIMITED`.

---

# Phase 2 — Content API & ingest

## Content API

Published marketing content from **Content Hub** for external websites.

```
Base: https://api.gofox.io/api/v1/content
Auth: content:read
Rate: 180 req/min
Cache: Cache-Control: public, max-age=60, stale-while-revalidate=300
```

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/` | Discovery / index |
| `GET` | `/blog` | List published blogs |
| `GET` | `/blog/:slug` | Get blog by slug |
| `GET` | `/pages` | List published pages |
| `GET` | `/pages/:slug` | Get page |
| `GET` | `/faqs` | List FAQs |
| `GET` | `/faqs/:slug` | Get FAQ |
| `GET` | `/articles` | List (filterable by `contentType`) |
| `GET` | `/articles/:slug` | Get article |
| `GET` | `/categories` | Categories |
| `GET` | `/categories/:slug` | Category |
| `GET` | `/authors` | Authors |
| `GET` | `/authors/:authorId` | Author |

**Article fields (typical):** `id`, `contentType`, `title`, `slug`, `excerpt`, `content` (HTML body), `featuredImage`, `author`, `category`, `tags`, `publishedAt`, `updatedAt`, `seo`, optional `cta`.

Only **published + public** marketing content is returned (no drafts / Help Center KB).

```bash
curl -s "https://api.gofox.io/api/v1/content/blog?page=1&limit=20" \
  -H "X-Gofox-Api-Key: gfk_xxxxxxxx" | jq
```

Mint a `content:read` key from **Content Hub → Website** or API keys settings.

## Lead ingest

| Method | Path | Auth | Rate |
|--------|------|------|------|
| `POST` | `/api/v1/public/leads/ingest` | `ingest:write` | 60/min |
| `POST` | `/api/v1/public/ingest/:token` | form public token | 60/min |

Accepts snake_case or camelCase fields, UTM params, and optional `visitor_id` / `visitorId` to stitch browsing history (see [trackingcodeapi](https://github.com/gofoxcrm-ai/trackingcodeapi)).

```bash
curl -s -X POST "https://api.gofox.io/api/v1/public/leads/ingest" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{"email":"ada@example.com","firstName":"Ada","source":"zapier"}' | jq
```

---

# Phase 3 — Extensions

## Site integration helpers (live)

```
Base: /api/v1/integrations/site
Auth: integrations:read or ingest:write
```

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/status` | Org slug, tracker URL template, connection hints |
| `GET` | `/forms` | Public forms list for plugins |
| `GET` | `/landing-pages` | Landing pages list for plugins |

## Planned / not yet on public REST

| Item | Notes |
|------|--------|
| `DELETE` CRM resources | Use tenant UI / session APIs today |
| Tickets, invoices, activities | Not on `/api/v1/rest` |
| OpenAPI / SDKs | Placeholders — generate from routes when ready |
| Full scope picker in UI | Some UIs default a subset of scopes |

---

## Adding new endpoints

Implementation lives in the Gofox monolith:

1. Add route in the appropriate `gofox-server` module (`public-rest`, `content-api`, etc.)
2. Reuse existing domain `*.service.ts` (do not duplicate CRM logic)
3. Guard with the correct scope middleware
4. Document the row in this README **and** tick [PHASES.md](./PHASES.md) in the same PR
5. Ship — routes are live when the server deploys

---

## Related

- Outbound events: [webhooks](https://github.com/gofoxcrm-ai/webhooks)
- Visitor tracking: [trackingcodeapi](https://github.com/gofoxcrm-ai/trackingcodeapi)
- Enterprise login: [sso](https://github.com/gofoxcrm-ai/sso)

---

## Media placeholders

| File | Purpose |
|------|---------|
| `docs/assets/api-keys-create.png` | Create API key UI |
| `docs/assets/rest-contacts-list.png` | Example JSON / Postman |
| `docs/assets/content-api-blog.png` | Content API blog list |
| `docs/assets/rest-api-quickstart.mp4` | Quickstart video |

---

## License

Documentation © Gofox. API access subject to your Gofox subscription and Terms of Service.
