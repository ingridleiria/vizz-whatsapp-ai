# The Vibe Coding Story Behind VIZZ

*How a health economist and a plastic surgeon built a production AI system — without a developer, without a salary, and without writing a single line of code*

---

## The Search That Started Everything

In 2024, Dr. Mateus Vizzotto was losing patients he never got to meet.

Not in surgery. Before that. In the gap between "a person sends a WhatsApp message at 10pm" and "a person shows up for a consultation."

That gap — filled with missed follow-ups, slow responses, inconsistent information, and no-shows — was costing the clinic patients, revenue, and the ability to help people who genuinely needed surgery.

Dr. Vizzotto started looking for an AI solution. This was not a casual search. He tested everything available for medical clinics in Brazil:

- Pre-built WhatsApp chatbots
- CRM automation tools with AI integrations
- Generic AI assistants configured for healthcare
- Brazilian healthtech platforms

**He couldn't find what he was looking for.**

Every solution had the same problem: it felt like a machine. Patients would engage for one or two messages, sense the script, and stop responding. Or worse — they'd keep interacting but feel like they were filling out a form, not talking to someone at the clinic.

The word he kept using was: **"desumanizado."** Dehumanized.

Plastic surgery patients are not looking for a product. They are considering changing something about their bodies — often something they've been self-conscious about for years. The moment they feel like they're talking to software, the trust collapses.

---

## November 2024 — The Idea

Dr. Vizzotto shared this frustration with Ingrid Leiria, a health economist who had been working on clinic operations and research.

Their conversation wasn't technical. It was about what the right experience should *feel like* for a patient who messaged the clinic for the first time.

- She should feel heard
- She should get real information about the procedures, not just "book a consultation"
- She should be guided, not interrogated
- If she asked something the AI didn't know, the AI should say so — and actually go find out
- She should never feel like she was talking to a bot

By the end of that conversation, they had the concept: not a chatbot, but a consultant. A person with a name, a personality, a knowledge base built from the clinic's real experience.

They called her **Vizz** — a variation on Vizzotto.

---

## What Ingrid Contributed vs. What the Clinic Contributed

This is important to understand, because it's a genuine collaboration — not a developer building something for a client.

**Ingrid contributed:**
- All technical development — via vibe coding with Replit Agent and Claude
- Product decisions — what features to build, in what order
- Testing — sending messages to VIZZ herself, identifying what felt wrong
- Iteration — translating "this response feels robotic" into specific instructions that changed the behavior
- Architecture — designing how all the pieces connect (database, WhatsApp API, escalation system, CRM)
- This is unpaid work. Ingrid does it because she believes in the project.

**The clinic contributed:**
- Clinical knowledge — what procedures exist, what patients typically ask, what the real answers are
- Process design — how consultations are scheduled, how payments work, what Fran's role is
- Feedback — "this is not how we talk to patients," "this phrasing is too formal," "a patient from the interior of RS would say it this way"
- Patient interaction history — the real conversations that trained VIZZ's intuitions

**Neither side could have built this alone.**

Ingrid could not have known the clinical nuances. Dr. Vizzotto and Fran could not have built the software. The collaboration is the product.

---

## February 2025 — First Real Patients

In February 2025, VIZZ went live with real incoming leads.

The first conversations were exciting and humbling. VIZZ worked — she responded instantly, collected information, answered basic questions. But watching real patients interact with her revealed problems no amount of planning could have anticipated.

**Problem 1: She asked for CPF too early.**
A patient would say "Oi, tenho interesse em cirurgia" and within two messages VIZZ was asking for their tax ID number. Patients would stop responding. They hadn't even decided if they trusted the clinic yet.

*Fix: restructure the qualification flow so CPF only comes after the patient has expressed clear intent and received real information.*

**Problem 2: She was too formal.**
VIZZ used "olá" and full formal constructions. In Rio Grande do Sul, people don't talk like that. It felt like a bank.

*Fix: rewrite the persona instructions with explicit regional tone guidance. "Oi" not "Olá." Contractions. Shorter sentences.*

**Problem 3: She repeated information patients had already given.**
A patient mentioned her name in the third message. Six messages later, VIZZ asked: "Como posso te chamar?"

*Fix: improve name extraction logic to catch all intro patterns ("sou a Maria", "me chamo Maria", "meu nome é Maria") and mine VIZZ's own previous responses.*

**Problem 4: She hallucinated when she didn't know.**
A patient asked about a contraindication with a specific medication. VIZZ answered confidently — with information that wasn't accurate.

*Fix: build the escalation system. If VIZZ is uncertain, she doesn't answer. She tells the patient she's verifying with the doctor, she messages Fran, and she waits.*

---

## How VIZZ Was Humanized — The Real Iterations

The humanization of VIZZ was not a feature that was added. It was a practice that was sustained over months.

Every week, Ingrid would read through real conversations. She would find the moments where VIZZ felt robotic, formal, or misaligned with how the clinic actually communicates.

Then she would translate those observations into instructions — sometimes very specific ones:

> "She said 'Entendido!' at the start of a response. Real people don't say that. It sounds like a customer service script. Remove that pattern entirely."

> "She listed three questions in numbered format. It felt like a form. She should ask one thing at a time, conversationally."

> "She said 'Fico à disposição para qualquer dúvida.' That's a closing line from a formal email. Remove it from WhatsApp responses."

> "She correctly identified the patient's procedure but then described it using clinical terminology. The patient asked about 'a cirurgia da barriga.' VIZZ should use the patient's language, not the textbook term."

> "This patient shared that she had been wanting this surgery for five years and was finally ready. VIZZ acknowledged it with one sentence and moved on to collecting data. She should have paused there. Acknowledged what that means. Then moved on."

Each of these observations went into the system prompt. Over months, the accumulated set of constraints and behaviors became VIZZ's personality.

---

## The Escalation to Fran — A System Born From Reality

The most important design decision in VIZZ was also the simplest: **let her ask for help.**

In the early versions, VIZZ would try to answer everything. This led to confident wrong answers — the worst possible outcome in a medical context.

The fix came from watching how Fran actually works. When a patient asks something Fran isn't sure about, she walks down the hall and asks Dr. Mateus. Or she messages him. She doesn't pretend to know. She goes and finds out.

VIZZ needed the same capability.

The system that was built:

1. When VIZZ is uncertain about something medical, her response to the patient includes a hidden marker: `[VIZZ_DOUBT: the specific question she's unsure about]`
2. The backend detects this marker, strips it from the patient-facing message
3. It sends a structured message to Fran's WhatsApp number with the patient's name, phone, and the question
4. Fran answers in plain language
5. The system links Fran's answer to the specific conversation and schedules VIZZ's follow-up

From the patient's perspective: VIZZ said she'd check with the doctor. A few minutes later, she came back with the answer. It feels completely natural.

From Fran's perspective: she gets a clear, structured message with the context she needs to answer quickly. It takes her 30 seconds.

---

## The Admin Mode — VIZZ Becomes Fran's Assistant

As VIZZ accumulated conversations, a new need emerged: Fran needed visibility into what was happening.

Instead of building a dashboard (which would require Fran to log into a web interface she didn't want to use), the team built **VIZZ Admin Mode** — a special mode that activates when Fran, Dr. Mateus, or Ingrid message VIZZ from their own WhatsApp numbers.

In admin mode, VIZZ becomes a management assistant. Fran can ask:

- "Me dá um resumo das conversas de hoje"
- "Quem está esperando meu retorno?"
- "Me fala sobre a paciente que perguntou sobre mamoplastia ontem"
- "Vizz, pause os atendimentos hoje à tarde"

And VIZZ answers from the live database — all conversations, their current stages, pending doubts, scheduled appointments.

This was not planned from the start. It emerged from a practical need: Fran wanted to know what was happening without learning a new tool. She already knew WhatsApp.

---

## What Vibe Coding Actually Means in Practice

Vibe coding is often described as "tell the AI what to build." That's technically accurate but misses what makes it hard.

**The hard part of vibe coding is not the code. It's the specification.**

To tell an AI to build something well, you have to understand what "well" means with precision. You have to catch when the AI misunderstood your intent. You have to hold a clear mental model of the system even when you can't read the code.

Ingrid describes her role as being a "ruthless product manager for the AI." The AI (Replit Agent + Claude) writes everything. Ingrid approves nothing that isn't right.

**Examples of interactions that shaped VIZZ:**

*"The database is not saving conversations correctly — 121 patients talked to VIZZ but only 6 are in the database. Something is wrong."*
→ The agent diagnosed a silent bug in the Neon PostgreSQL HTTP adapter (`INSERT ... RETURNING *` returning empty arrays), rewrote the persistence layer with raw SQL.

*"She's losing the patient's name when the CRM already has a record for them. Fix it."*
→ The agent found a condition that skipped name extraction when a CRM record existed. One-line logic fix. But Ingrid caught it by watching real conversations.

*"VIZZ needs to know when she's talking to Fran vs. a patient, and behave completely differently."*
→ The agent built the full admin mode system — phone number whitelist, separate conversation history, admin-specific system prompt, summary generation from DB.

Each of these started not with technical specification but with a behavioral observation. The translation from "this feels wrong" to "this is the code that fixes it" was the AI's job. Noticing that something felt wrong was Ingrid's.

---

## The Numbers

| Metric | Value |
|--------|-------|
| Project start | November 2024 |
| First live patient | February 2025 |
| Conversations in DB | 125 |
| Development cost | ~$45/month (Replit Core + Claude Pro) |
| Lines of code | ~7,000+ TypeScript |
| Lines written by hand | 0 |
| Developers employed | 0 |
| Salary paid | R$ 0 |
| Hours of iteration | Hundreds |

---

## Why This Matters

VIZZ is not a demo. She is not a prototype. She is not a pilot program. She is a production system that has handled over 125 real patient conversations, in Portuguese, about real medical procedures, with real money at stake.

She was built by someone with domain expertise and no programming background, using AI tools that are available to anyone today.

This is the shift that is happening: **expertise is becoming executable.**

The plastic surgeon who understands what patients need, the clinic coordinator who knows how the process works, the health economist who understands the data — they can now build the software that reflects their knowledge. Without intermediaries. Without translation loss. Without waiting for a developer to understand what they need.

VIZZ is early evidence of what that world looks like.

---

*Ingrid Leiria — Health Economist, vibe coder, VIZZ's creator*
*Dr. Mateus Vizzotto — Plastic surgeon, Vizzotto Cirurgia Plástica, Rio Grande do Sul, Brazil*
*Fran — Clinic coordinator, VIZZ's operational partner*
