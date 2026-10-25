# AI Trader India — AI Signal Automation Platform

🌐 **Live website:** [aitraderindia.online](https://aitraderindia.online/)  ·  📢 **IPO alerts:** [t.me/AITraderIndiaIPO](https://t.me/AITraderIndiaIPO)

> An end-to-end, AI-powered signal platform **designed, built and operated entirely solo** — from system architecture and backend engineering to payment workflows and production monitoring.

![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white) ![Node.js](https://img.shields.io/badge/Node.js-5FA04E?logo=nodedotjs&logoColor=white) ![MongoDB](https://img.shields.io/badge/MongoDB-47A248?logo=mongodb&logoColor=white) ![Telegram](https://img.shields.io/badge/Telegram_Bot_API-26A5E4?logo=telegram&logoColor=white) ![Automation](https://img.shields.io/badge/Workflow_Automation-FF6951)

**Role:** Sole builder — system design · backend · AI/automation · payments · ops · web  
**Timeline:** Jan 2026 – present  
**Status:** Live in production

> ℹ️ The product delivered informational/educational market signals. This repository documents the **engineering and automation** behind it, not the trading content itself.

---

## Overview

AI Trader India is a production platform that generates, formats and delivers real-time trading signals to subscribers over Telegram, manages paid access, and monitors itself — all with **no manual intervention** in the day-to-day loop.

I built every layer myself: the signal engine, the delivery bot, the subscription and access system, a deliberately fraud-resistant payment flow, and a monitoring pipeline. The goal was to prove that one person could design and automate an entire subscription operation so it effectively runs itself.

## The problem

Running a paid signal service by hand doesn't scale and breaks easily:

- **Timing is everything** — signals must be generated, formatted and delivered on time, every time. A human can't be awake 24/7.
- **Access must be controlled** — only paying members should receive premium signals, and access has to be granted and revoked reliably.
- **Payments attract fraud** — manual UPI payments invite fake screenshots and chargebacks.
- **Failures are silent** — if a bot or job dies at 2 a.m., you need to know immediately, not the next morning.

The challenge was to turn all of that into software that handles itself.

## Goals

1. Fully automate signal generation → formatting → delivery with zero manual steps.
2. Make paid access self-managing and tamper-resistant.
3. Design a payment flow that minimises fraud without a heavy payment-gateway dependency.
4. Build observability so the system reports its own health and failures.
5. Own the full lifecycle solo — and keep it maintainable.

## System architecture

```
                 ┌───────────────────────────┐
                 │      AI Signal Engine       │   Python + Node.js
                 │  generate · format · rank   │
                 └──────────────┬──────────────┘
                                │ structured signal
                                ▼
   ┌────────────────────────────────────────────────────────┐
   │                  Delivery & Bot Layer                    │
   │              Telegram Bot API (push to users)            │
   └───────────────┬───────────────────────┬─────────────────┘
                   │                         │
                   ▼                         ▼
        ┌────────────────────┐    ┌────────────────────────┐
        │  Subscription &     │    │  Payment Verification   │
        │  Access Control     │◄──►│  (UPI, fraud checks)    │
        └─────────┬──────────┘    └────────────────────────┘
                  │
                  ▼
            ┌──────────┐
            │ MongoDB  │  members · access state · signal history
            └──────────┘
                  │
                  ▼
        ┌────────────────────────────┐
        │   Monitoring & Logging       │  Discord webhooks
        │   status · errors · alerts   │
        └────────────────────────────┘
```

### 1. AI Signal Engine · `Python` `Node.js` `AI/LLM`

The core service. It generates real-time entry, exit and stop-loss signals, then formats them into a clean, consistent message structure before they ever reach a user. Built to run continuously as a scheduled, always-on process.

### 2. Delivery & Bot Layer · `Telegram Bot API`

A Telegram bot handles all member-facing communication — pushing signals to the right channels/users, responding to commands, and acting as the interface between the automation and the subscriber.

### 3. Subscription & Access Control · `MongoDB`

Member records, plan state and access rights live in MongoDB. Access is granted and tracked programmatically so premium content only reaches paid, active members.

### 4. Payment Verification (fraud-prevention)

A deliberate design choice: payments go through a **manual UPI verification step** rather than blind auto-approval. This was a conscious control to defend against **fake payment screenshots and chargebacks** — trading reliability of verification against pure convenience. It's the kind of trade-off I'd now revisit with a gateway webhook (see roadmap).

### 5. Scheduling & Distribution · `REST APIs`

Content scheduling and distribution workflows automate *when* and *how* messages go out, so the platform grew through organic reach and targeted ad campaigns without manual posting.

### 6. Monitoring & Logging · `Discord Webhooks`

The system watches itself. Status updates, errors and alerts are routed to a Discord channel in real time, giving a lightweight, zero-cost observability layer for record-keeping, debugging and ongoing maintenance.

## Data flow

1. The **signal engine** produces a signal and formats it.
2. The **bot layer** pushes it to subscribers via the Telegram Bot API.
3. **Access control** ensures only active, paid members receive premium signals.
4. New payments hit the **verification flow**; once confirmed, access is updated in **MongoDB**.
5. Every meaningful event (and every failure) is logged to **Discord** for monitoring.

## Phase 2 — IPO desk: website + alert bot (Oct 2026)

A second product on the same foundations: a fast, search-friendly website ([aitraderindia.online](https://aitraderindia.online/)) and a Telegram bot that keep Indian IPO information current without manual work.

**What it does**
- Tracks every open and upcoming IPO and keeps one page per IPO up to date: dates, price band, lot size, allotment links and listing result.
- A home-page calendar and IPO board, rebuilt automatically whenever a number changes.
- Telegram posts for each stage of an IPO (new, opening day, last day, allotment, listing), edited in place when a detail is corrected instead of posting twice.
- SME-specific rules handled correctly (for example the 2-lot minimum bid) and shown in rupees.
- Shows the grey market premium (GMP) for every IPO that has not yet listed, with the estimated listing price and the gain on one lot (an SME's smallest bid is 2 lots), coloured green for a gain and red for a loss.
- Links each IPO to its own allotment-status page at the registrar as soon as allotment day arrives, on the website and in the Telegram post.
- Keeps the data fresh on a fixed 15-minute loop, one IPO per minute, from the first IPO in the list to the last.

**Engineering highlights**
- **Static generation, no database on the web path.** A Python generator writes plain HTML; nginx serves it. Pages are written atomically per file, so a visitor never sees a half-built page. Works with JavaScript off.
- **Self-updating schedule.** Data is re-checked every 15 minutes; each IPO then takes its turn one minute apart, so the load is spread evenly and a failure in one step cannot block the others.
- **Polite and fault-tolerant by design.** Every outside read respects robots.txt, backs off when refused, and reports a changed page layout instead of guessing. One failing source never stops the rest.
- **Safe by default.** Secrets only in environment files, never logged; owner commands for status and manual overrides; admin alerts on anything unusual.
- **Tested like a product.** 470+ automated tests (bot logic, page output, real nginx behaviour, 18 phone widths for layout), plus an SEO/answer-engine audit script that runs on every build.
- **Traceable decisions.** Cross-checks IPO dates, price bands and lot sizes between independent sources and alerts me on any mismatch rather than silently trusting one.

> ℹ️ This repository documents the **engineering**, not the market content. Nothing here is investment advice.

## Tech stack & why

| Layer          | Tech                                | Why                                                 |
|----------------|-------------------------------------|-----------------------------------------------------|
| Signal logic   | **Python**                          | Fast to iterate on data/logic-heavy work            |
| Services & bot | **Node.js**                         | Event-driven, great for always-on bot I/O           |
| Data           | **MongoDB**                         | Flexible schema for evolving member/signal data     |
| Delivery       | **Telegram Bot API**                | Where the audience already was                      |
| Integration    | **REST APIs**                       | Glue between scheduling, access and delivery        |
| Observability  | **Discord Webhooks**                | Real-time alerts with zero infra overhead           |
| Website        | **Python static generator + nginx** | Fast, cacheable, no moving parts on the public path |
| Scheduling     | **APScheduler + pm2**               | Always-on jobs with automatic restart               |

## Key engineering decisions

- **Automation-first:** every recurring task was designed to run without me. If a step needed a human daily, it got redesigned.
- **Fraud-aware payments:** chose verification robustness over frictionless checkout — a reasoned trade-off, not an accident.
- **Cheap, real observability:** Discord webhooks gave production-grade visibility without standing up a monitoring stack.
- **Single-owner maintainability:** kept the moving parts few and well-separated so one person could operate the whole thing.

## What I learned

- Shipping a *system* is very different from writing a script — state, failure modes and money all have to be handled.
- Good monitoring is what lets a solo builder sleep; design for failure from day one.
- The boring parts (access control, payment edge cases) are where real product trust is won or lost.

## Roadmap / what I'd build next

- Move payment verification to a **payment-gateway webhook** for automatic reconciliation
- ✅ Automated tests around the bot and the website (470+); next: run them automatically on every commit (CI)
- Build an **admin dashboard** for members, payments and signal history
- Containerise and add health-check-based auto-restart

---

**Built by Om Gondaliya** — Product Designer & builder who ships the system behind the product.  
📧 theomgondaliya@gmail.com  ·  🌐 [aitraderindia.online](https://aitraderindia.online/)  ·  🔗 [linktr.ee/omgondaliya](https://linktr.ee/omgondaliya)  ·  💼 [linkedin.com/in/omgondaliya](https://linkedin.com/in/omgondaliya)
