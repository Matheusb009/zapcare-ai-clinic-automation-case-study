# User Stories

> 🇧🇷 [Ler em português](../pt-BR/user-stories.md)

Format: **As a [persona], I want [goal], so that [benefit].** Each story includes acceptance criteria.

## Clinic owner

**US-1**: As a clinic owner, I want patient messages answered even outside business hours, so that I stop losing leads to slow response times.
- Acceptance criteria:
  - Given a patient messages the clinic's WhatsApp at any hour, when consent is already resolved, then Sofia replies without waiting for a staff member to be online.
  - Given the AI is not paused for the clinic, replies happen without manual intervention.

**US-2**: As a clinic owner, I want my calendar to be the single source of truth for availability, so that the AI never double-books or offers a slot that doesn't exist.
- Acceptance criteria:
  - Given a patient requests a time slot, when Sofia checks availability, then the result reflects the actual state of the connected calendar at query time.
  - Given a booking is confirmed, a corresponding calendar event exists.

**US-3**: As a clinic owner, I want to know when my monthly bill is due or overdue, so that I don't get surprised by a service interruption.
- Acceptance criteria:
  - Given my billing status changes to overdue, when I open the admin panel, then I see a visible warning with a way to resolve it.
  - Given I'm within my trial period, I see how many days remain.

## Receptionist / front-desk staff

**US-4**: As a receptionist, I want to receive a conversation with full context when it's handed to me, so that I don't have to ask the patient to repeat themselves.
- Acceptance criteria:
  - Given a conversation is escalated, when it appears in Chatwoot, then it includes an AI-generated summary of what's happened so far.
  - Given I open the conversation, the patient's original messages are still visible above the summary.

**US-5**: As a receptionist, I want the AI to stay out of a conversation once I've taken it over, so that the patient doesn't get two conflicting replies.
- Acceptance criteria:
  - Given a conversation is in human-handled state, when the patient sends another message, then Sofia does not generate a reply.
  - Given I resolve the conversation in Chatwoot, the AI resumes automatically for future messages.

**US-6**: As a receptionist, I want to see only my own clinic's conversations, so that I never see another clinic's patient data.
- Acceptance criteria:
  - Given I'm logged into Chatwoot with my clinic's access, when I browse conversations, then only conversations labeled for my clinic appear.

## Patient / lead

**US-7**: As a patient, I want to send a voice message instead of typing, so that I can reach out however is easiest for me.
- Acceptance criteria:
  - Given I send an audio message, when the system processes it, then I get a reply as if I had typed the same content.

**US-8**: As a patient, I want to be asked for consent before an AI starts messaging me, so that I understand and control how my data is used.
- Acceptance criteria:
  - Given I'm a new contact for a clinic, when I send my first message, then I receive a consent request before any other AI-generated content.
  - Given I decline consent, I stop receiving AI-generated replies.

**US-9**: As a patient, I want to reach a real person when my situation isn't something a bot should handle, so that I feel safe asking anything.
- Acceptance criteria:
  - Given I explicitly ask for a human, when the request is processed, then I'm told a person will take over, and a human receives my conversation with context.

## ZapCare administrator (operator)

**US-10**: As the ZapCare administrator, I want to change a clinic's billing status from a single trusted place, so that billing state can never be altered by an unauthorized action.
- Acceptance criteria:
  - Given I am authenticated as the operator, when I update a clinic's billing status, then the change is applied and logged.
  - Given any other caller attempts the same database operation, it is rejected.

**US-11**: As the ZapCare administrator, I want a CRM view of the sales pipeline, so that I can track leads from first contact to paying customer without leaving the tool I use to run operations.
- Acceptance criteria:
  - Given a lead is created (landing page, diagnostic flow, or manual entry), when I open the CRM view, then the lead appears with its source and current stage.
  - Given I update a lead's stage, the change is reflected immediately in the pipeline view.

**US-12**: As the ZapCare administrator, I want to manually requeue a failed message, so that a transient failure doesn't leave a patient unanswered indefinitely.
- Acceptance criteria:
  - Given a message is in a failed state, when I trigger a requeue from the admin panel, then it re-enters processing and the action is logged.

## Support operator

**US-13**: As the support operator, I want to be alerted automatically when an integration goes down, so that I can respond before a clinic notices.
- Acceptance criteria:
  - Given a scheduled health check detects a failing dependency, when the failure isn't within its cooldown window, then I receive an alert identifying which dependency failed.

**US-14**: As the support operator, I want to see when a human handoff has gone unanswered too long, so that no patient conversation silently stalls.
- Acceptance criteria:
  - Given a handoff conversation exceeds the SLA threshold, when the SLA monitor runs, then I'm alerted and the conversation is flagged.

## Developer / system maintainer

**US-15**: As the developer maintaining this system, I want every workflow's dependencies documented, so that I can debug or hand off a workflow without reverse-engineering it from scratch.
- Acceptance criteria:
  - Given a workflow touches a database table, RPC, or external credential, when I read its documentation, then that dependency is listed.

**US-16**: As the developer maintaining this system, I want staging isolated from production by more than convention, so that an experimental change can never reach a real patient.
- Acceptance criteria:
  - Given a workflow runs in the staging environment, when it processes a message, then it only proceeds for phone numbers on an explicit allowlist.

**US-17**: As the developer maintaining this system, I want database schema changes tracked as migrations, so that the schema's history is reconstructable and reviewable.
- Acceptance criteria:
  - Given a new column or table is needed, when it's added to the live database, then a corresponding migration file exists in version control.
  - *(Known gap: not yet true for `crm_leads`/`crm_notes` — see [Architecture § 9](architecture.md#9-known-architecture-debt).)*
