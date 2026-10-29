# FactuCat AI skills

Agent skills for Mexican CFDI 4.0 on FactuCat.

| Skill | Use when |
| --- | --- |
| [`factucat-cli`](factucat-cli/) | Install, authenticate, or operate the FactuCat CLI |
| [`factucat`](factucat/) | Use FactuCat via CLI, MCP or REST: sandbox/production, stamp and cancel with confirmation |

## Install

[skills](https://skills.sh) CLI:

```bash
npx skills add factucat/ai-skills --list
npx skills add factucat/ai-skills@factucat
npx skills add factucat/ai-skills@factucat-cli
```

Equivalent: `npx skills add factucat/ai-skills --skill factucat`.

Without that CLI, copy the skill folder into the agent's skills directory, or fetch `SKILL.md`:

```
https://raw.githubusercontent.com/factucat/ai-skills/main/factucat/SKILL.md
https://raw.githubusercontent.com/factucat/ai-skills/main/factucat-cli/SKILL.md
```

API docs: [https://docs.factucat.com](https://docs.factucat.com)
