# Idempotency

Send `Idempotency-Key` on mutations (POST, PATCH, DELETE). Optional in the contract; required by this skill.

A key is one logical request. Persist it before the HTTP call. Reuse it for the same method, path, and body. Do not mint a new key per transport retry.

```
POST /api/v1/invoice-drafts
X-API-Key: fc_test_...
Content-Type: application/json
Idempotency-Key: pedido-1001-borrador-v1
```

Stamp that draft with a **different** key (for example `pedido-1001-timbrado-v1`).

## Repeat behavior

| Situation | Result |
| --- | --- |
| Same key, finished request | Stored response is replayed |
| Same key, different body | `409` `idempotency_conflict` |
| Request still running | `409` `idempotency_in_progress` |
| No key | Independent operation |

Keys are up to 255 characters and do not auto-expire. Scope is API key + method + path. Rotating the API key starts a new idempotency history.

Use a new key for a new operation, or after you change the body following a validation error. An error response may already be stored under the original key.

## Stamp timeouts

The PAC may have issued the CFDI even if the client saw a timeout. `GET /invoices/{id}` with the id saved at draft creation.

FactuCat blocks concurrent stamps of the same draft. An ambiguous result stays blocked until the outcome is known. Waiting does not clear that lock. Do not open a second draft for the same sale.

If the document is `issued` with `satUuid`, persist that and download files. If the result is still unknown, keep the document id and `requestId` for support. Do not send the API key or full XML in a support log.

## Docs

[Idempotencia y reintentos](https://docs.factucat.com/guias/idempotencia/)
