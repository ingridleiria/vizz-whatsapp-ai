# VIZZ — WhatsApp AI for a Plastic Surgery Clinic in Brazil

> **"Dr. Vizzotto looked everywhere for an AI that could talk to patients like a human being. He couldn't find one. So we built her."**

VIZZ is a production WhatsApp AI assistant built for **Vizzotto Cirurgia Plástica** — Dr. Mateus Vizzotto's plastic surgery clinic in Rio Grande do Sul, Brazil. She handles the full patient journey — from first contact to appointment scheduling — entirely over WhatsApp, in Portuguese, with a warm and humanized tone that patients often mistake for a real person.

Built with **vibe coding** using [Replit](https://replit.com) + [Claude (Anthropic)](https://anthropic.com) by **Ingrid Leiria**, who started developing this idea with Dr. Vizzotto in November 2024 and began live patient testing in February 2025. This is not a paid project — Ingrid built VIZZ because she believed it was the right problem to solve.

---

## Why VIZZ Exists — The Real Story

In 2024, Dr. Mateus Vizzotto faced a problem that every busy specialist faces: **the patients he could help were getting lost before they ever reached him.**

People would see his work on Instagram. They'd send a WhatsApp message at 10pm. By morning, three other clinics had already responded. His number stayed unread.

And when responses did happen — through his team — they were inconsistent. Some patients received warmth and information. Others got a generic "we'll call you back." Many never got followed up at all.

Dr. Vizzotto started looking for an AI solution. He tested chatbots. He tried pre-built WhatsApp automation tools. He evaluated what was available in the Brazilian market for medical clinics.

**None of it felt human.**

Every solution felt like a form with a chat bubble on top. Patients would disengage the moment they sensed a script. The responses were transactional, not conversational. There was no warmth, no personality, no clinical knowledge — just rigid flows that failed the moment a patient said something unexpected.

He shared this frustration with Ingrid Leiria, who had been consulting on health economics and clinic operations. They began designing something different — not a chatbot, but a consultant. A person with a name.

They named her **Vizz**.

---

## Who Built VIZZ

**Dr. Mateus Vizzotto** — Plastic surgeon, clinic owner. Provided the clinical knowledge, the patient communication philosophy, and the vision. Asked the hardest questions: "Why does she feel like a robot?" and "What would a real consultant say here?"

**Fran** — Clinic administrative coordinator. The operational brain of the clinic. Became VIZZ's escalation partner — when VIZZ doesn't know something, she asks Fran. Fran's responses shaped VIZZ's knowledge base in real-time.

**Ingrid Leiria** — Developer, product designer, trainer. Wrote zero lines of code manually. Built the entire system through vibe coding with Replit Agent and Claude. Spends hours iterating on VIZZ's responses, catching when she sounds robotic, and pushing her toward something more human. **This is unpaid work** — Ingrid does this because she believes AI can genuinely improve healthcare access in Brazil.

---

## VIZZ Was Not Born Humanized — She Was Taught

This is the most important part of the story.

The first version of VIZZ was technically functional and emotionally cold.

She answered questions correctly. She collected CPF numbers efficiently. She followed the script. And she felt exactly like a script.

Patients would stop responding mid-conversation. Not because VIZZ gave wrong answers — but because the *tone* was wrong. The pacing was wrong. The way she asked for information felt like a form, not a conversation.

Ingrid spent months teaching her. Not by writing code — by describing behavior.

**"She shouldn't ask for CPF right away. She should get to know the patient first."**

**"When someone asks a medical question she's not sure about, she shouldn't just say 'I don't know.' She should tell them she's checking with the doctor and actually do it."**

**"She uses 'olá' too much. Real people in Porto Alegre don't say that."**

**"She repeated the patient's name three times in one message. It feels fake. Reduce it."**

**"She put an emoji. The clinic's consultants never use emojis. Remove all of them. Forever."**

Each iteration made VIZZ slightly more real. The conversations got longer. Patients stopped dropping off. Some started calling her by name — "Vizz, me fala mais sobre a cruroplastia..." — as if they were texting a person they already trusted.

VIZZ is not a product that was configured. She is a persona that was grown.

---

## What is VIZZ?

VIZZ is a WhatsApp-native AI consultant that:

- **Qualifies leads** — understands the patient's interest, the procedure, and their payment capacity
- **Collects patient data** — name, CPF (Brazilian tax ID), email, phone — naturally, in conversation, not as a form
- **Answers clinical questions** — mamoplastia, lipoaspiração, cruroplastia, rinoplastia, blefaroplastia, abdominoplastia and more — with accurate, non-alarmist information
- **Escalates intelligently** — when she genuinely doesn't know, she tells the patient she'll check with the doctor, messages Fran directly on WhatsApp, waits for the answer, and returns to the patient
- **Schedules consultations** — guides patients through the R$ 500 consultation fee, PIX payment, and pre-scheduling with the team
- **Follows up proactively** — re-engages leads who went silent, at 24h, 2 days, 7 days, 30 days, and 90 days — without ever feeling pushy

---

## How to Talk to VIZZ

VIZZ runs on the clinic's official WhatsApp Business number:

**+55 51 2391-7004** (Consultora Vizz)

Simply send any message about plastic surgery procedures — she responds in real-time, 24/7.

**Example openers:**
- "Oi, tenho interesse em fazer uma lipo"
- "Quero saber sobre mamoplastia de aumento"
- "Quanto custa uma consulta com o Dr. Mateus?"
- "Tenho uma dúvida sobre cruroplastia"

---

## How VIZZ Talks to Fran — The Admin Escalation System

This is one of VIZZ's most distinctive features, and it emerged from a real operational insight: **Fran already handles the hard questions. VIZZ just needed to know how to ask Fran.**

When a patient asks something VIZZ isn't certain about:

1. VIZZ tells the patient warmly: *"Que boa pergunta — vou confirmar com o Dr. Mateus e já te retorno, não quero te dar uma informação sem ter certeza."*
2. She sends a structured message to Fran on WhatsApp with the patient's name, phone, and the specific question
3. When Fran replies, the system links that answer to the specific conversation
4. VIZZ returns to the patient with the confirmed information, as if she had gone and asked herself

**Fran also uses VIZZ as an admin dashboard** — she can ask VIZZ directly on WhatsApp:
- "Me dá um resumo das conversas de hoje"
- "Quem está esperando retorno?"
- "Me fala sobre a Maria que mandou mensagem ontem"

And VIZZ answers from the full conversation database — all 125+ leads, their current stage, their last message, their pending doubts.

---

## Impact Numbers

| Metric | Value |
|--------|-------|
| Conversations tracked in database | **125** |
| Total leads from Google Sheets | **121** |
| Follow-up cadence steps | **5 (24h → 2d → 7d → 30d → 90d)** |
| Response time | **< 3 seconds** |
| Uptime | **24/7** |
| Languages supported | **Portuguese, English, Spanish** |
| Procedures in knowledge base | **12+** |
| Development cost | **~$45/month** (Replit Core + Claude Pro) |
| Salary paid to Ingrid | **R$ 0** |

---

## The Timeline

| Date | Milestone |
|------|-----------|
| **November 2024** | Ingrid and Dr. Vizzotto begin designing VIZZ together |
| **November–January** | Concept development — defining VIZZ's persona, knowledge base, escalation flow |
| **February 2025** | First live patient conversations — VIZZ tested with real incoming leads |
| **February–April** | Intensive iteration — teaching humanization, fixing tone, correcting behavior |
| **April 2025** | VIZZ Admin Mode built — Fran and Dr. Mateus can query VIZZ via WhatsApp |
| **June 2025** | Emergency stop system, 2h payment confirmation SLA, attendance check at 20h |
| **Today** | 125 conversations in database, running live 24/7 |

---

## The Humanization Principles (Learned, Not Programmed)

These rules were not written into code on day one. They emerged from watching real conversations fail and asking why.

**1. She uses your name — once, naturally**
Not three times. Not at the start of every message. Once, when it flows. Like a person would.

**2. She doesn't ask for CPF immediately**
She builds a conversation first. The data collection happens after trust is established.

**3. She admits uncertainty without abandoning the patient**
"Não tenho certeza — mas vou verificar" is more trusted than a confident wrong answer.

**4. She reads the time**
"Bom dia", "Boa tarde", "Boa noite" — based on actual Brasília timezone. No patient receives "Bom dia" at 9pm.

**5. Zero emojis**
The clinic's real consultants don't use emojis. VIZZ doesn't either. This was a deliberate, enforced rule.

**6. She doesn't repeat questions**
She tracks every piece of information the patient has already shared. She never asks for it again.

**7. She knows when to stop**
When a patient has clearly lost interest, she doesn't spam follow-ups. The cadence has a maximum of 5 attempts — then silence.

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
      │     Humanized responses with full patient context
      │
      ├── Admin Escalation
      │     Unknown questions → Fran's WhatsApp → answer → patient
      │
      ├── PostgreSQL (Neon serverless)
      │     All conversations persisted with JSONB messages
      │
      └── Google Sheets
            Real-time sync of all leads and transcriptions
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
| CRM | Custom-built pipeline inside the same system |
| Deployment | Replit |

---

## Academic Application

This project is part of a Health Economics research initiative studying **no-show behavior in elective surgery clinics** in Brazil.

Data collected (with full LGPD compliance):
- Lead source (WhatsApp, Instagram, Google, referral, etc.)
- Time from first contact to consultation scheduled
- Payment method preference
- Drop-off stage in the qualification funnel
- Follow-up response rates by cadence step

Goal: a predictive model for no-shows that clinics across Brazil can use.

---

## LGPD Compliance (Brazilian Data Protection Law)

- Explicit consent before data processing
- Right to erasure (soft-delete + anonymization)
- Full audit trail for all data access
- Data portability (export to Excel/CSV/PDF/Word)
- No patient data shared with third parties
- All data stored in Brazil-region infrastructure

---

## For Researchers and Collaborators

VIZZ is a live case study of:

- **AI in healthcare access** — removing friction from specialist consultations in a country with significant healthcare inequality
- **Human-AI collaboration** — the admin escalation system is a working model of AI knowing its own limits and routing to humans appropriately
- **Persona training without fine-tuning** — VIZZ's humanization was achieved entirely through prompt engineering and iterative behavioral feedback, with no model training
- **Vibe coding in production** — a fully deployed, real-patient-facing system built without a single line of manually written code

---

*Vizzotto Cirurgia Plástica — Rio Grande do Sul, Brazil*
*For questions: open a GitHub Issue or contact via the repository*
