# Integrations

> 🇧🇷 [Ler em português](../pt-BR/integracoes.md)

This document compares the external services ZapCare integrates with, why each was chosen at this stage, and what the trade-offs are. For the decision narrative behind the two most debated choices (n8n and UazAPI), see [ADR-0001](adrs/0001-n8n-orchestration.md) and [ADR-0004](adrs/0004-uazapi-vs-official-whatsapp-api.md).

## 1. WhatsApp messaging: official WhatsApp Business API vs. UazAPI

| | Official WhatsApp Business API (via a Business Solution Provider) | UazAPI (used today) |
|---|---|---|
| Setup complexity | Higher — requires Meta Business verification, a registered phone number, and typically a BSP contract | Lower — connects an existing WhatsApp number via a hosted gateway, close to plug-and-play |
| Cost at low volume | Often has a higher fixed/minimum cost floor, built for scale | Lower entry cost, fits a single-clinic pilot |
| Time to first working integration | Days to weeks (business verification, number registration) | Hours |
| Compliance / ToS risk | Fully within WhatsApp's supported integration path | Operates outside WhatsApp's officially sanctioned API path — carries a real risk of number restriction if messaging patterns look automated/spammy `[risk, not yet materialized]` |
| Message template requirements | Strict (pre-approved templates for business-initiated messages outside the 24h window) | Effectively unrestricted at the gateway level, but the same 24h engagement window logic still applies at the WhatsApp protocol level |
| Best fit | A scaled product with many clinics and revenue to support BSP costs | Validating product-market fit with a small number of pilot clinics before committing to heavier infrastructure |

**Why UazAPI now**: the priority at this stage is validating whether clinics get value from the assistant at all — not building for a scale that doesn't exist yet. UazAPI let the first pilot go live in hours instead of weeks. The explicit trade-off accepted is compliance/ToS risk, tracked as a real item to revisit before scaling clinic count meaningfully (see [ADR-0004](adrs/0004-uazapi-vs-official-whatsapp-api.md)).

**Next step**: migrate to the official API once clinic count and revenue justify BSP costs and the heavier setup — the system was deliberately kept decoupled (all WhatsApp I/O goes through the Receptor/Worker workflows, not scattered across the codebase) specifically so this migration is a provider swap, not a rewrite.

## 2. Chatwoot (human handoff / support inbox)

- **Advantages**: open-source and self-hostable (no per-seat SaaS cost at this scale), purpose-built for exactly this use case (multi-channel support inbox with labels, assignment, and a webhook API for resolution events), and familiar UI for non-technical clinic staff.
- **Limitations**: self-hosting means ZapCare owns its uptime and patching, not a vendor SLA. `[to validate: current uptime track record]`.
- **Risks**: a Chatwoot outage removes the human-handoff path entirely — this is why Health Check monitors Chatwoot reachability specifically (see [Architecture § 6](architecture.md#6-reliability-mechanisms)).
- **Next steps**: none planned at this stage — Chatwoot has not been a bottleneck.

## 3. n8n (orchestration)

- **Advantages**: visual workflows make the system's logic auditable without reading source code line-by-line — relevant both for a solo operator's own maintainability and for explaining the system to a non-technical stakeholder. Built-in scheduling, error-trigger handling, and credential management removed the need to build that infrastructure from scratch.
- **Limitations**: workflow logic is harder to unit-test than plain code, and diffs on exported JSON are not human-friendly — a real cost documented in [Architecture § 9](architecture.md#9-known-architecture-debt) and [ADR-0001](adrs/0001-n8n-orchestration.md).
- **Risks**: the environment-variable limitation of this specific Docker deployment (`$env.*` unavailable) forces credentials into n8n's own store — acceptable, but a constraint worth knowing about before assuming standard 12-factor config would just work.
- **Next steps**: as workflow count and complexity grow, evaluate consolidating STG/prod duplicated workflows into single parameterized ones.

## 4. Supabase (database / backend)

- **Advantages**: managed PostgreSQL with Row-Level Security, Realtime subscriptions, and auto-generated REST/RPC access meant the admin panel and landing page could talk directly to the database with no separate backend API to build and operate — a meaningful velocity win for a solo-built product.
- **Limitations**: RLS policy correctness is the entire security model for public-facing writes (the landing page's anon key) — this concentrates risk in policy review rather than spreading it across an API layer with its own validation.
- **Risks**: two of the newer tables (`crm_leads`, `crm_notes`) exist only in the live database, not in version-controlled migrations — a known gap (see [Architecture § 9](architecture.md#9-known-architecture-debt)).
- **Next steps**: backfill the missing migrations for CRM tables; formalize a single migration stream (currently two separate migration folders exist).

## 5. The AI layer (LLM + speech-to-text)

- **Advantages**: tool-calling (function calling) support was a hard requirement — it's what makes Sofia's booking flow write to a real calendar instead of hallucinating confirmation text. Tiered routing between a fast/cheap model and a stronger model keeps average cost per conversation low while still escalating genuinely complex queries to a more capable model.
- **Limitations**: model behavior (tone, refusal boundaries, tool-call reliability) shifts across model versions/providers — prompts and guardrails need periodic revalidation, not a "set once" assumption.
- **Risks**: an LLM provider outage stalls every conversation simultaneously across every clinic — there is currently no fallback provider. `[to validate: acceptable given current scale, worth revisiting before it isn't]`.
- **Next steps**: formalize a lightweight eval process for prompt/model changes before they reach production, given there's no automated test suite for conversational quality today.

## 6. Google Calendar

- **Advantages**: clinics already use it — zero migration friction for the clinic, and it's the source of truth availability is checked against, avoiding a second calendar system the clinic would need to keep in sync manually.
- **Limitations**: OAuth2 credentials can expire/require reconnection, which has caused real incidents (see the internal runbook, not published here). A booking failure due to Calendar has a defined fallback (human handoff), but Calendar downtime still degrades the booking experience.
- **Next steps**: `[to validate: whether a calendar redundancy or alerting improvement is planned]`.
