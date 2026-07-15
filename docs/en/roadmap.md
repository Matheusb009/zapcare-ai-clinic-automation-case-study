# Roadmap

> 🇧🇷 [Ler em português](../pt-BR/roadmap.md)

This roadmap reflects the real, internally tracked backlog — grouped by track rather than by date, since a solo-operated product's priorities shift with what the current pilot clinic needs. Items are phrased as shipped or not; specific target dates are intentionally omitted where not yet committed.

## MVP — shipped

- End-to-end conversational flow: consent gate → Sofia (tiered AI) → booking or handoff → confirmation.
- Voice message support (transcription → text → AI reply).
- Human handoff into Chatwoot with an AI-generated briefing, and automatic AI reactivation on resolution.
- Appointment booking and cancellation, synced with Google Calendar.
- Automated appointment reminders with confirmation tracking.
- Asynchronous message queue with retry, exponential backoff, and dead-letter handling.
- Scheduled health checks, SLA monitoring for handoffs, and abandoned-conversation reactivation.
- Multi-clinic data isolation at the schema level.
- Admin panel: clinic management, conversation/appointment/queue views, manual billing controls, sales CRM.
- Public landing page with an interactive lead-qualification flow feeding the CRM.
- Staged rollout process: staging environment with an allowlist gate, validated against a real pilot clinic before wider rollout.

## Near-term improvements

- Resolve remaining pilot-validation findings from the most recent end-to-end test pass before onboarding additional clinics. `[to validate: current punch-list status]`
- Consolidate the two separate SQL migration streams into one chronological history.
- Backfill version-controlled migrations for the CRM tables (`crm_leads`, `crm_notes`) currently managed only in the live database.
- Formal credential rotation pass for third-party service tokens (WhatsApp provider, Chatwoot), as a preventive measure rather than a response to an incident.

## Product improvements

- Clinic onboarding flow that doesn't require the operator to hand-configure every field.
- Automated appointment confirmation flows beyond the current reminder/confirm pattern.
- Aggregate operational metrics dashboard (messages/day, bookings/day, handoff rate) rather than relying on raw log/queue views.
- Richer patient/lead CRM features as clinic count grows (currently sized for a small pilot set).

## Technical improvements

- CI/CD for the admin panel and landing page (currently manual `vercel --prod` deploys).
- Consolidate staging/production workflow duplication (Receptor, Worker, Sofia) into single parameterized workflows where practical.
- A lightweight evaluation process for AI prompt/model changes before they reach production.
- Push-based realtime updates in more admin panel views, reducing reliance on polling.

## Security and compliance

- Formalize third-party credential rotation cadence (currently ad hoc/preventive rather than scheduled).
- Legal review of current LGPD data-retention periods (logs, conversation history, appointment anonymization windows).
- Evaluate data-processing agreement requirements with AI/infrastructure providers.
- Extend Row-Level Security coverage to any remaining tables currently reliant on `service_role` bypass rather than policy-level protection. `[to validate: current coverage gaps]`

## Scalability

- Load-test the architecture beyond a single pilot clinic's traffic pattern before committing to a larger onboarding wave.
- Revisit the UazAPI-vs-official-WhatsApp-API decision once clinic count and revenue justify the migration (see [ADR-0004](adrs/0004-uazapi-vs-official-whatsapp-api.md)).
- Design the transition path from manual billing to automated billing (Stripe or equivalent) ahead of actually needing it (see [ADR-0008](adrs/0008-manual-billing-before-stripe.md)).

## Commercial / prospecting

- Systematize outbound qualification against the defined ideal-customer profile (1–5 professionals, WhatsApp-based booking today, Google Calendar/Agenda user, owner-led decision-making).
- Build out the sales CRM pipeline stages into repeatable playbooks (demo script, objection handling, pilot-to-paid conversion process).
- Track pilot-to-paying-customer conversion rate as the core commercial validation metric before scaling outbound volume.

## Future SaaS direction

- Self-service billing portal for clinics (explicitly deferred — see [ADR-0008](adrs/0008-manual-billing-before-stripe.md)).
- Full multi-doctor / multi-professional support beyond the current per-clinic calendar mapping.
- Basic medical-record ("prontuário") functionality, if validated as a genuine clinic need rather than scope creep.
- Formal patient/lead reactivation campaigns as a standalone product feature (today, reactivation exists only for abandoned human handoffs, not marketing win-back).
- A permissioned, multi-user admin panel (today, effectively single-operator) with per-clinic configuration self-service.
