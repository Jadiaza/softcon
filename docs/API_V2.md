# SoftContable — API REST V2

Estado: **DISEÑADO**. No existe API todavía. Los contratos TypeScript están en `src/services/contracts.ts`.

## Convenciones
- Base: `/api/v1`. JSON, `snake_case` en el cable; el cliente HTTP convierte a `camelCase`.
- Encabezados: `Authorization: Bearer <token>`, `X-Company-Id: <uuid>`, `Idempotency-Key` en POST de escritura, `X-Request-Id` opcional.
- Listas: `?q=&page_size=50&cursor=&sort=-date&filter[status]=posted` → `{ data: [], next_cursor, total }`.
- Dinero como string decimal (`"1250000.00"`) para evitar pérdida de precisión.
- Fechas ISO 8601 (`2026-10-08`).

## Errores (RFC 7807)
```json
{ "type": "https://softcontable.co/errors/unbalanced-entry", "title": "Asiento descuadrado",
  "status": 422, "code": "ENTRY_UNBALANCED", "detail": "Débitos 100 ≠ créditos 90", "errors": {"lines": ["..."]}, "request_id": "..." }
```
| HTTP | code |
|---|---|
| 400 | VALIDATION_ERROR |
| 401 | UNAUTHENTICATED, MFA_REQUIRED |
| 403 | FORBIDDEN, COMPANY_ACCESS_DENIED |
| 404 | NOT_FOUND |
| 409 | CONFLICT, DUPLICATE, ALREADY_POSTED |
| 422 | ENTRY_UNBALANCED, PERIOD_CLOSED, ACCOUNT_NOT_POSTABLE, THIRD_PARTY_REQUIRED, INSUFFICIENT_STOCK |
| 429 | RATE_LIMITED |
| 502/503 | DIAN_UNAVAILABLE, UPSTREAM_ERROR |

## Rutas

| Recurso | Endpoints |
|---|---|
| auth | `POST /auth/login`, `POST /auth/mfa/verify`, `POST /auth/refresh`, `POST /auth/logout`, `GET /auth/me`, `GET/DELETE /auth/sessions/:id` |
| companies | `GET /companies` (las del usuario), `GET/PATCH /companies/:id`, `GET/POST /companies/:id/branches` |
| users | `GET/POST /users`, `PATCH /users/:id`, `GET /roles`, `POST /roles`, `PUT /roles/:id/permissions`, `PUT /users/:id/roles` |
| third-parties | `GET/POST /third-parties`, `GET/PATCH /third-parties/:id`, `GET /third-parties/:id/statement` |
| accounts | `GET /accounts?tree=1`, `POST /accounts`, `PATCH /accounts/:id`, `POST /accounts/import` |
| periods | `GET /periods`, `POST /periods/:id/close`, `POST /periods/:id/reopen` (authorize) |
| journal-entries | `GET/POST /journal-entries`, `GET/PATCH /journal-entries/:id` (solo draft), `POST /:id/post`, `POST /:id/reverse`, `POST /:id/void`, `DELETE /:id` (solo draft) |
| sales | `GET/POST /sales`, `GET /sales/:id`, `POST /sales/:id/issue`, `POST /sales/:id/credit-notes`; `GET/POST /quotes`, `POST /quotes/:id/convert` |
| purchases | `GET/POST /purchases`, `POST /purchases/:id/post`, `POST /support-documents` |
| receivables | `GET /receivables?aging=1`, `GET /receivables/:id` |
| payables | `GET /payables?aging=1` |
| inventory | `GET/POST /products`, `GET /warehouses`, `GET /inventory/balances`, `GET /inventory/kardex?product_id=`, `POST /inventory/adjustments`, `POST /inventory/transfers` |
| treasury | `GET/POST /payments`, `GET /bank-accounts`, `POST /bank-accounts/:id/statements` (import), `GET/POST /bank-reconciliations`, `POST /bank-reconciliations/:id/match` |
| taxes | `GET/POST /taxes`, `GET/POST /tax-rules`, `POST /taxes/calculate` |
| dian | `GET /dian/documents`, `GET /dian/documents/:id`, `POST /dian/documents/:id/retry`, `GET /dian/events`, `POST /dian/events` (RADIAN) |
| reports | `GET /reports/journal`, `/reports/ledger`, `/reports/auxiliary`, `/reports/trial-balance`, `/reports/balance-sheet`, `/reports/income-statement`, `/reports/cash-flow`, `/reports/dashboard`; `?format=pdf|xlsx` asíncrono → `202 { job_id }` |
| audit | `GET /audit-logs?entity=&entity_id=&user_id=&from=&to=` (solo lectura) |
| ai | `POST /ai/explain-variance`, `/ai/anomalies`, `/ai/classify-document`, `/ai/reconcile-suggestions` (requiere habilitación por empresa) |

## DTO de ejemplo — crear asiento
```json
POST /api/v1/journal-entries
{ "document_type": "CC", "date": "2026-10-08", "third_party_id": null, "description": "Causación honorarios",
  "lines": [
    { "account_code": "5110", "third_party_id": "…", "cost_center_id": "…", "debit": "3200000.00", "credit": "0" },
    { "account_code": "2335", "third_party_id": "…", "debit": "0", "credit": "3200000.00" } ] }
→ 201 { "id": "…", "number": 311, "status": "draft", … }
```

## Webhooks públicos (PENDIENTE)
`dian.document.accepted`, `dian.document.rejected`, `sale.issued`, `payment.received`, firmados con HMAC.