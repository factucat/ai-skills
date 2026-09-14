# Errors

Success:

```json
{ "data": {}, "meta": { "requestId": "..." } }
```

Failure:

```json
{
  "error": {
    "code": "missing_scope",
    "message": "La API key no tiene permisos para esta operación.",
    "requestId": "identificador-de-la-solicitud",
    "retryable": false
  }
}
```

Also read header `X-Request-Id`. Keep that id when investigating.

## HTTP

| Status | Check |
| --- | --- |
| 400 | JSON, params, or business/SAT validation |
| 401 | Missing, invalid, revoked, or wrong-environment key |
| 403 | Scopes, plan, or outstanding payment |
| 404 | Wrong id, or resource in another account/environment. After stamp, `/invoice-drafts/{id}` is 404; use `/invoices/{id}` |
| 409 | Idempotency conflict or stamp still in progress |
| 429 | Rate limited; wait `Retry-After`. Do not invent a numeric quota |
| 500 | Internal; keep `requestId` and reconcile the document before retrying a stamp |

## Integration failures

| Symptom | What to do |
| --- | --- |
| 401 on `/me` | Key, revocation, and URL/prefix pairing ([ambientes](https://docs.factucat.com/guias/ambientes/)) |
| 403 `missing_scope` | Recreate the key with the needed scopes |
| Duplicate customer RFC | `GET /customers?query={rfc}` and reuse `id` |
| Stamp 400 (receiver, CSD, catalogs, totals) | Fix the same draft; new idempotency key for the corrected call. SAT checks run at stamp, not at `/me` ([emitir-factura](https://docs.factucat.com/guias/emitir-factura/)) |
| Stamp timeout or 409 | `GET /invoices/{id}` with the saved id ([idempotencia](https://docs.factucat.com/guias/idempotencia/)) |
| Need XML/PDF again | Repeat the GET; do not stamp again |

Full field and status detail: [Errores y paginación](https://docs.factucat.com/guias/respuestas/) · [Referencia API](https://docs.factucat.com/api/) · [primeros-pasos](https://docs.factucat.com/guias/primeros-pasos/)
