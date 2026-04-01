# CaseXchange Public API Documentation

## Overview

CaseXchange provides two API layers:

1. **Public API** (`/api/public/v1`) — API key authentication with tiered access (READ_ONLY, STANDARD, FULL). Designed for third-party integrations.
2. **Internal API** (`/api/v1`) — JWT bearer token authentication. Used by the CaseXchange web application and internal services.

This documentation primarily covers the **Public API**. The internal API is documented in the OpenAPI spec (`api-reference/openapi.yml`).

## Authentication

### Public API — API Key

All Public API requests require an `X-API-Key` header with a tiered key:

| Tier | Prefix | Permissions | Best for |
|------|--------|-------------|----------|
| READ_ONLY | `cxp_ro_...` | GET endpoints only | Dashboards, read integrations |
| STANDARD | `cxp_std_...` | Read + create/update referrals | Case intake, referral workflows |
| FULL | `cxp_full_...` | All operations including delete, bulk import, routing rules, firm profile updates | Admin integrations |

```
X-API-Key: cxp_std_your_key_here
```

Keys are created via the internal API (`POST /api/v1/public-api-keys`) by firm admins. The plaintext key is only returned once at creation time.

### Internal API — JWT

See the [JWT Guide](/auth/jwt) in the docs site for details on JWT session authentication.

## Base URLs

- **Public API (Production):** `https://api.casexchange.com/api/public/v1`
- **Internal API (Production):** `https://api.casexchange.com/api/v1`
- **QA/UAT:** Contact support for environment-specific URLs

## Public API Endpoint Groups

- **Health** — `GET /api/public/v1/health` (no auth required)
- **Reference Data** — Case types, jurisdictions, counties
- **Firms** — Directory, own firm profile, available firms, firm by ID
- **Sent Cases** — Create, list, detail, refer, update (sender perspective)
- **Received Referrals** — List, detail, update, status transitions (receiver perspective)
- **Documents** — Upload, list, download, delete
- **Users** — List, invite, update, deactivate
- **Routing Rules** — List, create, update, delete
- **Analytics** — Dashboard summary, case analytics
- **Notifications** — Preferences, email recipients
- **Medical Referrals** — CRUD operations
- **Bulk Import** — CSV import and bulk update
- **Key Management** — Create, list, usage, revoke (via internal API)

## Postman Collection

A pre-built Postman collection and production environment are available in the `postman/` directory. See the quickstart guide for setup instructions.

## Response Format

**Success:**
```json
{
  "data": { },
  "meta": { "requestId": "...", "timestamp": "..." },
  "pagination": { "total": 50, "page": 1, "limit": 20, "totalPages": 3 }
}
```

**Error:**
```json
{
  "error": {
    "code": "validation_error",
    "message": "Human-readable message",
    "details": []
  },
  "meta": { "requestId": "...", "timestamp": "..." }
}
```

## Error Handling

### Common HTTP Status Codes

- `200 OK` / `201 Created` / `204 No Content` — Success
- `400 Bad Request` — Validation error or invalid state transition
- `401 Unauthorized` — Missing or invalid API key / token
- `403 Forbidden` — Insufficient tier or permissions
- `404 Not Found` — Resource not found
- `409 Conflict` — Duplicate or conflicting state
- `429 Too Many Requests` — Rate limit exceeded

## Rate Limiting

Public API endpoints are rate limited per API key. Rate limit headers are included on every response:

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Maximum requests allowed in the current window |
| `X-RateLimit-Remaining` | Remaining requests in the current window |
| `X-RateLimit-Reset` | Unix timestamp when the window resets |

When you exceed the limit, you will receive a `429 Too Many Requests` response. Implement exponential backoff and respect the `X-RateLimit-Reset` header before retrying.

## Support

For API access, key provisioning, or questions: help@casexchange.com