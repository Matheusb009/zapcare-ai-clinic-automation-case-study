# Business Rules

> 🇧🇷 [Ler em português](../pt-BR/regras-de-negocio.md)

These rules are extracted from the live schema constraints, RPC logic, and internal process documentation — they are the actual constraints the system enforces or the operator follows, not aspirational policy. Anything not enforced in code is labeled accordingly.

## 1. Clinic rules

- Every clinic is an independent tenant: its own WhatsApp number, business hours, AI prompt, plan, calendar mapping, and rate limit. No data is ever shared across `clinic_id`.
- A clinic can be deactivated (`active = false`) without deleting its data — deactivation stops the AI and reminders but preserves history for the retention period below.
- A clinic's AI can be paused at the clinic level (all conversations) or at the individual conversation level (one patient only) — these are two different controls for two different situations (e.g., "clinic is closed for a holiday" vs. "this one patient needs a human").
- A clinic has a configurable rate limit (messages per minute) enforced before enqueueing — this protects shared infrastructure if one clinic's traffic spikes abnormally (e.g., a broadcast campaign the clinic sent on its own).

## 2. Conversation / service rules

- A conversation is uniquely identified by `(phone, clinic_id)` — the same phone number talking to two different clinics produces two independent conversations with independent context, not one conversation bleeding across tenants.
- No AI reply is ever generated for a conversation with unresolved LGPD consent — this is a hard gate in the processing Worker, evaluated before every AI call, not a prompt instruction the model could ignore.
- Every inbound message is deduplicated by its provider-assigned message ID before processing — a WhatsApp delivery retry must never produce two AI replies to the same message.
- Sofia must not: give a clinical diagnosis, promise a specific procedure outcome, quote a price not explicitly configured for that clinic, or continue answering once a conversation is in human-handled state.
- Conversation context is bounded (windowed memory + periodic summarization), not stored and replayed in full forever — this is both a cost control and a rule about what "context" means operationally.

## 3. Human handoff rules

- A conversation must be handed to a human when any of: the patient explicitly asks for a human, the message content is flagged as sensitive, the question falls outside the AI's configured scope, or an automated booking operation fails.
- The moment a handoff triggers, the AI is paused for that conversation immediately — there is no window where both the AI and a human could reply to the same patient.
- Every handoff must arrive in the human's inbox (Chatwoot) with an AI-generated summary of the conversation so far — a human is never expected to scroll raw chat history to get context.
- A handoff conversation is labeled by clinic, so the receiving clinic's staff only see their own patients even inside a shared support tool.
- The AI resumes automatically once a human marks the conversation resolved — handoff is a loop back to automation by default, not a permanent hand-off.
- A conversation abandoned mid-handoff past a defined inactivity cutoff is eligible for automated reactivation: the AI sends a generated follow-up and resumes ownership, rather than leaving the patient permanently stranded in an unanswered human queue.

## 4. Scheduling / hours rules

- Availability is always checked against the clinic's actual calendar before a booking is confirmed — the AI does not book speculatively and correct after the fact.
- A booking failure (e.g., calendar API error) routes to human handoff rather than silently failing or retelling the patient to "try again later" with no follow-up.
- Appointment reminders are sent on a fixed schedule relative to the appointment (a next-day window and a same-day window), and each reminder's delivery/confirmation is tracked per appointment, not assumed.
- Appointment status follows a defined lifecycle (pending → scheduled → confirmed → completed, or → cancelled), and cancellations require a reason to be recorded.

## 5. Billing / trial rules

- A new clinic starts in `trial` billing status with a defined trial period; billing status transitions (`trial → active → overdue → cancelled`) are the operator's decision, made through a single restricted admin action — not automatic based on payment webhooks, because billing is currently a manual process by design (see [ADR-0008](adrs/0008-manual-billing-before-stripe.md)).
- Only the operator's account can execute a billing status change — this is enforced at the database layer (a `SECURITY DEFINER` RPC checks the caller), not just hidden in the UI.
- Overdue billing triggers a visible warning inside the clinic's own admin view, and cancellation follows a defined process: no penalty, no proration, data retained for a minimum period after cancellation rather than deleted immediately.
- Automated recurring billing, invoicing, and a self-service billing portal are explicitly out of scope until there are enough paying clinics to justify the integration — a deliberate "manual until it hurts" sequencing decision, not an oversight.

## 6. Lead qualification rules

- A lead enters the CRM from one of three sources: the public landing page's lead form, the interactive "diagnóstico" qualification flow, or manual entry by the operator — each is tagged with its originating source.
- A lead progresses through a defined pipeline stage sequence (new → qualified → contacted → demo scheduled → pilot active → customer, or → lost with a recorded reason) — stage changes are not free-text status updates.
- The public-facing lead form and qualification flow can only create a lead, never read the pipeline back — public traffic must not be able to enumerate or scrape the sales pipeline.
- Moving a lead from "qualified" to "demo scheduled" via the public diagnostic flow happens through a restricted stored procedure, not a direct table write, to keep the public-write surface as narrow as possible even though the flow needs some write access.
- Ideal-customer criteria used to prioritize outbound qualification (not a hard system constraint, a process rule): 1–5 professionals, already booking via WhatsApp, uses Google Calendar/Agenda, and the owner is the direct decision-maker.

## 7. Messaging and limit rules

- Each clinic has an enforced per-minute rate limit on outbound processing to protect shared infrastructure — traffic beyond the limit queues rather than firing immediately.
- Repeated processing failures for the same message are retried with exponential backoff up to a fixed attempt ceiling, after which the item is moved to a dead-letter state and flagged to the operator instead of retrying indefinitely.
- Abnormally high daily send volume for a clinic triggers an operational alert — this is treated as a signal worth investigating (misconfiguration, abuse, or a runaway loop), not just capacity headroom.
- Voice messages are always transcribed before being treated as conversation input — the AI never processes raw audio directly as part of its reasoning step.

## 8. Security and privacy rules

- Every patient message requires recorded consent before the AI may reply; consent records are immutable and are never deleted, since they serve as legal proof of consent, not just operational state.
- Operational logs are retained for a limited period (short-lived by default) except when explicitly flagged as an audit record.
- Conversation history and working memory are purged after a defined period of patient inactivity; appointment records are anonymized (not deleted outright) after a longer retention window, balancing legal/record-keeping needs against data minimization.
- No frontend application (admin panel or landing page) ever holds elevated database privileges — only the trusted backend orchestration layer does, and only for the operations that require it.
- No production credential, patient phone number, or real clinic identity may appear in an exported workflow file, this documentation, or any public repository — this rule shaped how this very case study was written (see the anonymization note in the [README](../../README.md)).
