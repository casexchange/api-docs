Exploring the codebase to identify public-facing endpoints and their authentication mechanisms.


[4 tools called]


Searching for more details on specific endpoints mentioned:


[4 tools called]


Reviewing OAuth details and request/response schemas:


[3 tools called]


Creating the public API documentation:


[1 tool called]


Creating the public API documentation:


[1 tool called]


# CaseXchange Public API Documentation

## Table of Contents
1. [Introduction](#introduction)
2. [Authentication](#authentication)
3. [Base URL](#base-url)
4. [Endpoints](#endpoints)
5. [Error Handling](#error-handling)
6. [Rate Limiting](#rate-limiting)

---

## Introduction

This guide covers public-facing endpoints for third-party integrations. All endpoints require authentication via JWT Bearer tokens or OAuth 2.0.

---

## Authentication

### JWT Bearer Token Authentication

Most endpoints use JWT Bearer tokens in the `Authorization` header.

#### Obtaining a JWT Token

**Endpoint:** `POST /api/v1/auth/login`

**Request:**
```json
{
  "email": "user@example.com",
  "password": "your-password"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "firstName": "John",
      "lastName": "Doe",
      "role": "member",
      "firmId": "uuid",
      "isActive": true
    }
  }
}
```

**Note:** The response also sets HttpOnly cookies (`auth_token` and `refresh_token_id`) for web applications. For API clients, use the Bearer token method described below.

#### Using JWT Tokens

Include the token in the `Authorization` header:

```
Authorization: Bearer <your-jwt-token>
```

#### Token Refresh

**Endpoint:** `POST /api/v1/auth/refresh-session`

**Request:** No body required. The endpoint uses the `refresh_token_id` cookie if available, or you can include a refresh token in the request.

**Response:**
```json
{
  "success": true,
  "message": "Session refreshed successfully"
}
```

**Note:** Refresh tokens are typically stored as HttpOnly cookies. For server-to-server integrations, contact support for refresh token handling.

### OAuth 2.0 Authentication

For OAuth 2.0 integrations, CaseXchange supports the Authorization Code flow.

#### OAuth Discovery

**Endpoint:** `GET /.well-known/openid-configuration`

Returns OAuth/OIDC configuration:

```json
{
  "issuer": "https://api.casexchange.com/api/v1",
  "authorization_endpoint": "https://api.casexchange.com/api/v1/oauth/authorize",
  "token_endpoint": "https://api.casexchange.com/api/v1/oauth/token",
  "userinfo_endpoint": "https://api.casexchange.com/api/v1/oauth/userinfo",
  "revocation_endpoint": "https://api.casexchange.com/api/v1/oauth/revoke",
  "response_types_supported": ["code"],
  "grant_types_supported": ["authorization_code", "refresh_token"],
  "scopes_supported": ["openid", "email", "profile", "cases:read", "cases:write"]
}
```

#### OAuth Flow Steps

1. **Authorization Request**
   - Redirect user to: `GET /api/v1/oauth/authorize?client_id=<client_id>&redirect_uri=<redirect_uri>&response_type=code&scope=<scopes>&state=<state>`

2. **Token Exchange**
   - **Endpoint:** `POST /api/v1/oauth/token`
   - **Request:**
     ```json
     {
       "grant_type": "authorization_code",
       "code": "<authorization_code>",
       "redirect_uri": "<redirect_uri>",
       "client_id": "<client_id>",
       "client_secret": "<client_secret>"
     }
     ```
   - **Response:**
     ```json
     {
       "access_token": "<jwt-access-token>",
       "token_type": "Bearer",
       "expires_in": 900,
       "refresh_token": "<refresh-token>",
       "scope": "cases:read cases:write"
     }
     ```

3. **Refresh Token**
   - **Endpoint:** `POST /api/v1/oauth/token`
   - **Request:**
     ```json
     {
       "grant_type": "refresh_token",
       "refresh_token": "<refresh-token>",
       "client_id": "<client_id>",
       "client_secret": "<client_secret>"
     }
     ```

4. **Revoke Token**
   - **Endpoint:** `POST /api/v1/oauth/revoke`
   - **Request:**
     ```json
     {
       "token": "<token-to-revoke>",
       "token_type_hint": "access_token"
     }
     ```

#### OAuth Scopes

- `openid` - OpenID Connect authentication
- `email` - Access to user email
- `profile` - Access to user profile information
- `cases:read` - Read access to cases
- `cases:write` - Write access to cases (create, update)

---

## Base URL

- **Production:** `https://api.casexchange.com/api/v1`
- **QA/UAT:** Contact support for environment-specific URLs

All endpoints are prefixed with `/api/v1`.

---

## Endpoints

### 1. Create Case

Create a new case referral.

**Endpoint:** `POST /api/v1/cases`

**Authentication:** Required (JWT Bearer Token or OAuth)

**Request Body:**
```json
{
  "title": "Personal Injury Case",
  "description": "Case description",
  "caseType": "personal_injury",
  "jurisdiction": "CA",
  "clientFirstName": "John",
  "clientLastName": "Doe",
  "clientEmail": "john.doe@example.com",
  "clientPhone": "+1234567890",
  "referentFirmId": "uuid-of-receiving-firm",
  "notes": "Additional notes",
  "salesforceRecordId": "a0X1234567890ABC",
  "salesforceObjectType": "Case"
}
```

**Response:** `201 Created`
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "Personal Injury Case",
    "referenceNumber": "FIRM-0001",
    "status": "sent",
    "caseType": "personal_injury",
    "jurisdiction": "CA",
    "clientFirstName": "John",
    "clientLastName": "Doe",
    "referringFirm": {
      "id": "uuid",
      "name": "Your Firm Name"
    },
    "referentFirm": {
      "id": "uuid",
      "name": "Receiving Firm Name"
    },
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T00:00:00Z"
  }
}
```

**Required Fields:**
- `title`
- `caseType`
- `jurisdiction`
- `clientFirstName`
- `clientLastName`

**Optional Fields:**
- `description`
- `clientEmail`
- `clientPhone`
- `referentFirmId` (if omitted, case is created as `draft`)
- `notes`
- `salesforceRecordId` (must be provided with `salesforceObjectType`)
- `salesforceObjectType` (must be provided with `salesforceRecordId`)

---

### 2. Update Case

Update an existing case by ID.

**Endpoint:** `PUT /api/v1/cases/{id}`

**Authentication:** Required (JWT Bearer Token or OAuth)

**Path Parameters:**
- `id` (UUID) - Case ID

**Request Body:**
```json
{
  "title": "Updated Case Title",
  "description": "Updated description",
  "notes": "Updated notes",
  "clientEmail": "newemail@example.com",
  "clientPhone": "+1987654321"
}
```

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "Updated Case Title",
    "status": "under_evaluation",
    "updatedAt": "2024-01-01T12:00:00Z"
  }
}
```

**Note:** Permissions depend on:
- Referring firm: Can update before acknowledgment; limited fields after
- Receiving firm: Can update notes after acknowledgment
- System Admin: Can update any field

---

### 3. Bulk Update Cases

Update multiple cases via CSV file.

**Endpoint:** `POST /api/v1/cases/bulk-update`

**Authentication:** Required (JWT Bearer Token or OAuth - Admin or Firm Admin role)

**Request:** `multipart/form-data`
- `file` (CSV file) - Required

**CSV Format:**
```csv
Reference Number,Status,Salesforce Id,Salesforce Object,Case Status ID,Settlement Amount,Attorney Fees,Closure Reason
FIRM-0001,under_evaluation,a0X1234567890ABC,Case,P-55443,,
FIRM-0002,signed,,,P-55444,100000.00,33333.33,
```

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "fileName": "bulk-update.csv",
    "totalRows": 2,
    "successful": 2,
    "failed": 0,
    "created": 0,
    "updated": 2,
    "successfulCases": [
      {
        "id": "uuid",
        "title": "Case Title",
        "status": "under_evaluation",
        "referenceNumber": "FIRM-0001",
        "operation": "update"
      }
    ],
    "failedRows": []
  }
}
```

**Supported Update Fields:**
- `Reference Number` (required) - Identifies the case
- `Status` - Case status
- `Salesforce Id` & `Salesforce Object` - Must be provided together
- `Case Status ID` - Case Status link ID
- `Settlement Amount` - Numeric
- `Attorney Fees` - Numeric
- `Closure Reason` - Text

**Note:** Only active referrals (non-terminal statuses) can be updated. Users can only update cases owned by their firm (Admins can update any case).

---

### 4. Re-Refer Case

Re-refer a case to a different firm (typically after rejection).

**Endpoint:** `POST /api/v1/cases/{id}/re-refer`

**Authentication:** Required (JWT Bearer Token or OAuth)

**Path Parameters:**
- `id` (UUID) - Base case ID

**Request Body:**
```json
{
  "referentFirmId": "uuid-of-new-receiving-firm",
  "notes": "Optional notes for re-referral"
}
```

**Response:** `201 Created`
```json
{
  "success": true,
  "message": "Case successfully re-referred",
  "data": {
    "id": "uuid",
    "baseCaseId": "uuid",
    "status": "sent",
    "referringFirmId": "uuid",
    "referentFirmId": "uuid",
    "sequenceNumber": 2,
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```

**Business Rules:**
- Only the case owner (referring firm) can re-refer
- An active relationship must exist between firms
- Creates a new referral record while maintaining the base case
- Previous referral history is preserved

---

### 5. Get Firms

Retrieve a list of firms.

**Endpoint:** `GET /api/v1/firms`

**Authentication:** Required (JWT Bearer Token or OAuth)

**Query Parameters:**
- `page` (integer, default: 1) - Page number
- `limit` (integer, default: 20, max: 100) - Items per page
- `search` (string) - Search by firm name
- `isInternal` (boolean) - Filter by internal status (System Admin only)

**Response:** `200 OK`
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "Law Firm Name",
      "description": "Firm description",
      "contactEmail": "contact@firm.com",
      "contactPhone": "+1234567890",
      "website": "https://firm.com",
      "canSendReferrals": true,
      "caseStatusSubscriber": false,
      "specialties": ["personal_injury", "medical_malpractice"],
      "jurisdictions": ["CA", "NY"],
      "address": {
        "street": "123 Main St",
        "city": "Los Angeles",
        "state": "CA",
        "zipCode": "90001",
        "country": "USA"
      },
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-01T00:00:00Z"
    }
  ],
  "pagination": {
    "total": 100,
    "page": 1,
    "limit": 20,
    "totalPages": 5
  }
}
```

**Additional Firm Endpoints:**

- **Get Firm by ID:** `GET /api/v1/firms/{id}`
- **Firm Directory:** `GET /api/v1/firms/directory` - Browse customer firms with filtering options

---

### 6. Get Jurisdictions

Retrieve a list of available jurisdictions.

**Endpoint:** `GET /api/v1/firms/jurisdictions/list`

**Authentication:** Required (JWT Bearer Token or OAuth)

**Response:** `200 OK`
```json
{
  "success": true,
  "data": ["CA", "NY", "TX", "FL", "IL"]
}
```

**Detailed Jurisdictions:**

**Endpoint:** `GET /api/v1/firms/jurisdictions`

**Response:** `200 OK`
```json
{
  "success": true,
  "data": [
    {
      "code": "CA",
      "name": "California",
      "type": "state"
    },
    {
      "code": "NY",
      "name": "New York",
      "type": "state"
    }
  ]
}
```

---

### 7. Get Case Types

Retrieve a list of available case types.

**Endpoint:** `GET /api/v1/firms/case-types`

**Authentication:** Required (JWT Bearer Token or OAuth)

**Response:** `200 OK`
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "personal_injury",
      "description": "Personal injury cases"
    },
    {
      "id": "uuid",
      "name": "medical_malpractice",
      "description": "Medical malpractice cases"
    }
  ]
}
```

**Alternative Endpoint (Specialties):**

**Endpoint:** `GET /api/v1/firms/specialties/list`

**Response:** `200 OK`
```json
{
  "success": true,
  "data": ["personal_injury", "medical_malpractice", "product_liability"]
}
```

---

### 8. Additional Public Endpoints

#### Get Case by ID

**Endpoint:** `GET /api/v1/cases/{id}`

**Authentication:** Required (JWT Bearer Token or OAuth)

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "Case Title",
    "referenceNumber": "FIRM-0001",
    "status": "under_evaluation",
    "caseType": "personal_injury",
    "jurisdiction": "CA",
    "clientFirstName": "John",
    "clientLastName": "Doe",
    "referringFirm": { "id": "uuid", "name": "Firm Name" },
    "referentFirm": { "id": "uuid", "name": "Receiving Firm" },
    "statusUpdates": [],
    "documents": [],
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T00:00:00Z"
  }
}
```

#### List Cases

**Endpoint:** `GET /api/v1/cases`

**Authentication:** Required (JWT Bearer Token or OAuth)

**Query Parameters:**
- `page` (integer, default: 1)
- `limit` (integer, default: 20)
- `search` (string) - Search by title or client name
- `status` (string) - Filter by status (comma-separated: `under_evaluation,investigating`)
- `caseType` (string) - Filter by case type
- `jurisdiction` (string) - Filter by jurisdiction
- `view` (string) - `sent`, `received`, or `all`
- `fromDate` (datetime) - Filter cases from date
- `toDate` (datetime) - Filter cases to date

**Response:** `200 OK`
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "title": "Case Title",
      "referenceNumber": "FIRM-0001",
      "status": "under_evaluation",
      "caseType": "personal_injury",
      "jurisdiction": "CA",
      "clientName": "John Doe",
      "referringFirm": { "id": "uuid", "name": "Firm Name" },
      "referentFirm": { "id": "uuid", "name": "Receiving Firm" },
      "createdAt": "2024-01-01T00:00:00Z"
    }
  ],
  "pagination": {
    "total": 50,
    "page": 1,
    "limit": 20,
    "totalPages": 3
  }
}
```

#### Evaluate Routing Rules

**Endpoint:** `POST /api/v1/routing/evaluate-api`

**Authentication:** Required (JWT Bearer Token or OAuth)

**Request Body:**
```json
{
  "severity": 3,
  "caseType": "personal_injury",
  "jurisdiction": "CA"
}
```

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "firm": {
      "id": "uuid",
      "name": "Routed Firm Name",
      "contactEmail": "contact@firm.com",
      "contactPhone": "+1234567890",
      "isExternalContact": false
    },
    "isActive": true,
    "rule": {
      "id": "uuid",
      "name": "Rule Name",
      "description": "Rule description"
    }
  }
}
```

**Note:** Routing evaluation is specific to the firm authorization that called this endpoint (determined from the JWT token).

#### Get Current User Profile

**Endpoint:** `GET /api/v1/auth/me`

**Authentication:** Required (JWT Bearer Token or OAuth)

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "member",
    "firmId": "uuid",
    "firm": {
      "id": "uuid",
      "name": "Firm Name"
    },
    "isActive": true,
    "lastLogin": "2024-01-01T00:00:00Z",
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```

---

## Error Handling

All endpoints return errors in a consistent format:

**Error Response Format:**
```json
{
  "success": false,
  "error": {
    "code": "error_code",
    "message": "Human-readable error message",
    "details": []
  }
}
```

### Common HTTP Status Codes

- `200 OK` - Success
- `201 Created` - Resource created successfully
- `400 Bad Request` - Invalid request parameters or validation error
- `401 Unauthorized` - Authentication required or token invalid/expired
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Resource not found
- `409 Conflict` - Resource conflict (e.g., duplicate email)
- `429 Too Many Requests` - Rate limit exceeded
- `500 Internal Server Error` - Server error

### Common Error Codes

- `no_token` - No authentication token provided
- `token_expired` - JWT token has expired
- `token_invalid` - JWT token is invalid
- `rate_limited_email` - Too many login attempts for email
- `rate_limited_ip` - Too many login attempts from IP
- `validation_error` - Request validation failed
- `resource_not_found` - Requested resource not found
- `permission_denied` - Insufficient permissions

---

## Rate Limiting

- Login endpoints: Rate limited per email and IP address
- Other endpoints: Standard rate limiting applies
- Rate limit headers:
  - `X-RateLimit-Limit` - Maximum requests allowed
  - `X-RateLimit-Remaining` - Remaining requests
  - `X-RateLimit-Reset` - Time when limit resets (Unix timestamp)

When rate limited, you'll receive a `429 Too Many Requests` response with details in the error message.

---

## Support

For API access, OAuth client registration, or questions:
- Email: help@casexchange.com
- Documentation: [Link to full API documentation]

---

## Version History

- **v0.1.0** - Initial public API release

---

**Last Updated:** January 2024

This documentation covers the public-facing endpoints for third-party integrations. For internal endpoints or additional features, contact support.

Please implement this into our docs