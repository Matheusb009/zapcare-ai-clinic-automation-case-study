# ADR-0008: Manual Billing Before Automated Billing (Stripe)

> 🇧🇷 [Ler em português](../../pt-BR/adrs/0008-cobranca-manual-antes-do-stripe.md)

- **Status**: Accepted
- **Date**: `[to validate: exact decision date]` — documented alongside the billing implementation

## Context

The product needs a way to bill clinics and track trial/active/overdue/cancelled state. At low clinic counts, integrating a payment platform (Stripe or a local equivalent) means real engineering and compliance overhead — webhook handling, reconciliation, invoicing, failure/dispute handling — before there's enough billing volume to justify it.

## Decision

Run billing manually: the operator tracks payment (via PIX/boleto, outside the product) and updates each clinic's billing status through a single restricted admin action. Explicitly defer Stripe, automated invoicing, recurring auto-billing, and a customer-facing billing portal until clinic count justifies the integration cost.

## Alternatives considered

- **Integrate Stripe (or a local billing provider) from the start**: automated, scales cleanly, but represents real upfront engineering effort (webhook handling, payment-state reconciliation, failure/retry logic, likely a compliance review) for a stage where there might be a handful of clinics — high cost relative to the number of transactions it would actually process at this point.
- **No billing status tracking at all (informal, outside the system)**: zero engineering cost, but leaves the admin panel unable to reflect a clinic's real payment state, and removes the ability to gate/warn on overdue accounts inside the product experience.
- **Manual billing status, tracked in-product, enforced by a single restricted RPC (chosen)**: gets the product-facing benefits (trial banners, overdue warnings, a queryable billing_status per clinic) without the cost of a payment integration, while keeping the write path narrow and auditable so "manual" doesn't mean "insecure."

## Consequences

**Positive**:
- Avoided real integration cost and compliance surface area before there was billing volume to justify it — a deliberate sequencing decision, not a shortcut taken carelessly (it's documented, with a defined trigger for revisiting: enough paying clinics).
- The billing-status RPC pattern (single restricted write path) means "manual" billing still has the same security discipline as anything else in the system — see [Architecture § 7](../architecture.md#7-security-boundaries).
- Clinics still get a real product experience around billing (trial countdown, overdue warning) despite payment collection itself happening outside the system.

**Negative**:
- Every billing status change requires the operator's manual attention — this does not scale past a relatively small number of clinics without becoming an operational bottleneck itself.
- No automated dunning, no self-service invoice history for the clinic, no proration logic — all explicitly out of scope for now (see [Business Rules § 5](../business-rules.md#5-billing--trial-rules)).
- The eventual migration to automated billing will require real design work (mapping the current manual states onto Stripe's subscription lifecycle) — deferred, not eliminated, cost.

## Related

[Business Rules § 5](../business-rules.md#5-billing--trial-rules), [Roadmap](../roadmap.md), [Use Case UC-7](../use-cases.md#uc-7--admin-changes-a-clinics-billing-status)
