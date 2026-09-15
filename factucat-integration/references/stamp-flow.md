# Minimal PUE stamp flow

Income CFDI, paid in full at issuance (`paymentMethod: "PUE"`). Run against sandbox until the integration is proven.

Default:

```bash
export FACTUCAT_API_URL='https://sandbox.factucat.com/api/v1'
export OPERACION_ID='pedido-1001'
```

Key scopes: `customer:read`, `customer:write`, `invoice:read`, `invoice:write`, `invoice:stamp`.

Receiver RFC, legal name, postal code, and tax regime must match SAT. Creating a customer does not validate those fields with SAT.

## 1. Customer

Reuse a stored `customerId` when you have one. Search first if you only have an RFC: `GET /customers?query={rfc}`.

```bash
curl --fail-with-body --max-time 60 --request POST \
  --url "$FACTUCAT_API_URL/customers" \
  --header "X-API-Key: $FACTUCAT_API_KEY" \
  --header "Idempotency-Key: $OPERACION_ID-cliente-v1" \
  --header "Content-Type: application/json" \
  --data '{
    "legalName": "EMPRESA DE EJEMPLO",
    "rfc": "AAA010101AAA",
    "postalCode": "06000",
    "taxRegime": "601",
    "sendInvoices": false,
    "invoiceLanguage": "es",
    "currency": "MXN"
  }'
```

Save `data.id`. A duplicate RFC in the same account is a conflict: look up that customer and reuse its id. Replace the example receiver with SAT-registered data before a real stamp; the values above are shape-only.

## 2. Draft

```bash
curl --fail-with-body --max-time 60 --request POST \
  --url "$FACTUCAT_API_URL/invoice-drafts" \
  --header "X-API-Key: $FACTUCAT_API_KEY" \
  --header "Idempotency-Key: $OPERACION_ID-borrador-v1" \
  --header "Content-Type: application/json" \
  --data '{"currency":"MXN","cfdiUse":"G03","exportacion":"01","customerId":"CUSTOMER_ID"}'
```

Save `data.id` next to the sale **before** stamp. FactuCat copies customer fiscal fields onto the draft receiver. Confirm `customerId` and `receiverRfc`.

An empty `POST /invoice-drafts` with `{}` is valid for a write check; fill `receiver*` via `PATCH /invoice-drafts/{id}/receiver` before items.

## 3. Comprobante (PUE)

```bash
curl --fail-with-body --max-time 60 --request PATCH \
  --url "$FACTUCAT_API_URL/invoice-drafts/DRAFT_ID/meta" \
  --header "X-API-Key: $FACTUCAT_API_KEY" \
  --header "Idempotency-Key: $OPERACION_ID-comprobante-v1" \
  --header "Content-Type: application/json" \
  --data '{
    "cfdiUse": "G03",
    "exportacion": "01",
    "paymentMethod": "PUE",
    "paymentForm": "03",
    "currency": "MXN"
  }'
```

`paymentForm` `03` is transferencia. `cfdiUse` must be compatible with the receiver's régimen.

**PPD:** if the sale is not paid at issuance, this PUE body is wrong. Deferred payment needs its own fiscal treatment and payment complements. The public API does not expose complement issuance. See [Cómo timbrar un CFDI completo](https://docs.factucat.com/guias/emitir-factura/).

## 4. Concepts

```bash
curl --fail-with-body --max-time 60 --request POST \
  --url "$FACTUCAT_API_URL/invoice-drafts/DRAFT_ID/items" \
  --header "X-API-Key: $FACTUCAT_API_KEY" \
  --header "Idempotency-Key: $OPERACION_ID-concepto-1-v1" \
  --header "Content-Type: application/json" \
  --data '{
    "productCode": "81111504",
    "unitCode": "E48",
    "description": "Servicio de programación de aplicaciones",
    "quantity": 1,
    "unitPrice": 1000,
    "vatRate": 0.16,
    "isrWithholdingRate": 0
  }'
```

`unitPrice` is pre-tax. Rates are fractions (`0.16` = 16%). FactuCat computes `subtotal`, `taxTotal`, and `total` — do not send them as writable fields. There are no product-catalog endpoints; keep SAT codes in your system. Further lines need their own body and idempotency key.

## 5. Review

`GET /invoice-drafts/{id}`. Check receiver, `cfdiUse` / `paymentMethod` / `paymentForm`, items, and totals. This is not a SAT pre-validation.

Corrections on the same draft:

- Receiver: `PATCH /invoice-drafts/{id}/receiver` with `receiver*` fields (changing only `customerId` here does **not** copy the new customer's fiscal data)
- Meta: `PATCH /invoice-drafts/{id}/meta`
- Item: `PATCH` or `DELETE` `/invoice-drafts/{id}/items/{itemId}`

Use a new idempotency key for each correction.

## 6. Stamp

```bash
curl --fail-with-body --max-time 60 --request POST \
  --url "$FACTUCAT_API_URL/invoice-drafts/DRAFT_ID/stamp" \
  --header "X-API-Key: $FACTUCAT_API_KEY" \
  --header "Idempotency-Key: $OPERACION_ID-timbrado-v1" \
  --header "Content-Type: application/json" \
  --data '{
    "skipIssuedEmailSend": true,
    "sendToCustomerContacts": false,
    "sendCustomerEmail": false,
    "sendCustomerWhatsApp": false
  }'
```

Success: `data.status` is `issued` and `data.satUuid` is present. Save `id`, `satUuid`, and `issuedAt`. Production stamp consumes a timbre.

If the HTTP call times out, `GET /invoices/{id}` with the saved id. Do not open another draft.

## 7. Files

`GET /invoices/{id}/xml` and `GET /invoices/{id}/pdf`. Bodies are JSON: `data.fileName`, `data.mimeType`, `data.base64`. Decode those bytes for the client. Retry a failed GET; do not stamp again.

## Persist

| Store | When |
| --- | --- |
| Your customer id ↔ `customerId` | After create or lookup |
| Your sale id ↔ document `id` | After draft create, before stamp |
| Idempotency key, path, and body | Before each mutation |
| `satUuid`, `status`, `issuedAt` | After stamp |
| XML and PDF | After download |
| `requestId` | Every response |

## Docs

[Cómo timbrar un CFDI completo](https://docs.factucat.com/guias/emitir-factura/) · [Clientes](https://docs.factucat.com/guias/clientes/) · [API reference](https://docs.factucat.com/api/)
