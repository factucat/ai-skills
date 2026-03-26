# Install and authenticate

## Install

```bash
npm install -g @factucat/cli
```

Verify:

```bash
factucat --help
```

## Update

```bash
npm install -g @factucat/cli@latest
```

## Save an API key

Interactive:

```bash
factucat auth api-key set
```

Explicit value:

```bash
factucat auth api-key set --value fc_live_xxxxx
```

Standard input:

```bash
printf 'fc_live_xxxxx\n' | factucat auth api-key set
```

## Verify auth

Human-readable:

```bash
factucat auth status
```

Machine-readable:

```bash
factucat auth status --json
```

If the user sees `API key inválida o revocada`, re-save a valid key and retry.
