# ADR-0006: Separate Admin Panel, Landing Page, and Workflows into Independent Deployables

> 🇧🇷 [Ler em português](../../pt-BR/adrs/0006-separacao-admin-landing-workflows.md)

- **Status**: Accepted
- **Date**: `[to validate: exact decision date]` — early build phase of the project

## Context

The system has three very different concerns: internal operations tooling (admin panel), public marketing/lead-capture (landing page), and the automation/orchestration logic (n8n workflows). These have different audiences (operator vs. public visitors vs. the runtime system itself), different risk profiles (a landing page bug is embarrassing; a workflow bug can send a wrong message to a real patient), and different deploy cadences.

## Decision

Keep the admin panel, landing page, and n8n workflows as three independently deployable units, coordinated only through the shared Supabase database — never sharing a codebase, a deploy pipeline, or direct service-to-service calls.

## Alternatives considered

- **A single monolithic web app serving both public marketing pages and the authenticated admin area**: fewer moving parts to deploy, but couples the public site's uptime/deploy risk to the admin tool's, and mixes a public unauthenticated surface with an authenticated internal one in the same codebase — a worse security boundary.
- **n8n workflows calling the admin panel's API directly (or vice versa)**: tighter coupling would allow richer real-time interaction, but breaks the "the database is the contract" principle (see [Architecture § 1](../architecture.md#1-overview)), making each side depend on the other's uptime and API stability rather than only on Supabase.
- **Three independent deployables coordinated via the database (chosen)**: the landing page can be redeployed without touching workflows; a workflow can be edited without redeploying either web app; and an outage in one (e.g., the admin panel is down) doesn't take down patient-facing conversation handling.

## Consequences

**Positive**:
- Clear blast-radius containment: a landing page bug cannot break patient conversations; a workflow bug cannot corrupt the marketing site.
- Each piece can be built, tested, and deployed on its own timeline — relevant for a solo developer context-switching between "sales asset" work and "core product" work.
- The database-as-contract pattern keeps all three pieces honest about what data actually looks like, since there's no private internal API to paper over a schema mismatch.

**Negative**:
- No shared code between the admin panel and landing page despite both being React/Vite/Tailwind apps with overlapping utility needs (e.g., phone number formatting) — some duplication exists rather than a shared internal package. `[to validate: extent of duplication]`
- Coordinating a change that spans, say, a new CRM field touched by both the landing page's lead form and the admin panel's CRM view requires updating two codebases plus the shared schema, rather than one.

## Related

[Architecture](../architecture.md), [System Requirements § 6 (Maintenance)](../system-requirements.md#6-maintenance-requirements)
