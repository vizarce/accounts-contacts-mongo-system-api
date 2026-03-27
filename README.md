# accounts-contacts-mongo-system-api

MuleSoft RAML 1.0 specification for the **System Layer API** that exposes full CRUD operations and business actions against a MongoDB backend (`accounts` database on Atlas cluster `Cluster0`).

---

## Overview

This API is the **data-access layer** in a three-tier MuleSoft integration architecture:

```
Salesforce API  ←──  Process Layer API  ←──  accounts-contacts-mongo-system-api (this spec)
                                                          │
                                                   MongoDB Atlas
                                                   database: accounts
                                                   ├── accountapi  (existing)
                                                   ├── contacts    (created on first write)
                                                   └── leads       (created on first write)
```

---

## Resources

| Endpoint | Collection | Description |
|---|---|---|
| `GET/POST /accounts` | `accountapi` | List / create accounts |
| `GET/PUT/PATCH/DELETE /accounts/{id}` | `accountapi` | Read / replace / update / delete a single account |
| `GET /accounts/{id}/contacts` | `contacts` | List contacts for an account |
| `GET/POST /contacts` | `contacts` | List / create contacts |
| `GET/PUT/PATCH/DELETE /contacts/{id}` | `contacts` | Read / replace / update / delete a single contact |
| `GET/POST /leads` | `leads` | List / create leads |
| `GET/PUT/PATCH/DELETE /leads/{id}` | `leads` | Read / replace / update / delete a single lead |
| `POST /leads/{id}/convert` | `leads`, `accountapi`, `contacts` | Convert a lead into an account and/or contact |
| `GET /health` | — | Liveness / readiness probe (unauthenticated) |

---

## RAML Project Structure

```
src/main/resources/api/
├── accounts-contacts-mongo-system-api.raml   ← main specification
├── datatypes/
│   ├── common-types.raml     ← MongoId, ISODateTime, Address, PaginationMeta
│   ├── account-type.raml     ← Account, AccountRequest, AccountUpdateRequest, AccountsResponse
│   ├── contact-type.raml     ← Contact, ContactRequest, ContactUpdateRequest, ContactsResponse
│   ├── lead-type.raml        ← Lead, LeadRequest, LeadUpdateRequest, LeadConvertRequest/Response
│   └── error-type.raml       ← ErrorResponse, ErrorDetail
├── traits/
│   ├── pageable.raml         ← page / pageSize query parameters
│   ├── searchable.raml       ← q / name / email / date-range filters
│   ├── sortable.raml         ← sortBy / sortOrder query parameters
│   ├── cacheable.raml        ← Cache-Control / ETag / Last-Modified headers
│   └── error-handling.raml   ← Standard 400/401/403/404/409/500 error responses
├── securitySchemes/
│   ├── client-id-enforcement.raml  ← Anypoint client_id + client_secret headers
│   ├── basic-auth.raml             ← HTTP Basic Authentication
│   └── oauth2.raml                 ← OAuth 2.0 Bearer Token (client_credentials)
├── libraries/
│   └── commons.raml          ← Central library re-exporting all types, traits & schemes
└── examples/
    ├── accounts/             ← account-request, account-response, accounts-response
    ├── contacts/             ← contact-request, contact-response, contacts-response
    ├── leads/                ← lead-request, lead-response, leads-response, convert-*
    └── errors/               ← error-400, 401, 403, 404, 409, 500
```

---

## RAML Fragments Used

| Fragment type | Files |
|---|---|
| `#%RAML 1.0` (API root) | `accounts-contacts-mongo-system-api.raml` |
| `#%RAML 1.0 Library` | `libraries/commons.raml`, `datatypes/*.raml` |
| `#%RAML 1.0 Trait` | `traits/*.raml` |
| `#%RAML 1.0 SecurityScheme` | `securitySchemes/*.raml` |

---

## Security

All endpoints are protected by **Client ID Enforcement** by default.
OAuth 2.0 Bearer Tokens are additionally supported for process-layer consumers.
The `/health` endpoint is explicitly unsecured (`securedBy: []`).

---

## Salesforce Integration Notes

Each resource type (`Account`, `Contact`, `Lead`) exposes a `salesforceId` field
(Salesforce 18-char record ID) to support bidirectional synchronisation with Salesforce
implemented in the **Process Layer API** (`accounts-contacts-process-api`).

The `POST /leads/{id}/convert` action mirrors the Salesforce "Convert Lead" workflow
and returns the resulting account and contact IDs so the process layer can create the
corresponding records in Salesforce.

---

## Environments

| `{environment}` | Base URI |
|---|---|
| `dev` | `https://dev.accounts-contacts-mongo-sys-api.example.com/api/v1` |
| `sit` | `https://sit.accounts-contacts-mongo-sys-api.example.com/api/v1` |
| `uat` | `https://uat.accounts-contacts-mongo-sys-api.example.com/api/v1` |
| `prod` | `https://prod.accounts-contacts-mongo-sys-api.example.com/api/v1` |
