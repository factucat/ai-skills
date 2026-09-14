# FactuCat AI skills

Agent skills for Mexican CFDI 4.0 on FactuCat.

| Skill | Use when |
| --- | --- |
| [`factucat-cli`](factucat-cli/) | Install, authenticate, or operate the FactuCat CLI |
| [`factucat-integration`](factucat-integration/) | Integrate the FactuCat REST API into an app, stamp via HTTP, or vibecode against sandbox/production |

## Install

[skills](https://skills.sh) CLI:

```bash
npx skills add factucat/ai-skills --list
npx skills add factucat/ai-skills@factucat-integration
npx skills add factucat/ai-skills@factucat-cli
```

Equivalent: `npx skills add factucat/ai-skills --skill factucat-integration`.

Without that CLI, copy the skill folder into the agent's skills directory, or fetch `SKILL.md`:

```
https://raw.githubusercontent.com/factucat/ai-skills/main/factucat-integration/SKILL.md
https://raw.githubusercontent.com/factucat/ai-skills/main/factucat-cli/SKILL.md
```

API docs: [https://docs.factucat.com](https://docs.factucat.com)
