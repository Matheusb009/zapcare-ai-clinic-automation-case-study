# ADR-0002: Use Supabase as the Backend/Database Layer

> 🇧🇷 [Ler em português](../../pt-BR/adrs/0002-supabase-como-backend.md)

- **Status**: Accepted
- **Date**: `[to validate: exact decision date]` — early build phase of the project

## Context

The system needs a relational database (structured, relational data: clinics, conversations, appointments, a message queue) that can be written to by n8n workflows (trusted backend), and also needs to be readable/writable directly by two frontend applications (admin panel, landing page) without necessarily building and hosting a separate REST/GraphQL API for each.

## Decision

Use Supabase (managed PostgreSQL with Row-Level Security, auto-generated REST/RPC access, and Realtime subscriptions) as the single backend for both the automation layer and the two web applications.

## Alternatives considered

- **Self-managed PostgreSQL + a custom API layer (e.g., Express/FastAPI)**: full control over the API surface, but requires building, hosting, and securing a separate backend service, plus its own auth layer — meaningful extra scope for a solo build.
- **Firebase/Firestore**: comparable "backend-as-a-service" convenience, but a document model is a worse fit for the genuinely relational data here (clinics → conversations → appointments, with real foreign keys and constraints like `(phone, clinic_id)` uniqueness).
- **Supabase (chosen)**: relational Postgres (a better fit for the data shape), Row-Level Security enforced at the database layer (so the same security rules apply regardless of which client connects), auto-generated APIs eliminating the need for a hand-built REST layer, and Realtime for live-updating UI where needed.

## Consequences

**Positive**:
- No separate backend API needed for either frontend — both talk to Supabase directly, cutting real build time.
- RLS pushes the security model into the database itself, so a bug in frontend code cannot expose data a policy doesn't allow, regardless of which app the bug is in.
- `SECURITY DEFINER` RPCs give a clean pattern for narrow, auditable privilege escalation (e.g., the billing-status RPC) without giving broad write access to any client role.

**Negative**:
- RLS policy correctness becomes the entire security model for public writes (e.g., the landing page's anon key) — a misconfigured policy is a direct data exposure risk, concentrated in one layer rather than distributed across API-level validation.
- Schema evolution discipline (migrations for every change) is a process the team has to enforce itself; it has already slipped once (`crm_leads`/`crm_notes` created directly in the live database — see [Architecture § 9](../architecture.md#9-known-architecture-debt)).
- Vendor dependency on Supabase specifically for Realtime/RLS/RPC conventions — migrating off would mean re-implementing meaningful backend logic, not just swapping a connection string.

## Related

[Architecture § 3.3](../architecture.md#33-data-layer-supabase--postgresql), [Integrations § 4](../integrations.md#4-supabase-database--backend)
