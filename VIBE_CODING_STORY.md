# The Vibe Coding Story Behind VIZZ

*How a clinic owner with no programming background built a production AI system in 3 months using Replit + Claude*

---

## The Problem

Dr. Mateus Vizzotto runs a plastic surgery clinic in Porto Alegre, Brazil. Like most specialist clinics, the biggest operational headache isn't the medicine — it's the patients who never show up.

The journey from "I saw your Instagram" to "I'm in the waiting room" involves:
- A WhatsApp message (often sent at 11pm)
- A follow-up (often not received)
- A qualification call
- A payment
- A scheduling confirmation
- A reminder
- An attendance check

At every step, leads drop off. Manually managing this for a busy surgeon is unsustainable.

---

## The Insight

The insight was simple: **the entire pre-consultation funnel is a conversation**. It follows predictable patterns, asks predictable questions, and requires predictable follow-ups.

If an AI could handle that conversation — warmly, knowledgeably, and persistently — the surgeon could focus on surgery.

---

## Why "Vibe Coding"?

Vibe coding is the practice of building software by describing what you want in natural language, letting an AI write the code, testing the result, and iterating.

I am not a software developer. I don't know TypeScript. I cannot read a database schema. But I deeply understand:
- How plastic surgery patients think and worry
- What questions they ask
- What makes them trust a clinic
- When they need a human and when they don't

That domain expertise is what I contributed. Claude (via Replit Agent) contributed everything else.

---

## How It Actually Worked

### Phase 1: "Make her talk"
First conversations with Replit Agent were about getting a basic WhatsApp bot running. The agent scaffolded the entire Express.js backend, Meta webhook integration, and basic Claude response generation in a single session.

I tested by sending messages to the number. The responses were technically correct but felt robotic.

**My instruction to the AI:** "She sounds like a chatbot. Make her sound like a consultant who genuinely cares."

The agent rewrote the system prompt. Better, but still not right.

### Phase 2: "She needs to know her limits"
Early version would hallucinate clinical information. When a patient asked about drug interactions, she invented an answer.

**My instruction:** "When she doesn't know something medical, she should NOT answer. She should tell the patient she's checking with the doctor, then actually message the clinic admin."

This required a complex multi-step system: marker detection in AI output, admin notification via WhatsApp, response tracking, patient reply scheduling.

The agent built all of it. I just described the behavior I wanted.

### Phase 3: "She keeps losing patient names"
A subtle bug: patients would give their name and VIZZ would acknowledge it but not save it. Next message, she'd ask again.

**My instruction:** "She keeps forgetting names. Fix this — once she knows someone's name, she should always use it."

The agent identified the issue (a condition that skipped name extraction when a CRM patient record existed) and fixed it.

### Phase 4: The persistence bug
After going live, I noticed the database had only 6 conversations despite 121 patients having talked to VIZZ.

**My instruction:** "Something is wrong — 121 people talked to her but only 6 are in the database."

The agent diagnosed: a silent bug in the Neon PostgreSQL HTTP adapter where `INSERT ... RETURNING *` returned empty arrays without throwing an error. It rewrote the entire persistence layer to bypass the ORM and use raw SQL.

---

## What I Learned

### About AI development:
- **Describing behavior is harder than writing code** — you have to be precise about edge cases
- **Domain knowledge is the competitive advantage** — the AI can write code, but it can't feel the patient's anxiety about choosing a surgeon
- **Iteration is everything** — the first version of anything was wrong; the tenth was good; the twentieth was production-ready

### About vibe coding:
- It's not "press a button and get software" — it's a skilled collaboration
- The human's job is to be a ruthless product manager: test everything, report bugs precisely, refuse mediocre results
- The AI's job is to translate intent into working code

### About healthcare AI:
- Patients are more accepting of AI assistants than expected — when they feel heard, they don't ask "are you a bot?"
- The humanization is in the *constraints*, not the features: no emojis, correct time-of-day greetings, never repeating questions, admitting uncertainty
- The escalation to a human (Fran) when VIZZ doesn't know is the most trusted feature — patients feel the system is careful, not just automated

---

## Numbers That Matter

- **3 months** from first line of AI-generated code to production deployment
- **$45/month** total infrastructure cost (Replit Core + Claude Pro)
- **~7,000 lines** of TypeScript — none written by hand
- **178 API endpoints** — scaffolded and maintained by AI
- **121+ patients** handled in production
- **0 developers** employed — just a clinic owner with domain knowledge and Replit

---

## Why I'm Applying to Anthropic Research

I believe VIZZ is evidence for something important:

**The next frontier of AI impact is not replacing experts — it's amplifying them.**

Dr. Mateus doesn't need AI to do surgery. He needs AI to handle everything around the surgery so he can do more of it, better.

The patients who talk to VIZZ aren't interacting with a generic chatbot. They're interacting with a system that carries the clinic's specific knowledge, personality, and values — because I was able to encode all of that through natural language, not code.

This is a new kind of software creation. And I want to understand it better, contribute to its development, and help define what it looks like when it's done responsibly.

---

*Built by the founder of the VZT Ecosystem, Porto Alegre, Brazil, 2025–2026.*
