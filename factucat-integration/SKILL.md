---
name: factucat-integration
description: Use this skill when an agent integrates the FactuCat CFDI API into an app (sandbox or production), stamps invoices via REST, or sets up agentic/vibecoding against FactuCat.
metadata: {"openclaw":{"emoji":"🧾","homepage":"https://github.com/factucat/ai-skills/tree/main/factucat-integration","skillKey":"factucat-integration","primaryEnv":"FACTUCAT_API_KEY"}}
---

# FactuCat Integration

REST integration for **Mexican CFDI 4.0** on FactuCat Cloud. Use HTTPS JSON against `/api/v1`. This skill is not the CLI; for terminal workflows use `factucat-cli`.

## Use This Skill When

- An app needs to create, stamp, or download CFDI through FactuCat REST
- A coding agent is vibecoding an integration against sandbox or production
- Auth, environments, idempotency, or SAT/API errors come up during that work

## Operating Rules

- Pair base URL and key from the same environment; never mix them
- Send the secret as header `X-API-Key` from the server only (never in the browser, never in query strings)
- Use Mexican CFDI and SAT terminology
- Send `Idempotency-Key` on every mutation
- Before production stamp: `GET /me`; on 403 `plan_required` stop. Free/Miau need Miau Pro or FactuCat API via webapp Configuración → Suscripción (`https://factucat.com/settings/subscription` or staging equivalent) → Stripe Checkout (optional promo/coupon when Checkout shows it). No public REST API to upgrade, start Checkout, or apply coupons — do not invent one. Sandbox needs no paid plan. See [facturacion](https://docs.factucat.com/guias/facturacion/)
- When fiscal details, stamp steps, retries, or errors matter, read the matching file under `references/`
- Canonical HTTP contract: [docs.factucat.com](https://docs.factucat.com)

## Environments

| Environment | Base URL | Key prefix |
| --- | --- | --- |
| Sandbox | `https://sandbox.factucat.com/api/v1` | `fc_test_...` |
| Production | `https://factucat.com/api/v1` | `fc_live_...` |

Renaming a key prefix does not change its environment. Customer, draft, and invoice IDs belong to the environment that created them.

On sandbox, FactuCat runs with `FACTUCAT_API_SANDBOX` on and preloads the shared SAT test CSD `EKU9003173C9`. The account can stamp without uploading a CSD. Production needs fiscal data, a current CSD, and a plan with API access (FactuCat API or Miau Pro), configured in the webapp — the public API has no CSD-upload endpoint.

Details: [references/environments.md](references/environments.md)

## Quick Path

1. Register at [https://sandbox.factucat.com](https://sandbox.factucat.com) (no card)
2. Create a key in Configuración → API keys (`fc_test_...`; shown once)
3. Confirm with `GET /me`

```bash
export FACTUCAT_API_URL='https://sandbox.factucat.com/api/v1'
curl --fail-with-body --max-time 60 \
  --url "$FACTUCAT_API_URL/me" \
  --header "X-API-Key: $FACTUCAT_API_KEY"
```

A 200 returns `data` (account + key scopes) and `meta.requestId`. It does not prove the CSD is ready to stamp.

Recommended stamp scopes: `customer:read`, `customer:write`, `invoice:read`, `invoice:write`, `invoice:stamp`.

## Minimal Stamp Flow (PUE)

Pago en una sola exhibición: **customer → draft → concepts/meta → review → stamp → XML/PDF**.

1. `POST /customers` (or reuse a stored `customerId`)
2. `POST /invoice-drafts` with that `customerId`
3. `PATCH /invoice-drafts/{id}/meta` with `paymentMethod: "PUE"`
4. `POST /invoice-drafts/{id}/items`
5. `GET /invoice-drafts/{id}` and review receiver, meta, items, totals
6. `POST /invoice-drafts/{id}/stamp`
7. `GET /invoices/{id}/xml` and `GET /invoices/{id}/pdf`

Persist the document `id` before stamp. After a successful stamp, `status` is `issued` and `satUuid` is set; read issued docs from `/invoices/{id}`, not `/invoice-drafts/{id}`.

**Sandbox demo tip:** the sample customer RFC in `references/stamp-flow.md` is shape-only and **fails** SAT receiver checks at stamp. For a first sandbox `issued` (XML/PDF), use público en general (`XAXX010101000` / régimen `616` / `cfdiUse` `S01`) as documented there — or a real SAT-registered receiver.

**PPD** (pago en parcialidades o diferido) and payment complements need a different fiscal treatment. The public API does not expose complement issuance. Follow [emitir-factura](https://docs.factucat.com/guias/emitir-factura/).

Full sequence: [references/stamp-flow.md](references/stamp-flow.md)

## Idempotency

Send `Idempotency-Key` on POST/PATCH/DELETE. One key is one logical request (same method, path, and body). Reuse it on transport retries; use a new key for a new operation or a corrected body. Stamp with a different key than draft creation.

Timeouts during stamp are not proof of failure: `GET /invoices/{id}` with the saved id. Do not create a second draft.

Details: [references/idempotency.md](references/idempotency.md)

## Errors

Success bodies have `data` and `meta.requestId`. Failures have `error.code`, `error.message`, `error.requestId`, `error.retryable`. Keep `X-Request-Id` / `requestId` when debugging.

SAT catalog and receiver checks run at stamp, not at `GET /me`. Do not invent rate-limit numbers; on **429** honor `Retry-After`.

Common cases and doc pointers: [references/errors.md](references/errors.md)

Guides: [primeros-pasos](https://docs.factucat.com/guias/primeros-pasos/), [emitir-factura](https://docs.factucat.com/guias/emitir-factura/), [idempotencia](https://docs.factucat.com/guias/idempotencia/), [respuestas](https://docs.factucat.com/guias/respuestas/), [ambientes](https://docs.factucat.com/guias/ambientes/). API map: [docs.factucat.com/api](https://docs.factucat.com/api/).

## Starter Prompt

Paste this into a coding agent:

```
Integrate FactuCat CFDI 4.0 over REST (not the CLI). Start in sandbox.

Base URL: https://sandbox.factucat.com/api/v1
Auth: header X-API-Key with a fc_test_... key. Never mix sandbox keys with https://factucat.com/api/v1 (or fc_live_... with sandbox). Keep the key on the server.

Setup: register at https://sandbox.factucat.com (no card) → Configuración → API keys → GET /me.

Sandbox preloads SAT test CSD EKU9003173C9; you can stamp without uploading a CSD.

PUE flow: POST /customers → POST /invoice-drafts (customerId) → PATCH /invoice-drafts/{id}/meta (paymentMethod PUE) → POST .../items → GET draft → POST .../stamp. Send Idempotency-Key on every mutation. Save the document id before stamp.

For a sandbox demo stamp without a real customer RFC: empty draft → PATCH receiver XAXX010101000 / PUBLICO EN GENERAL / régimen 616 → meta cfdiUse S01 + PUE → items → stamp. Do not stamp with the skill's shape-only AAA010101AAA example.

Docs: https://docs.factucat.com (primeros-pasos, emitir-factura, idempotencia, respuestas, ambientes).
```

## Install

```bash
npx skills add factucat/ai-skills@factucat-integration
```

Equivalent: `npx skills add factucat/ai-skills --skill factucat-integration`.

Without the skills CLI, copy `SKILL.md` plus `references/` into the agent's skills directory, or fetch:

```
https://raw.githubusercontent.com/factucat/ai-skills/main/factucat-integration/SKILL.md
```
