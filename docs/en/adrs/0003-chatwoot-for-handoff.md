# ADR-0003: Use Chatwoot for Human Handoff

> 🇧🇷 [Ler em português](../../pt-BR/adrs/0003-chatwoot-para-transferencia-humana.md)

- **Status**: Accepted
- **Date**: `[to validate: exact decision date]` — early build phase of the project

## Context

When Sofia hands a conversation to a human, clinic staff need a place to see it, reply to the patient, and mark it resolved — something usable by non-technical front-desk staff, not a raw database view or a developer tool.

## Decision

Use Chatwoot (self-hosted, open-source) as the human-facing support inbox for all escalated conversations.

## Alternatives considered

- **Build a custom "handoff inbox" inside the admin panel**: full control over UX, but duplicates a large amount of functionality (multi-conversation inbox, assignment, labels, read/unread state, resolution tracking) that already exists in mature support tools — a poor use of solo-build time for a non-differentiating feature.
- **A commercial helpdesk SaaS (e.g., Zendesk, Intercom)**: mature and polished, but per-seat/per-conversation SaaS pricing is a poor fit at this stage, and most of their feature depth (ticketing workflows, SLAs across many channels) is unnecessary here.
- **Chatwoot (chosen)**: open-source and self-hostable (no per-seat cost at this scale), has a webhook API for resolution events (needed to reactivate the AI automatically), and its UI is already familiar to many support staff since it resembles common helpdesk tools.

## Consequences

**Positive**:
- Avoided building a bespoke inbox UI entirely — meaningful scope reduction for a solo build.
- The webhook-driven "resolved → reactivate AI" loop (see [Architecture § 5](../architecture.md#5-data-flow-human-handoff)) was straightforward to implement because Chatwoot exposes conversation-state events, not just a UI.
- No recurring per-seat SaaS cost as more clinics are added.

**Negative**:
- Self-hosting means ZapCare — not a vendor — owns Chatwoot's uptime, patching, and backups.
- A Chatwoot outage removes the entire human-handoff path; this is why it's included in the automated health check (see [Architecture § 6](../architecture.md#6-reliability-mechanisms)).

## Related

[Architecture § 3.4](../architecture.md#34-human-handoff-chatwoot), [Business Rules § 3](../business-rules.md#3-human-handoff-rules), [Integrations § 2](../integrations.md#2-chatwoot-human-handoff--support-inbox)
