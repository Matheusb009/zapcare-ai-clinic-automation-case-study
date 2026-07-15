# ADR-0004: UazAPI vs. the Official WhatsApp Business API

> 🇧🇷 [Ler em português](../../pt-BR/adrs/0004-uazapi-vs-api-oficial-whatsapp.md)

- **Status**: Accepted, with a planned future revisit
- **Date**: `[to validate: exact decision date]` — early build phase of the project

## Context

Every conversation the product exists to automate happens over WhatsApp. There are two broad paths to send/receive WhatsApp messages programmatically: the official WhatsApp Business API (via a Meta-approved Business Solution Provider) or a third-party gateway that connects to an existing WhatsApp number outside Meta's officially sanctioned integration path (e.g., UazAPI).

## Decision

Use UazAPI for the pilot and early-clinic stage. Treat the official API as a planned future migration, not a permanently rejected option.

## Alternatives considered

- **Official WhatsApp Business API via a BSP**: fully compliant, built for scale, but requires Meta business verification, a registered/dedicated number, pre-approved message templates for business-initiated contact, and typically a BSP contract — days to weeks of setup and a higher cost floor, all before validating whether a single clinic would even use the product.
- **UazAPI (chosen for now)**: connects an existing clinic WhatsApp number through a hosted gateway in hours, with a much lower cost floor — but operates outside WhatsApp's officially sanctioned integration path, carrying real risk of number restriction if messaging patterns look automated or abusive.

## Consequences

**Positive**:
- The first pilot clinic could go live in hours instead of weeks, which mattered when the goal was validating product value, not building for scale that didn't exist yet.
- Lower fixed cost fits a pre-revenue/early-revenue stage.

**Negative / accepted risk**:
- Real, acknowledged risk of WhatsApp restricting or banning the connected number if usage patterns are flagged — this is a business continuity risk, not just a technical inconvenience, and is explicitly tracked rather than ignored.
- No Meta-backed SLA or support path if something goes wrong at the platform level.

**Mitigation / migration path**:
- All WhatsApp I/O is isolated to the Receptor/Worker/Reminders workflows rather than scattered through the codebase — a deliberate design choice specifically so that migrating to the official API later is a provider swap at the integration boundary, not an architecture rewrite.
- The decision is explicitly scoped to "now, at this scale" — see [Integrations § 1](../integrations.md#1-whatsapp-messaging-official-whatsapp-business-api-vs-uazapi) for the full comparison and the planned trigger for revisiting it (clinic count and revenue justifying BSP cost and setup).

## Related

[Integrations § 1](../integrations.md#1-whatsapp-messaging-official-whatsapp-business-api-vs-uazapi), [Architecture](../architecture.md)
