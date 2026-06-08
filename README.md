# VIZZ — WhatsApp AI for a Plastic Surgery Clinic in Brazil

> **"She knows the procedures, she asks for your name, she escalates to the team when she doesn't know — and she feels human."**

VIZZ is a production WhatsApp AI assistant built for **Dr. Mateus Vizzotto's plastic surgery clinic** in Porto Alegre, Brazil. She handles the full patient journey — from first contact to appointment scheduling — entirely over WhatsApp, in Portuguese, with a warm and humanized tone that patients often mistake for a real person.

Built with **vibe coding** using [Replit](https://replit.com) + [Claude (Anthropic)](https://anthropic.com) as the primary development tools.

---

## What is VIZZ?

VIZZ is a WhatsApp-native AI consultant that:

- **Qualifies leads** — identifies interest, procedure, payment preference
- **Collects patient data** — name, CPF (Brazilian tax ID), email, phone
- **Answers clinical questions** — procedures offered, prices, pre-op requirements
- **Escalates intelligently** — when she doesn't know something, she messages the clinic admin on WhatsApp and waits for the answer before responding to the patient
- **Schedules consultations** — guides patients through payment (PIX) and pre-schedules with the team
- **Follows up proactively** — re-engages leads that went silent after 24h, 2 days, 7 days, 30 days, and 90 days

---

## How to Talk to VIZZ

VIZZ runs on the clinic's official WhatsApp Business number:

**+55 51 2391-7004** (Consultora Vizz)

Simply send any message about plastic surgery procedures — she responds in real-time, 24/7.

**Example openers that work:**
- "Oi, tenho interesse em fazer uma lipo"
- "Quero saber sobre mamoplastia"
- "Quanto custa uma consulta com o Dr. Mateus?"
- "Fiz uma lipoaspiração há 2 anos e quero uma revisão"

---

## The Goal

The clinic was managing all patient inquiries manually — a mix of WhatsApp messages, phone calls, and Instagram DMs — with no consistent follow-up and many leads getting lost.

**VIZZ was built to:**

1. **Never miss a lead** — responds instantly, 24/7, even on weekends
2. **Qualify before the consultation** — ensures the patient understands the value, the price, and is ready to pay the R$ 500 consultation fee
3. **Reduce no-shows** — sends reminders and confirms attendance the day before
4. **Generate academic data** — all conversations feed a Health Economics research project on no-show prediction in elective surgery

---

## Impact Numbers

| Metric | Value |
|--------|-------|
| Total leads handled | **121+** |
| Conversations actively tracked | **125 in database** |
| Follow-up cadence steps | **5 (24h → 2d → 7d → 30d → 90d)** |
| Response time | **< 3 seconds** |
| Uptime | **24/7** |
| Languages supported | **Portuguese, English, Spanish** |
| Procedures in knowledge base | **12+** |

> The clinic went from losing ~60% of leads due to slow response to having every single inquiry answered within seconds.

---

## The Humanization Approach

This was the hardest part. The clinic's specific request: *"She cannot feel like a bot."*

### What makes VIZZ feel human:

**1. She uses your name**
The moment a patient mentions their name (or she extracts it from context), she uses it naturally throughout the conversation.

**2. She admits when she doesn't know**
Instead of hallucinating, she says:
> "Que boa pergunta! Vou confirmar com o Dr. Mateus e já te retorno — não quero te dar uma informação sem ter certeza."

Then she literally messages the clinic admin on WhatsApp, waits for the answer, and returns to the patient.

**3. She respects timing**
She greets with "Bom dia", "Boa tarde", or "Boa noite" based on the actual time in Brasília.

**4. She never repeats herself**
A state machine tracks exactly where each patient is in the conversation — she never re-asks for information she already has.

**5. Zero emojis**
A deliberate design choice. Real consultants at this clinic type without emojis. VIZZ does the same.

**6. She recognizes foreign patients**
If the name looks foreign or the patient mentions being abroad, she accepts a passport number instead of CPF and adapts her language.

---

## Architecture Overview

```
Patient WhatsApp
      │
      ▼
Meta Cloud API (Webhook)
      │
      ▼
Express.js Backend (Replit)
      │
      ├── Stage Detection (rule-based state machine)
      │     collecting_nome → collecting_cpf → collecting_email
      │     → asked_valor_ok → awaiting_payment → scheduled
      │
      ├── Claude AI (Anthropic claude-sonnet-4-5)
      │     Humanized responses with full context injection
      │
      ├── Admin Escalation
      │     Unknown questions → Fran's WhatsApp → answer → patient
      │
      ├── PostgreSQL (Neon serverless)
      │     All conversations persisted with JSONB messages
      │
      └── Google Sheets
            Real-time sync of all leads + transcriptions
```

### Tech Stack

| Component | Technology |
|-----------|-----------|
| Runtime | Node.js + TypeScript |
| Framework | Express.js |
| AI Model | Claude Sonnet 4.5 (Anthropic) |
| Database | PostgreSQL via Neon (serverless) |
| ORM | Drizzle ORM |
| WhatsApp | Meta Cloud API (direct, no Twilio) |
| Frontend | React + Vite + Tailwind + shadcn/ui |
| Spreadsheet | Google Sheets (via Google Drive connector) |
| Deployment | Replit |

---

## How VIZZ Escalates Doubts to the Admin

One of VIZZ's most powerful features is her ability to **know what she doesn't know**.

When a patient asks a question outside her training (e.g., "Can I have surgery with lupus?"), VIZZ:

1. Emits a `[VIZZ_DOUBT:question text]` marker in her response
2. The system detects this and sends a WhatsApp message to Fran (the clinic admin):
   > "Fran, preciso da sua orientação antes de responder ao lead Maria (+55 51 9xxxx-xxxx). Ela tem uma dúvida que não consigo responder com segurança: 1. Perguntou se pode fazer lipoaspiração com diagnóstico de lúpus. Responda diretamente aqui..."
3. VIZZ tells the patient she's checking and will return shortly
4. When Fran replies, the system links the answer back to the specific patient and schedules VIZZ's response

This creates a seamless experience where the patient never knows a human was consulted.

---

## The Vibe Coding Story

This entire system was built using **vibe coding** — a development approach where the developer describes what they want in natural language and an AI coding assistant (in this case, Replit Agent + Claude) writes the code.

**What I (the founder) contributed:**
- The clinical domain knowledge
- The product decisions ("she should never feel like a bot")
- The testing and feedback ("this response is too formal")
- The business logic ("if she doesn't know, ask Fran")

**What Claude contributed:**
- ~95% of the actual code written
- Database schema design
- API integrations (Meta, Google, Anthropic)
- Error handling and edge cases
- The entire state machine for patient qualification

**The development environment:**
- All coding done inside [Replit](https://replit.com) using the AI Agent feature
- Conversations with the AI agent lasted hours, iterating on behavior
- The consultant persona was refined through dozens of test conversations

**Total development time:** ~3 months of iterative vibe coding  
**Total cost:** ~$45/month (Replit Core + Claude Pro)  
**Lines of code:** ~7,000+ (TypeScript, all AI-generated)

---

## Key Technical Challenges Solved

### 1. Neon HTTP Adapter Bug
The Neon serverless PostgreSQL adapter had a silent bug where `INSERT ... RETURNING *` returned empty rows, and `null` timestamps were serialized as empty strings. Fixed by bypassing the ORM with raw SQL and doing a separate `SELECT` after every `INSERT`.

### 2. Conversation Persistence After Restart
WhatsApp conversations needed to survive server restarts. Solution: memory cache with PostgreSQL fallback — on every message, the conversation is persisted to DB. On startup, conversations are recovered from DB and from Google Sheets.

### 3. Name Extraction Without Asking Directly
Patients often introduce themselves mid-sentence ("Oi, sou a Maria, quero saber sobre lipo"). VIZZ extracts names using three layers:
- Direct extraction from common intro patterns ("me chamo", "sou a", "meu nome é")
- Mining VIZZ's own response ("Oi, Maria!") to find the name she used
- Fallback from the scheduling data if the stage machine already captured it

### 4. Multi-Admin Doubt Routing
When VIZZ escalates a doubt, multiple admins can respond. The system tracks who responded, detects conflicting answers (via Claude), and uses the most authoritative response.

---

## Academic Application

This project is part of a Health Economics research initiative studying **no-show behavior in elective surgery clinics** in Brazil.

Data collected (with LGPD compliance):
- Lead source (WhatsApp, Instagram, Google, referral, etc.)
- Time from first contact to consultation scheduled
- Payment method preference
- Drop-off stage in the qualification funnel
- Follow-up response rates

The goal is to build a **predictive model for no-shows** that clinics can use to optimize their scheduling.

---

## LGPD Compliance (Brazilian Data Protection Law)

- Consent collected before data processing
- Right to erasure (soft-delete + anonymization)
- Audit trail for all data access
- Data portability (export to Excel/CSV/PDF)
- No data shared with third parties

---

## About the Creator

Built by **Dr. Mateus Vizzotto's team** in Porto Alegre, Brazil, as a practical application of AI in healthcare operations.

This project demonstrates that with modern AI tools (Replit + Claude), a non-developer with domain expertise can build production-grade software that genuinely helps people — without writing a single line of code manually.

---

## Why This Matters for AI Research

VIZZ is a real-world case study of:

1. **AI in healthcare access** — Making specialist consultations more accessible by removing friction from the booking process
2. **Human-AI collaboration** — The admin escalation system is a model for AI knowing its own limits
3. **Vibe coding at scale** — A fully deployed production system built without traditional software development
4. **Behavioral AI** — Training a persona to be warm, consistent, and non-robotic in an emotionally sensitive domain (patients considering surgery)

---

*For questions or collaboration: contact via GitHub Issues*
