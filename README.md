# ZapCare — Case Study: AI Automation for Clinics

> 🇧🇷 Leia em português: [README.pt-BR.md](README.pt-BR.md)

**ZapCare** is a real, operating product that automates patient conversations for small and mid-size healthcare clinics in Brazil over WhatsApp, using a conversational AI assistant ("Sofia"), a workflow orchestration layer, and a purpose-built admin panel.

This repository is a **case study**, not the product's source code. It documents the requirements, business rules, architecture, decisions, and trade-offs behind a system that has been designed, built, and operated end-to-end by a single person — covering product thinking, requirements analysis, systems architecture, and hybrid low-code/full-stack engineering.

> Sensitive details (real clinic names, phone numbers, infrastructure IPs, internal database identifiers) have been anonymized or replaced with placeholders throughout this documentation. Architecture, business rules, and technical decisions are real and unmodified. Anything not yet confirmed against the live system is explicitly marked **`[to validate]`**.

---

## Table of contents

- [Overview](#overview)
- [The problem](#the-problem)
- [Target audience](#target-audience)
- [The solution](#the-solution)
- [Key features](#key-features)
- [Tech stack](#tech-stack)
- [Project status](#project-status)
- [Documentation map](#documentation-map)
- [What this project demonstrates](#what-this-project-demonstrates)
- [Contact](#contact)

---

## Overview

Clinics in Brazil overwhelmingly run patient communication through a single shared WhatsApp number, staffed manually by a receptionist. This works until volume grows — then response time collapses, bookings get lost in chat history, and no one has visibility into what's happening across conversations.

ZapCare replaces that manual bottleneck with an AI assistant that talks to patients in natural language on the clinic's existing WhatsApp number, qualifies them, answers frequently asked questions, checks calendar availability, books appointments, and hands off to a human the moment a conversation needs one — while a lightweight admin panel gives the clinic owner visibility and control without touching a terminal or a workflow editor.

## The problem

- Clinics lose leads and patients to slow WhatsApp response times, especially outside business hours.
- A single receptionist juggling WhatsApp, a paper/Google Calendar agenda, and in-person patients cannot scale conversation volume without more staff.
- Off-the-shelf chatbot builders are either too rigid (menu-tree bots that frustrate patients) or too generic (not aware of the clinic's calendar, services, or booking rules).
- Clinic owners have no operational visibility: no queue of pending conversations, no record of what the bot said, no alerting when something breaks silently.

## Target audience

- **Primary**: independent aesthetic, dental, and specialist clinics in Brazil with 1–5 professionals, who already receive bookings over WhatsApp and use Google Calendar/Agenda.
- **Secondary (within the product)**: the clinic's front-desk staff (who take over conversations Sofia escalates) and the clinic owner (who monitors billing, conversations, and appointments through the admin panel).
- **Internal**: ZapCare's own operator (product owner / on-call), who monitors system health, billing status, and the sales pipeline.

## The solution

ZapCare is a layered system, not a single chatbot:

1. **Sofia (the AI assistant)** — an LLM-backed conversational agent (tiered between a fast/cheap model and a stronger model depending on query complexity) that holds context per conversation, answers questions, and calls tools to check availability and book appointments.
2. **An orchestration layer** built on n8n, responsible for receiving WhatsApp webhooks, deduplicating and queueing messages, enforcing LGPD consent before any AI reply, routing to Sofia or to a human, sending reminders, and self-monitoring (health checks, SLA breach alerts, retry/dead-letter handling).
3. **A relational backend** (Supabase/PostgreSQL) holding clinics, conversations, the message queue, appointments, consent records, and structured logs — with row-level security and SECURITY DEFINER RPCs used deliberately to keep write paths narrow.
4. **A human handoff path** into Chatwoot, so that when a conversation needs a person, it becomes a normal support ticket with full context (an AI-generated briefing), not a cold handover.
5. **An admin panel** (React/Vite/Supabase) for the ZapCare operator to manage clinics, billing status, conversations, the processing queue, and a sales CRM — without needing access to n8n or the database directly.
6. **A public landing page** with an interactive lead-qualification flow ("diagnóstico") that feeds the sales CRM.

See [`docs/en/architecture.md`](docs/en/architecture.md) for the full system diagram and component breakdown.

## Key features

- Natural-language WhatsApp conversations, including **voice message transcription** (audio → text → AI reply).
- **LGPD-compliant consent gate**: no AI response is generated for a new patient until explicit consent is recorded.
- **Tiered AI routing**: simple queries go to a faster/cheaper model, complex ones escalate to a stronger model — a deliberate cost/quality trade-off.
- **Tool-using agent**: Sofia calls dedicated sub-workflows to check calendar availability and book appointments, rather than hallucinating scheduling logic in the prompt.
- **Human handoff with briefing**: when Sofia can't or shouldn't continue (explicit request, sensitive case, out-of-scope question, booking error), the conversation moves to Chatwoot with an AI-written summary, and the AI is paused for that conversation until a human resolves it.
- **Automated reminders** for upcoming appointments, with confirmation tracking.
- **Self-healing message queue**: retry with exponential backoff, a dead-letter path after repeated failures, and admin alerting.
- **Operational self-monitoring**: scheduled health checks (integrations, queue backlog, send volume) and SLA-breach detection for stalled human handoffs, both alerting the operator proactively.
- **Multi-clinic architecture**: every table and unique constraint is scoped by `clinic_id`, not assumed single-tenant.
- **Admin panel**: clinic management, live conversation/appointment/queue views, manual billing status control, and a sales CRM (lead pipeline from first contact to paying customer).
- **Audited write actions**: sensitive admin actions (pausing AI, requeueing a failed message, updating billing) are logged, not silent.

## Tech stack

| Layer | Technology |
|---|---|
| Conversational AI | LLM APIs (tiered fast/strong models) via an orchestrated agent with tool-calling |
| Orchestration / workflow engine | [n8n](https://n8n.io) (self-hosted, Docker) |
| Messaging channel | WhatsApp, via a third-party WhatsApp Cloud API provider |
| Human handoff / support inbox | [Chatwoot](https://www.chatwoot.com) (self-hosted) |
| Database / backend | [Supabase](https://supabase.com) (PostgreSQL, Row-Level Security, RPCs, Realtime) |
| Admin panel | React 18 + TypeScript + Vite + Tailwind CSS v4 + React Router, deployed on Vercel |
| Landing page | React 18 + TypeScript + Vite + Tailwind CSS v4, deployed on Vercel |
| Calendar | Google Calendar API (OAuth2) |
| Voice transcription | Speech-to-text API |

## Project status

**Pre-scale / first-paying-clinic stage.** Core conversational, booking, handoff, reminder, and monitoring flows are built and have been validated end-to-end in a staging environment against a real pilot clinic. Billing is intentionally manual at this stage (see [ADR-0008](docs/en/adrs/0008-manual-billing-before-stripe.md)) — automated billing (Stripe, invoicing) is deferred until there are enough paying clinics to justify the integration cost. See [`docs/en/roadmap.md`](docs/en/roadmap.md) for what's shipped versus planned.

## Documentation map

| Document | What it covers |
|---|---|
| [System Requirements](docs/en/system-requirements.md) | Functional, non-functional, security, observability, scalability, maintenance, and integration requirements |
| [Business Rules](docs/en/business-rules.md) | Clinic, conversation, handoff, scheduling, billing, lead-qualification, messaging, and privacy rules |
| [Architecture](docs/en/architecture.md) | Component breakdown, data flow, integration details, Mermaid diagrams |
| [Sofia AI Flows](docs/en/sofia-ai-flows.md) | First contact, qualification, FAQ, voice, booking, objection, handoff, fallback, and follow-up flows |
| [Use Cases](docs/en/use-cases.md) | End-to-end scenarios across patient, clinic, and operator personas |
| [User Stories](docs/en/user-stories.md) | Stories with acceptance criteria per persona |
| [Roadmap](docs/en/roadmap.md) | Shipped MVP, near-term, product, technical, security/compliance, scalability, commercial, and future-SaaS tracks |
| [ADRs](docs/en/adrs/) | 8 technical decision records with context, alternatives, and trade-offs |
| [Integrations](docs/en/integrations.md) | Comparison of WhatsApp Cloud API vs. third-party provider, plus Chatwoot, n8n, Supabase, and the AI layer |
| [Portfolio Positioning](docs/en/portfolio-positioning.md) | Which professional competencies this project demonstrates, and where to see evidence of each |

Every document above has a Portuguese counterpart under [`docs/pt-BR/`](docs/pt-BR/).

## What this project demonstrates

This case study is meant to show, with evidence rather than claims, the ability to:

- Gather and structure requirements for a real operational product, not a toy problem.
- Design business rules that hold up under edge cases (consent races, retries, multi-tenant isolation, SLA breaches).
- Architect a multi-system solution (AI, orchestration, database, messaging, support tooling, admin UI) and justify each technology choice in writing.
- Build both the automation layer (workflows) and the software layer (React/TypeScript admin + landing apps) — a hybrid no-code/low-code + full-stack skill set.
- Operate what was built: monitoring, alerting, incident runbooks, and a real pilot rollout.

See [Portfolio Positioning](docs/en/portfolio-positioning.md) for the full breakdown mapped to specific competencies (requirements analysis, business analysis, systems architecture, product thinking, and full-stack/hybrid development).

## Contact

**Matheus Bittencourt** — [contact-email]

This repository is a portfolio artifact. It intentionally excludes credentials, production infrastructure details, and any real patient or prospect data.
