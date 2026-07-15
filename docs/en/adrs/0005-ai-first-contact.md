# ADR-0005: Use AI for First Contact, with Tiered Models and a Hard Handoff Boundary

> 🇧🇷 [Ler em português](../../pt-BR/adrs/0005-ia-no-primeiro-contato.md)

- **Status**: Accepted
- **Date**: `[to validate: exact decision date]` — early build phase of the project

## Context

Clinics need every incoming patient message answered promptly, but a receptionist can't be online 24/7, and hiring more staff to cover volume is exactly the cost problem the product exists to solve. The alternative to a human-first model is an AI-first model — but healthcare-adjacent conversations carry real risk if the AI overreaches (clinical claims, price commitments, procedural promises).

## Decision

Let an AI assistant (Sofia) handle first contact and routine conversation by default, backed by tiered models (a fast/cheap tier for simple queries, a stronger tier for complex ones), with hard, code-enforced boundaries on what it's allowed to do, and an unconditional, immediate handoff to a human for anything outside those boundaries.

## Alternatives considered

- **Human-first, AI-assisted (AI drafts replies, a human approves/sends)**: safer in principle, but reintroduces the exact staffing bottleneck the product is meant to remove — a human still has to be online to approve every message.
- **AI-first with a single model tier**: simpler to build and reason about, but either overpays for simple queries (always using the strongest model) or underperforms on complex ones (always using the cheap model) — no single tier fit both ends of the query complexity range economically.
- **AI-first, tiered, with hard behavioral limits and automatic handoff (chosen)**: keeps the always-on responsiveness that solves the core problem, controls cost via tiering, and treats "the AI should not decide this" as a routing decision enforced by the Worker/handoff logic — not just a prompt instruction the model could ignore under pressure.

## Consequences

**Positive**:
- Patients get an immediate, always-available first response — directly addressing the core problem (see [README](../../../README.md#the-problem)).
- Cost per conversation stays controlled because most queries route to the cheaper model tier; only genuinely complex ones escalate.
- The handoff boundary is enforced in workflow logic (see [Business Rules § 2, § 3](../business-rules.md)), not solely in the prompt, so a model behaving unexpectedly still can't, for example, continue a conversation after a human has taken over — the pause is a database state check, not a suggestion to the model.

**Negative**:
- Two model tiers means two sets of behavior to validate and monitor for drift over time, rather than one.
- There is currently no automated eval suite for conversational quality — prompt/model changes are validated manually against staging, which doesn't scale indefinitely as flow complexity grows (flagged in [Integrations § 5](../integrations.md#5-the-ai-layer-llm--speech-to-text)).
- A full LLM provider outage stalls every conversation at once, with no fallback provider today.

## Related

[Sofia AI Flows](../sofia-ai-flows.md), [Business Rules § 2, § 3](../business-rules.md), [Integrations § 5](../integrations.md#5-the-ai-layer-llm--speech-to-text)
