# System Requirements

> 🇧🇷 [Ler em português](../pt-BR/requisitos-de-sistema.md)

Status legend: **Implemented** (built and validated at least in staging), **Partial** (built but with a known gap), **Planned** (on the roadmap, not built), **`[to validate]`** (behavior believed true but not confirmed against production at the time of writing).

## 1. Functional requirements

| ID | Requirement | Status |
|---|---|---|
| FR-1 | The system must receive inbound WhatsApp messages (text and voice) for a clinic's configured number and route them to the AI assistant. | Implemented |
| FR-2 | The system must transcribe voice messages to text before generating an AI reply. | Implemented |
| FR-3 | The system must hold conversation context (history + summarized memory) per patient/clinic pair across multiple messages. | Implemented |
| FR-4 | The AI assistant must check calendar availability before confirming a booking. | Implemented |
| FR-5 | The AI assistant must be able to create, and the system must be able to update/cancel, an appointment, reflected in both the database and Google Calendar. | Implemented |
| FR-6 | The system must escalate a conversation to a human when the patient explicitly requests it, the case is sensitive, the question is out of scope, or a booking operation fails. | Implemented |
| FR-7 | A human-escalated conversation must include an AI-generated summary of the conversation so far. | Implemented |
| FR-8 | The AI must stop responding to a conversation the moment it is handed to a human, and must resume only after the human resolves it (or after a reactivation policy applies). | Implemented |
| FR-9 | The system must send automated appointment reminders ahead of the scheduled time and track patient confirmation. | Implemented |
| FR-10 | The system must capture and record explicit LGPD consent from a new patient before generating any AI reply to them. | Implemented |
| FR-11 | The admin panel must let the operator view clinics, conversations, the processing queue, and appointments across all clinics. | Implemented |
| FR-12 | The admin panel must let the operator manually set a clinic's billing status (trial / active / overdue / cancelled). | Implemented |
| FR-13 | The admin panel must provide a sales CRM (lead capture, pipeline stage, notes) independent of the clinic-operations views. | Implemented |
| FR-14 | The public landing page must offer a lead-qualification flow that writes a new lead into the CRM without requiring the visitor to create an account. | Implemented |
| FR-15 | The system must support more than one clinic concurrently, each with independent configuration, conversations, and data. | Implemented |
| FR-16 | Clinic front-desk staff must be able to view and respond to escalated conversations without accessing the admin panel or n8n. | Implemented (via Chatwoot) |
| FR-17 | The system must provide a self-service patient/client billing portal. | Planned — see [Roadmap](roadmap.md) |
| FR-18 | The system must support recurring, automated billing (not just manual status updates). | Planned — see [ADR-0008](adrs/0008-manual-billing-before-stripe.md) |

## 2. Non-functional requirements

| ID | Requirement | Status |
|---|---|---|
| NFR-1 | A patient message should receive an AI reply within a time window that feels conversational (not menu-bot slow). | `[to validate: no formally measured p95 latency yet]` |
| NFR-2 | The system should minimize LLM cost per conversation by routing simple queries to a cheaper/faster model and only escalating complexity to a stronger model. | Implemented |
| NFR-3 | The system should degrade gracefully when an external dependency (Calendar, WhatsApp provider, Chatwoot) is unavailable, rather than losing the message. | Partial — Calendar has an explicit fallback path; WhatsApp/Chatwoot outages rely on the queue's retry mechanism |
| NFR-4 | The admin panel should reflect near-real-time operational state without requiring a manual refresh. | Implemented — polling-based auto-refresh (not push-based realtime for every view) |
| NFR-5 | Conversation memory should be bounded so that prompt size (and therefore cost/latency) does not grow unbounded with conversation length. | Implemented — windowed memory |
| NFR-6 | The system should be operable by a single person without a dedicated ops team. | Implemented — this is a design constraint, not an afterthought (see [ADR-0001](adrs/0001-n8n-orchestration.md)) |

## 3. Security requirements

| ID | Requirement | Status |
|---|---|---|
| SEC-1 | Row-Level Security must be enabled on every table holding clinic or patient data. | Implemented |
| SEC-2 | The public-facing anonymous database key must not be able to read data it did not itself insert (e.g., CRM leads). | Implemented — insert-only policy on `crm_leads` for the anon role |
| SEC-3 | Privileged operations (e.g., billing status changes) must go through an explicit, narrowly-scoped stored procedure rather than a direct table write from client code. | Implemented |
| SEC-4 | Trusted backend processes (n8n) may use an elevated database role, but that role must never be exposed to any frontend, exported file, or public repository. | Implemented (operational discipline, not enforced by tooling — flagged as a manual-process risk) |
| SEC-5 | Workflow secrets/credentials must not be stored as plaintext environment variables readable outside the orchestration platform. | Implemented — n8n's built-in credential store is used; environment variables are unavailable in this deployment (see [Architecture § 7](architecture.md#7-security-boundaries)) |
| SEC-6 | Credentials for third-party services must be rotated periodically, not left static indefinitely. | Planned / in progress — `[to validate: rotation cadence not yet formalized]` |
| SEC-7 | No patient-identifying data should appear in a public repository, exported workflow file, or this documentation. | Implemented in this repository by design (anonymization applied throughout) |

## 4. Observability requirements

| ID | Requirement | Status |
|---|---|---|
| OBS-1 | The system must log structured error, warning, and informational events tagged by workflow and node. | Implemented — `logs` table |
| OBS-2 | The system must detect and alert on its own dependency failures (database, WhatsApp provider, human-support inbox) without waiting for a clinic to report an outage. | Implemented — scheduled health checks |
| OBS-3 | The system must detect abnormal operational patterns (e.g., queue backlog, abnormal send volume) and alert proactively. | Implemented |
| OBS-4 | The system must detect and alert on human handoffs that exceed a response-time SLA. | Implemented |
| OBS-5 | Alerting must avoid duplicate/repeated notifications for the same ongoing issue. | Implemented — per-alert-type cooldown logic |
| OBS-6 | The system must have a documented incident runbook covering the most likely failure modes. | Implemented (internal runbook; not included in this public case study for confidentiality) |
| OBS-7 | The system should expose aggregate operational metrics (messages/day, bookings/day, handoff rate) in a dashboard, not only raw logs. | `[to validate: metrics exist informally; a dedicated metrics view is on the roadmap]` |

## 5. Scalability requirements

| ID | Requirement | Status |
|---|---|---|
| SCALE-1 | Every table and uniqueness constraint must be scoped by clinic, not assume a single tenant. | Implemented |
| SCALE-2 | Message processing must be asynchronous (queued) rather than handled synchronously inside the inbound webhook, so a slow AI call cannot block ingestion. | Implemented |
| SCALE-3 | The system must apply a per-clinic rate limit to protect shared infrastructure from a single clinic's traffic spike. | Implemented |
| SCALE-4 | The system must support horizontal growth in clinic count without a schema redesign. | Implemented at current scale; `[to validate: not yet load-tested beyond a single pilot clinic]` |
| SCALE-5 | Billing must scale from a fully manual process to an automated one as clinic count grows, without a full rebuild. | Planned — deliberately deferred, see [ADR-0008](adrs/0008-manual-billing-before-stripe.md) |

## 6. Maintenance requirements

| ID | Requirement | Status |
|---|---|---|
| MAINT-1 | Staging and production must be isolated so that in-progress changes cannot reach real patients. | Implemented — allowlist-gated staging workflows |
| MAINT-2 | Database schema changes must be applied through versioned migrations, not ad hoc edits to the live database. | Partial — most core tables are migration-tracked; `crm_leads`/`crm_notes` are a known exception (see [Architecture § 9](architecture.md#9-known-architecture-debt)) |
| MAINT-3 | Workflow definitions must be exportable/backed up outside the orchestration tool. | Implemented — workflow JSON exports are version-controlled |
| MAINT-4 | The codebase and workflows must document which credentials/dependencies each component requires. | Implemented (internal dependency map; not published here for confidentiality) |
| MAINT-5 | Deploys should be automatable (CI/CD), not exclusively manual. | Planned — currently manual |

## 7. Integration requirements

| ID | Requirement | Status |
|---|---|---|
| INT-1 | The system must send and receive WhatsApp messages through a Cloud API-compatible provider. | Implemented — see [Integrations](integrations.md) |
| INT-2 | The system must integrate with a human-support inbox that clinic staff can use without technical training. | Implemented — Chatwoot |
| INT-3 | The system must integrate with Google Calendar for availability checks and event creation. | Implemented |
| INT-4 | The system must integrate with an LLM provider supporting tool-calling (function calling), not just plain text completion. | Implemented |
| INT-5 | The system must integrate with a speech-to-text provider for voice message support. | Implemented |
| INT-6 | The database layer must expose realtime change notifications for the admin panel to consume, in addition to on-demand queries. | Partial — Realtime is enabled on core tables; the admin panel currently relies primarily on polling |
| INT-7 | The system must support a future migration to the official WhatsApp Business API without a full architecture rewrite. | Design goal — see [ADR-0004](adrs/0004-uazapi-vs-official-whatsapp-api.md) |
