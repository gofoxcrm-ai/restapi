# REST API — documentation & delivery phases

This repo documents the **public integration REST surface** implemented in `gofox-server`. Phases below mirror what is live today versus planned extensions.

| Phase | Theme | Status |
|-------|--------|--------|
| **1** | API keys, auth, CRM REST (contacts / companies / leads / deals / tasks) | ✅ Implemented — documented in [README.md](./README.md#phase-1--crm-rest-api) |
| **2** | Content API (`content:read`), lead ingest, Content Hub publishing keys | ✅ Implemented — documented in [README.md](./README.md#phase-2--content-api--ingest) |
| **3** | Site plugin helpers, OpenAPI/SDKs, DELETE/extra resources | 🚧 Partial / planned — see [README.md](./README.md#phase-3--extensions) |

## Phase checklist

### Phase 1 — CRM REST (must-have)
- [x] Org API keys (`gfk_…`) with scopes
- [x] Auth via `Authorization: Bearer` or `X-Gofox-Api-Key`
- [x] `GET /api/v1/rest` discovery
- [x] Contacts / Companies / Leads / Deals / Tasks list + get + create + patch
- [x] Rate limit 120/min
- [x] Plan gate `api_access` (Prime+) on key creation
- [ ] Screenshots / quickstart video (`docs/assets/`)

### Phase 2 — Content & ingest
- [x] Content API `/api/v1/content/*` with `content:read`
- [x] Lead ingest `POST /api/v1/public/leads/ingest` (`ingest:write`)
- [x] Form-token ingest `POST /api/v1/public/ingest/:token`
- [x] CMS Website panel can mint `content:read` keys
- [ ] Postman collection / OpenAPI export

### Phase 3 — Extensions
- [x] Site helpers `/api/v1/integrations/site` (`integrations:read` / `ingest:write`)
- [ ] REST `DELETE` for CRM resources
- [ ] Tickets / invoices / activities on public REST
- [ ] Official SDKs (JS / Python)
- [ ] Full scope picker in Account Settings UI

## Source of truth

| Concern | Path |
|---------|------|
| CRM REST routes | `gofox-server/src/modules/public-rest/public-rest.routes.ts` |
| Content API | `gofox-server/src/modules/marketing-cms/content-api.routes.ts` |
| API keys | `gofox-server/src/modules/api-keys/` |
| Ingest | `gofox-server/src/modules/ingestion/` |
| Site helpers | `gofox-server/src/modules/integrations-site/` |

When you ship a new public route, update **README.md** and tick the relevant box here in the same PR.
