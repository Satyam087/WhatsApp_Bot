# Campus Critique WhatsApp Bot — Project Documentation & Submission Suite

This repository contains the complete product, engineering, and conversational specifications for the **Campus Critique WhatsApp Advisory Agent**, engineered for Indian engineering aspirants.

---

## 🎓 Primary Academic & Faculty Review Documents

For project evaluation, technical reviews, and faculty defense, start with these three core documents:

| # | Document | Target Audience & Contents |
|---|---|---|
| 1 | **[`PRD.md`](file:///Users/true.man/Whatsapp_bot/PRD.md)** | **Project Requirements Document (PRD):** Indian engineering admission landscape, problem analysis (~13–15L unplaced JEE students), product thesis, user personas, 7-stage conversational framework, functional features, and DPDP Act / Tele-MANAS compliance. |
| 2 | **[`TRD.md`](file:///Users/true.man/Whatsapp_bot/TRD.md)** | **Technical Requirements Document (TRD):** System architecture, webhook ingestion & 4s debounce pipeline, 5-rail guardrail engine (TypeScript), mathematical models (rank, loan caps, EMIs), full PostgreSQL DDL schema (`wa`), Promptfoo CI evals, and unit economics. |
| 3 | **[`CONVERSATION_WALKTHROUGH.md`](file:///Users/true.man/Whatsapp_bot/CONVERSATION_WALKTHROUGH.md)** | **Conversational Simulation & Test Suite:** Full 11-turn golden path dialogue ("Aarav") annotated with state transitions, rails, data layers, and grounding checks, plus 4 edge-case scenarios (unlisted colleges, loan reality check, Tele-MANAS crisis intercept, prompt injection). |

For the browser-ready submission, open **[`index.html`](index.html)** first. It links to the consolidated faculty report (`report.html`), the detailed PRD (`prd.html`), the detailed TRD (`trd.html`), and the standalone conversation walkthrough (`conversation.html`).

---

## 📚 Foundational Research & Internal Architectural Records

These documents provide deep architectural logs, design decisions, and economic calculations:

| Document | Description |
|---|---|
| **[`campus-critique-bot.html`](file:///Users/true.man/Whatsapp_bot/campus-critique-bot.html)** | One-page consolidated visual executive brief with diagrams and data tables. |
| **[`PRODUCT_THESIS.md`](file:///Users/true.man/Whatsapp_bot/PRODUCT_THESIS.md)** | Market analysis, college fee & placement audit, and the rationale for disinterest in admissions. |
| **[`DECISIONS.md`](file:///Users/true.man/Whatsapp_bot/DECISIONS.md)** | Authoritative log of locked architectural decisions (D1–D12). |
| **[`ENGINEERING_ARCHITECTURE.md`](file:///Users/true.man/Whatsapp_bot/ENGINEERING_ARCHITECTURE.md)** | Engineering research notes on NeMo guardrails, Vercel AI SDK, Promptfoo, and Langfuse. |
| **[`DATA_STRATEGY.md`](file:///Users/true.man/Whatsapp_bot/DATA_STRATEGY.md)** | Why RAG fails on tabular cutoffs, and the 3-layer data model (Compute, SQL, RAG). |
| **[`MODEL_STRATEGY.md`](file:///Users/true.man/Whatsapp_bot/MODEL_STRATEGY.md)** | Model portfolio, evaluation of TypeSafe Jev, and prompt decomposition. |
| **[`WHATSAPP_COSTS.md`](file:///Users/true.man/Whatsapp_bot/WHATSAPP_COSTS.md)** | Granular messaging unit economics under Meta's Cloud API pricing. |
| **[`SAMPLE_CONVERSATION.md`](file:///Users/true.man/Whatsapp_bot/SAMPLE_CONVERSATION.md)** | Initial conversational spec notes. |

---

## ⚠️ Critical Operational Prerequisites

1. **Meta Payment Method on WABA:** Service messages require a valid payment method on file in Meta Business Manager.
2. **Dedicated Supabase Project:** Requires a dedicated PostgreSQL instance hosting the `wa` schema to ensure DPDP Act compliance and isolation from web applications.
