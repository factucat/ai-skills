# Environments and auth

Use one pairing per process. Do not send a sandbox key to production, or a live key to sandbox.

| Environment | Base URL | Key prefix | Timbrado |
| --- | --- | --- | --- |
| Sandbox | `https://sandbox.factucat.com/api/v1` | `fc_test_...` | No fiscal validity; no stamp charges |
| Production | `https://factucat.com/api/v1` | `fc_live_...` | Real CFDI; billable consumption |

## Configure the process

```bash
export FACTUCAT_API_KEY='fc_test_...'
export FACTUCAT_API_URL='https://sandbox.factucat.com/api/v1'
```

Switch environment by replacing **both** values. Each environment has its own account, keys, customers, and documents. IDs do not transfer.

Auth header on every request:

```
X-API-Key: $FACTUCAT_API_KEY
```

JSON mutations also send `Content-Type: application/json`.

## Sandbox

1. Register at [https://sandbox.factucat.com](https://sandbox.factucat.com). No card is required for this first path.
2. Create a key in [Configuración → API keys](https://sandbox.factucat.com/settings/api-keys). Copy the secret immediately.
3. Call `GET /me`.

When `FACTUCAT_API_SANDBOX` is on, sandbox preloads the shared SAT test CSD **EKU9003173C9**. The account can stamp without uploading a CSD. Do not treat sandbox XML/PDF as fiscally valid.

Emitter CSD alone is not enough: the **receiver** must pass SAT list checks at stamp. For a first demo without a real customer, use público en general (`XAXX010101000`, régimen `616`, `cfdiUse` `S01`) — see [stamp-flow.md](stamp-flow.md). Shape-only RFCs like `AAA010101AAA` fail at stamp.

`GET /me` checks the key and scopes. It does not certify that production fiscal data or a production CSD are ready.

## Production

On [https://factucat.com](https://factucat.com): fiscal data (RFC, legal name, régimen, código postal), a current CSD (`.cer`, `.key`, password) in Configuración, a plan with API access (FactuCat API or Miau Pro), and an `fc_live_...` key.

The public API does not upload CSD or register issuers. One account is one issuer RFC.

## Docs

[Ambientes y pruebas](https://docs.factucat.com/guias/ambientes/) · [Tu primera solicitud](https://docs.factucat.com/guias/primeros-pasos/)
