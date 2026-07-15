# ADR-0001: Use n8n for Workflow Orchestration

> 🇧🇷 [Ler em português](../../pt-BR/adrs/0001-orquestracao-com-n8n.md)

- **Status**: Accepted
- **Date**: `[to validate: exact decision date]` — early build phase of the project

## Context

The system needs to: receive WhatsApp webhooks, queue and process messages, call an LLM with tool support, integrate with Google Calendar, post to Chatwoot, run scheduled jobs (reminders, health checks, retries), and alert on failures — all maintained by a single person with no dedicated ops team. The choice was between hand-writing this as backend services (Node/Python) or building it on a visual workflow orchestration platform.

## Decision

Use n8n, self-hosted via Docker, as the orchestration layer for all of the above.

## Alternatives considered

- **Custom backend services (Node.js/Python) with a job queue (e.g., BullMQ)**: full control and better testability, but every integration (WhatsApp provider, Calendar, Chatwoot, LLM tool-calling, scheduling, retry logic) would need to be built and maintained from scratch, and every change would require a deploy.
- **A different automation platform (e.g., Zapier, Make)**: faster to start, but SaaS pricing scales badly with message volume, and neither offers the same level of control over self-hosted credentials, error-trigger workflows, or database-level access needed here.
- **n8n (chosen)**: visual workflows, built-in scheduling and error triggers, a large library of pre-built integration nodes, and self-hostable — avoiding both the "build everything from scratch" cost and the "pay per execution at scale" cost of SaaS automation tools.

## Consequences

**Positive**:
- Dramatically faster to build and iterate on integration-heavy logic than hand-written backend code, which mattered for a solo build under time pressure.
- The system's control flow is visually auditable — useful both for the maintainer's own debugging and for explaining the system to a non-technical stakeholder.
- Built-in credential storage, scheduling, and error-trigger handling removed infrastructure that would otherwise need to be built.

**Negative**:
- Workflow logic is harder to unit-test than plain code — correctness relies more on end-to-end testing (staging + allowlist) than automated test coverage.
- Exported workflow JSON produces noisy diffs, making code review and change history less legible than a typical pull request.
- This specific Docker deployment does not support `$env.*` variables inside workflows, forcing all secrets into n8n's own credential store rather than standard environment-variable configuration.
- Staging/production duplication of the highest-risk workflows (rather than one parameterized workflow) is a maintenance cost that grows with workflow count — tracked as architecture debt (see [Architecture § 9](../architecture.md#9-known-architecture-debt)).

## Related

[Architecture](../architecture.md), [Integrations § 3](../integrations.md#3-n8n-orchestration)
