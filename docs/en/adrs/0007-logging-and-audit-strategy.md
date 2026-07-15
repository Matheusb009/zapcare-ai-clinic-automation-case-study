# ADR-0007: Structured Logging and Explicit Audit Trails Over Silent Writes

> 🇧🇷 [Ler em português](../../pt-BR/adrs/0007-estrategia-de-logs-e-auditoria.md)

- **Status**: Accepted
- **Date**: `[to validate: exact decision date]` — early build phase of the project

## Context

A system operated by one person, with no dedicated ops or QA team, will fail in ways nobody is watching for in real time unless failures and sensitive actions are made visible by design. Two different needs exist here: (1) operational logging, so failures can be diagnosed after the fact, and (2) audit trails for sensitive actions (billing changes, AI pause/resume, consent), which need to answer "who did what, when" even if the operator is the only "who" today.

## Decision

- Log every workflow error/warning/info event to a structured `logs` table, tagged by workflow name, node name, and relevant identifiers (phone, message ID) — not just to n8n's own execution history, which is harder to query and has its own retention limits.
- Treat consent records as permanent, immutable audit records, never subject to the standard log-retention cleanup.
- Route every sensitive admin-panel action (pausing AI, requeueing a message, changing billing status) through code paths that write an explicit log entry, rather than a bare database update with no trace.

## Alternatives considered

- **Rely on n8n's built-in execution log only**: zero extra work, but execution logs are workflow-scoped and not easily queryable/joinable with business data (e.g., "show me every error for this clinic across every workflow"), and are subject to n8n's own retention settings, not a deliberate business-driven retention policy.
- **Log everything indefinitely**: simplest retention policy, but conflicts directly with the LGPD data-minimization principle applied elsewhere in the system (see [Business Rules § 8](../business-rules.md#8-security-and-privacy-rules)) — most operational logs don't need to live forever, and some (like message content in error logs) are exactly the kind of data that should be minimized.
- **Structured logs table with differentiated retention, plus explicit audit-trail writes for sensitive actions (chosen)**: gives queryable, cross-workflow diagnosability, keeps genuinely permanent records (consent) permanent, and lets everything else expire on a defined schedule.

## Consequences

**Positive**:
- Failures are diagnosable by workflow and node without needing to open n8n's UI and hunt through execution history.
- Sensitive admin actions have a trace — relevant both for the operator's own accountability and as a foundation if a second operator/team member is ever added.
- Log retention is a deliberate, documented policy rather than "whatever the tool defaults to."

**Negative**:
- This is a manual discipline enforced by how each workflow/panel action is written, not something a framework guarantees automatically — a new workflow that forgets to log an error is a real, recurring risk, not a hypothetical one.
- Retention cleanup itself is a scheduled job (part of the Retry Scheduler) that, if it silently failed, would let logs grow unbounded without anyone noticing immediately — `[to validate: is cleanup failure itself monitored]`.

## Related

[Architecture § 6, § 7](../architecture.md), [Business Rules § 8](../business-rules.md#8-security-and-privacy-rules), [System Requirements § 4 (Observability)](../system-requirements.md#4-observability-requirements)
