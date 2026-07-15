# Use Cases

> 🇧🇷 [Ler em português](../pt-BR/casos-de-uso.md)

Each use case follows: **Actor → Trigger → Preconditions → Main flow → Outcome → Related rules.**

## UC-1 — New patient makes first contact

- **Actor**: Patient (new phone number to this clinic)
- **Trigger**: Patient sends any WhatsApp message to the clinic's number.
- **Preconditions**: Clinic is active and AI is not clinic-wide paused.
- **Main flow**:
  1. Receptor receives the webhook, resolves the clinic, applies rate limiting, deduplicates by message ID.
  2. System detects no consent record exists for this phone/clinic pair.
  3. System sends an LGPD consent request instead of routing to Sofia.
  4. Patient accepts → consent recorded → conversation proceeds normally from the next message onward.
- **Outcome**: A compliant conversation record exists before any AI-generated content is sent to a new patient.
- **Related rules**: [Business Rules § 2, § 8](business-rules.md); [Sofia AI Flows § 1](sofia-ai-flows.md#1-first-message-flow).

## UC-2 — Lead asks for pricing

- **Actor**: Patient / prospective patient
- **Trigger**: A message asking about the price of a service.
- **Preconditions**: Consent already recorded; conversation in AI-active state.
- **Main flow**:
  1. Sofia checks whether pricing for the asked service is configured for this clinic.
  2. If configured, Sofia answers with the configured price/range.
  3. If not configured, Sofia does not invent a number — it either says pricing depends on evaluation, or escalates if the patient insists on an exact figure Sofia cannot provide.
- **Outcome**: The patient never receives a price the clinic didn't authorize.
- **Related rules**: [Business Rules § 2](business-rules.md#2-conversation--service-rules); [Sofia AI Flows § 6](sofia-ai-flows.md#6-objection-handling-flow).

## UC-3 — Lead sends a voice message

- **Actor**: Patient
- **Trigger**: Patient sends an audio message instead of text.
- **Preconditions**: Consent already resolved.
- **Main flow**:
  1. Receptor detects the message is audio, downloads the file.
  2. Audio is transcribed to text via the speech-to-text integration.
  3. The transcription is treated as the message body and enters the normal processing flow.
  4. Sofia replies as if the patient had typed the message.
- **Outcome**: Voice and text patients get functionally identical service.
- **Related rules**: [Sofia AI Flows § 4](sofia-ai-flows.md#4-voice-message-audio-flow).

## UC-4 — Lead wants to book an evaluation

- **Actor**: Patient
- **Trigger**: Patient asks to schedule an appointment/evaluation.
- **Preconditions**: Consent resolved; clinic calendar configured.
- **Main flow**:
  1. Sofia calls the availability tool for the requested service/professional/date range.
  2. Available slots are presented to the patient.
  3. Patient confirms a slot.
  4. Sofia calls the booking tool: Calendar event created, `appointments` row written, confirmation sent.
  5. Alternate path: if booking fails, conversation is handed to a human instead of failing silently.
- **Outcome**: A real, calendar-backed appointment exists, or the patient is in a human's queue with full context — never a dead end.
- **Related rules**: [Sofia AI Flows § 5](sofia-ai-flows.md#5-scheduling--booking-flow); [Business Rules § 4](business-rules.md#4-scheduling--hours-rules).

## UC-5 — Lead asks for a human

- **Actor**: Patient
- **Trigger**: Patient explicitly asks to talk to a person (or the system detects a sensitive/out-of-scope case).
- **Preconditions**: None beyond an active conversation.
- **Main flow**:
  1. Handoff workflow triggers: AI briefing generated, AI paused for this conversation.
  2. Chatwoot contact/conversation found or created, briefing posted, clinic label applied.
  3. Internal team notified; patient told they'll be attended by a person.
  4. Clinic staff pick up the conversation in Chatwoot with full context already available.
- **Outcome**: No cold handoffs — a human always starts with context, never a blank chat.
- **Related rules**: [Business Rules § 3](business-rules.md#3-human-handoff-rules); [Sofia AI Flows § 7](sofia-ai-flows.md#7-human-handoff-flow).

## UC-6 — Clinic follows its own conversations

- **Actor**: Clinic front-desk staff
- **Trigger**: Staff logs into Chatwoot to check on patient conversations.
- **Preconditions**: Staff has Chatwoot access scoped to their clinic's label.
- **Main flow**:
  1. Staff sees only conversations labeled for their clinic, including AI-handled history and any handed-off conversations.
  2. Staff can take over a conversation, reply, and mark it resolved (which reactivates the AI automatically).
- **Outcome**: Clinic staff get operational visibility without needing admin-panel or database access.
- **Related rules**: [Architecture § 3.4](architecture.md#34-human-handoff-chatwoot).

## UC-7 — Admin changes a clinic's billing status

- **Actor**: ZapCare operator
- **Trigger**: A clinic's trial ends, a payment is confirmed, or a payment is overdue.
- **Preconditions**: Operator is authenticated in the admin panel.
- **Main flow**:
  1. Operator opens the clinic's row in the Clinics view.
  2. Operator selects the new billing status (e.g., trial → active).
  3. The panel calls the restricted billing RPC, which verifies the caller is the authorized operator account before applying the change.
  4. The action is logged.
  5. If the new status is `overdue`, the clinic's own admin view surfaces a warning banner.
- **Outcome**: Billing state changes are deliberate, attributable, and restricted to one authorized path.
- **Related rules**: [Business Rules § 5](business-rules.md#5-billing--trial-rules); [Architecture § 7](architecture.md#7-security-boundaries).

## UC-8 — Operator monitors logs and failures

- **Actor**: ZapCare operator
- **Trigger**: A scheduled health check, SLA monitor, or retry scheduler alert fires — or the operator proactively checks the Queue/Monitoring views.
- **Preconditions**: None.
- **Main flow**:
  1. Operator receives an alert (e.g., queue backlog, integration unreachable, SLA breach) via the alerting channel.
  2. Operator opens the admin panel's Monitoring/Queue view to see the specific failing item(s) and their logged error context.
  3. Operator resolves the underlying issue (e.g., reconnect an integration) and, if needed, manually requeues a failed message from the panel — an action that is itself logged.
- **Outcome**: Failures are visible and actionable within minutes, not discovered days later from a clinic complaint.
- **Related rules**: [System Requirements § 4 (Observability)](system-requirements.md#4-observability-requirements); [Architecture § 6](architecture.md#6-reliability-mechanisms).

## UC-9 — System detects an integration error

- **Actor**: System (Health Check workflow)
- **Trigger**: Scheduled run (every 5 minutes).
- **Preconditions**: None — this runs unconditionally on schedule.
- **Main flow**:
  1. Health Check pings the database, WhatsApp provider, and Chatwoot.
  2. It also checks for queue backlog and abnormal daily send volume.
  3. If any check fails or crosses a threshold, and the corresponding alert isn't in cooldown, an alert is sent to the operator.
  4. Each failure type has its own cooldown so a persistent outage produces one alert stream, not a flood.
- **Outcome**: The system reports its own degradation before a patient or clinic notices — proactive rather than reactive monitoring.
- **Related rules**: [Architecture § 6](architecture.md#6-reliability-mechanisms); [System Requirements § 4](system-requirements.md#4-observability-requirements).
