# FactuCat MCP server

Transporte: Streamable HTTP, sin estado, **solo POST** (GET → 405), JSON. Versiones de protocolo negociadas (SDK 2.4.0; clientes por defecto `2025-11-25`). Guarda `X-Request-Id` para soporte. `Origin` ajeno → 403.

Conectar:
- **claude.ai / Claude Desktop / ChatGPT (OAuth):** agrega conector personalizado con URL `https://mcp.factucat.com/mcp`, inicia sesión, **elige el RFC/espacio** y acepta permisos. Token 15 min + refresh (`offline_access`).
- **Claude Code:** `claude mcp add --transport http factucat https://mcp.factucat.com/mcp --header "Authorization: Bearer ${FACTUCAT_API_KEY}"`
- **Cursor** (`~/.cursor/mcp.json`): `{"mcpServers":{"factucat":{"url":"https://mcp.factucat.com/mcp","headers":{"Authorization":"Bearer ${env:FACTUCAT_API_KEY}"}}}}`
- **SDK:** `@modelcontextprotocol/client` + `StreamableHTTPClientTransport` con header `Authorization: Bearer …`.

Scopes (iguales en OAuth y API key; `tools/list` solo anuncia lo que tu credencial puede usar; sin permiso → `missing_scope` + `required_scope`):

| Scope | Herramientas |
| --- | --- |
| (ninguno) | `whoami` |
| `customer:read` | `customers_list`, `customers_get` |
| `customer:write` | `customers_create` |
| `invoice:read` | `drafts_list`, `drafts_get`, `invoices_list`, `invoices_get`, `invoice_get_xml`, `invoice_get_pdf` |
| `invoice:write` | `draft_create`, `draft_set_receiver`, `draft_set_meta`, `draft_add_item`, `draft_update_item`, `draft_remove_item`, `draft_delete` |
| `invoice:stamp` | `invoice_stamp` |
| `invoice:cancel` | `invoice_cancel` (y sustitución en `invoice_stamp`) |
| `payroll:read` | `employees_list`, `employees_get`, `payroll_batches_list`, `payroll_batches_get`, `payroll_receipts_get`, `payroll_receipt_get_pdf`, `payroll_receipt_get_xml` |
| `payroll:write` | `payroll_batch_stamp` (lote completo), `payroll_receipt_cancel` (solo motivo 02) |


Nómina por MCP: requiere **plan Miau** (como la web), solo OAuth (API keys sin `payroll:*`). CURP, NSS y cuenta bancaria salen **enmascarados** (últimos 4); percepciones y salarios sí se muestran. Crear lotes/empleados y sustitución (motivo 01) no están en v1.

Escrituras fiscales (`invoice_stamp`, `invoice_cancel`, `payroll_batch_stamp`, `payroll_receipt_cancel`) — dos pasos:
1. Llama **sin** `confirmation_token` y con `idempotency_key` → `status:"confirmation_required"`, `preview`, `confirmation_token` (vence 5 min). No timbra.
2. Muestra la preview y espera el "sí".
3. Repite con el `confirmation_token`, **la misma** `idempotency_key` y **los mismos** argumentos → resultado (UUID / `cancelled` / `not_confirmed`).
Errores: `confirmation_mismatch` (cambió el documento) / `_invalid` / `_expired` → pide preview nueva; `idempotency_in_progress` → reintenta igual; `idempotency_conflict` → llave nueva; `free_card_required`, `free_extra_limit_reached`, `plan_limit_reached` → plan/tarjeta en la web.

Timbrado en producción por MCP depende de `MCP_LIVE_ENABLED` (si está apagado, OAuth no puede pedir `invoice:stamp`/`invoice:cancel` en prod). Staging siempre es `test`.

