# VIZZ — Full Technical Architecture

## System Map

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         VIZZ Ecosystem                                   │
│                                                                          │
│   Patient (WhatsApp)                                                     │
│        │                                                                 │
│        ▼                                                                 │
│   Meta Cloud API  ──HMAC-SHA256──►  POST /api/webhooks/whatsapp          │
│        │                                     │                           │
│        │              ┌──────────────────────┤                           │
│        │              ▼                      ▼                           │
│        │        isAdminPhone?           Patient Flow                     │
│        │            │                       │                            │
│        │            ▼                       ▼                            │
│        │      Admin Mode              Stage Machine                      │
│        │   (Fran / Dr. Mateus)    (rule-based + AI)                     │
│        │            │                       │                            │
│        │            └──────────┬────────────┘                            │
│        │                       ▼                                         │
│        │              Claude AI (Anthropic)                              │
│        │              claude-sonnet-4-5  ◄──── System Prompt             │
│        │              claude-haiku-4-5        (3,000+ tokens)            │
│        │                       │                                         │
│        │              ┌────────┴─────────┐                              │
│        │              ▼                  ▼                               │
│        │         PostgreSQL         Google Sheets                        │
│        │       (Neon serverless)   (Lead tracking)                       │
│        │              │                                                  │
│        │    ┌─────────┴──────────┐                                      │
│        │    │                    │                                       │
│        │  Audio               Doubts                                     │
│        │  (voice msg)        escalated                                   │
│        │    │                    │                                       │
│        │    ▼                    ▼                                       │
│   Groq Whisper /          Fran's WhatsApp                               │
│   Gemini Flash           (+55 51 99520-0513)                             │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## AI Models Used

VIZZ uses three different Anthropic Claude models depending on the task — chosen for the right balance of quality vs. latency vs. cost:

| Model | Used For | Max Tokens | Temperature |
|-------|----------|-----------|-------------|
| **claude-sonnet-4-5** | Main patient responses, admin mode, 2h conversation review, morning briefing, Fran doubt analysis | 1,000–1,500 | 0.8 |
| **claude-haiku-4-5** | Stage detection, name extraction, intent classification, quick admin summaries | 50–500 | 0–0.4 |
| **claude-opus-4-5** | WhatsApp group message analysis (higher complexity reasoning) | 800 | — |

### Why three models?

Every WhatsApp message triggers at minimum:
1. A stage classification call (haiku — fast, cheap, deterministic)
2. A response generation call (sonnet — quality, contextual)

Using Haiku for classification reduces latency and cost significantly. Sonnet handles everything that touches the patient directly. Opus is reserved for the most complex reasoning tasks.

---

## Audio Transcription Pipeline

When a patient sends a voice message:

```
Patient sends audio (OGG/Opus from WhatsApp)
        │
        ▼
Download binary from Meta CDN
        │
        ▼
Try Groq Whisper (primary)
  Model: whisper-large-v3-turbo
  API: https://api.groq.com/openai/v1/audio/transcriptions
  No fixed language → auto-detects PT / EN / ES
        │
        ├── Success → transcript text → VIZZ response
        │
        └── Fail or no GROQ_API_KEY
              │
              ▼
        Gemini 1.5 Flash (fallback)
          API: https://generativelanguage.googleapis.com
          Inline base64 audio + prompt
              │
              ▼
        Gemini 2.0 Flash (media processing)
          Used for images and documents → description text
```

---

## External APIs & Services

| Service | Purpose | API / SDK |
|---------|---------|-----------|
| **Anthropic Claude** | All AI responses | Anthropic Node.js SDK via Replit AI Integration |
| **Meta WhatsApp Cloud API** | Send/receive WhatsApp messages | REST — `graph.facebook.com/v18.0/` |
| **Groq** | Audio transcription (Whisper) | REST — `api.groq.com/openai/v1/audio/transcriptions` |
| **Google Gemini** | Audio fallback + image/doc description | `@google/generative-ai` SDK |
| **Neon** | PostgreSQL serverless database | `@neondatabase/serverless` driver |
| **Google Drive** | Patient file storage, Sheets lead tracking | Replit Google Drive Connector (OAuth) |
| **Resend** | Transactional email (lead notifications) | REST — `api.resend.com` |
| **Read.ai** | Meeting transcription webhook receiver | Webhook `POST /api/webhooks/readai` |
| **GitHub** | This repository | Replit GitHub Connector (OAuth) |

### Meta WhatsApp Cloud API — Key Endpoints Used

```
POST  graph.facebook.com/v18.0/{phone_number_id}/messages   ← send message
GET   graph.facebook.com/v18.0/{media_id}                    ← get media URL
GET   {media_url}                                             ← download binary

Inbound webhook: GET/POST /api/webhooks/whatsapp
  Verification: HMAC-SHA256 of raw body with WHATSAPP_APP_SECRET
  Message types handled: text, image, video, document, audio, voice, sticker
```

---

## Background Scanners (Always Running)

These are Node.js `setInterval` timers that run continuously on the server — no cron, no external job queue:

| Scanner | Interval | What It Does |
|---------|----------|-------------|
| **Conversation stage sync** | 9 min | Syncs all WhatsApp conversation stages to CRM patients table |
| **Pending follow-up check** | 2 min | Processes any follow-ups that are due right now (sub-cadence trigger) |
| **Lead follow-up scanner** | 15 min | Main cadence scanner: finds leads past their silence threshold and sends follow-up messages |
| **Proactive review (Claude)** | 2 h | Claude analyses conversations where the patient sent the last message 2–4h ago and decides contextually if VIZZ should re-engage |
| **Fran confirmation reminder** | 30 min | If Fran hasn't confirmed a payment in 2h, sends her a reminder |
| **Consultation attendance check** | 30 min | At 20h Brasília time, asks Fran if scheduled patients attended their consultation |
| **Google Sheets sync** | 5 min | Pushes new conversations and learnings to the tracking spreadsheet |

### Morning Briefing (7:30–10:00 Brasília)

On server startup, if the local time is between 07:30 and 10:00 Brasília:
- `runMorningPipelineReview()` runs immediately
- Generates a full pipeline summary and sends it to Fran's WhatsApp
- Includes: active leads, pending doubts, today's scheduled consultations, follow-up queue

---

## API Endpoint Reference

The system exposes **178 REST endpoints**. Key groups:

### Authentication
```
POST   /api/auth/login
POST   /api/auth/logout
GET    /api/auth/me
POST   /api/admin/reset-password
```

### WhatsApp / VIZZ
```
GET    /api/webhooks/whatsapp              ← Meta webhook verification
POST   /api/webhooks/whatsapp              ← inbound messages (main entry point)
GET    /api/whatsapp/conversations         ← list all conversations
GET    /api/whatsapp/real-conversations    ← real patient convs from DB (read-only view)
GET    /api/whatsapp/conversation/:phone   ← single conversation by phone
DELETE /api/whatsapp/conversation/:phone
POST   /api/whatsapp/send                 ← manual message send
POST   /api/whatsapp/scan-leads           ← manually trigger follow-up scanner
GET    /api/admin/vizz-history/:phone     ← full conversation transcript
GET    /api/admin/vizz-training           ← VIZZ learnings list
GET    /api/admin/whatsapp-token-status   ← token validation (checked on startup)
GET    /api/admin/briefing-preview        ← preview morning briefing without sending
```

### Patients (CRM / Patient 360)
```
GET    /api/patients                       ← list with search (name, CPF, phone)
GET    /api/patients/:id
POST   /api/patients
PUT    /api/patients/:id
DELETE /api/patients/:id
GET    /api/patients/export/excel
GET    /api/patients/export/csv
GET    /api/patients/export/pdf
GET    /api/patients/export/word
GET    /api/patients/:id/interactions
GET    /api/patients/:id/files
GET    /api/patients/:id/tasks
GET    /api/patients/:id/transcripts
GET    /api/patients/:id/drive-files
GET    /api/patients/:id/communications
GET    /api/patients/:id/consents
GET    /api/patients/:id/access-history
GET    /api/patients/:id/export           ← LGPD data portability
```

### Appointments
```
GET    /api/appointments
POST   /api/appointments
GET    /api/appointments/:id
PUT    /api/appointments/:id
DELETE /api/appointments/:id
GET    /api/analytics/no-show             ← no-show analytics for research
GET    /api/patients/:id/appointments
```

### AI Chat (Internal)
```
POST   /api/ai/chat                        ← internal AI chat with patient context
GET    /api/ai/conversations
GET    /api/ai/conversations/:id
DELETE /api/ai/conversations/:id
```

### Google Drive
```
GET    /api/drive/folders
GET    /api/drive/files
GET    /api/drive/search
GET    /api/drive/recent
GET    /api/drive/status
GET    /api/drive/root-url
GET    /api/drive/administrativo
GET    /api/drive/marketing
GET    /api/drive/processos
```

### Finance & Research
```
GET    /api/finance/transactions
GET    /api/finance/accounts
GET    /api/finance/dre-summary
GET    /api/finance/dre-trends
GET    /api/finance/snapshots
GET    /api/research/export/demographics
GET    /api/research/export/procedures
GET    /api/research/export/financial
GET    /api/research/export/outcomes
GET    /api/research/export/satisfaction
```

### Admin
```
GET    /api/admin/users
POST   /api/admin/users
DELETE /api/admin/users/:id
GET    /api/admin/learnings
DELETE /api/admin/learnings/:id
GET    /api/admin/transcription-status
GET    /api/stats
```

### Webhooks
```
POST   /api/webhooks/whatsapp             ← Meta WhatsApp
POST   /api/webhooks/readai               ← Read.ai meeting transcriptions
```

---

## Database Schema — All 20 Tables

PostgreSQL via Neon serverless. ORM: Drizzle ORM with `drizzle-zod` for validation.

```sql
-- Identity & Auth
sessions                    -- Express session store
tenants                     -- Multi-tenant isolation (one per clinic)
users                       -- Clinic staff accounts (super_admin / admin / user)
password_reset_tokens
google_whitelist            -- Allowed Google OAuth emails

-- CRM / Patient Management
patients                    -- Full patient records (CPF, procedures, status)
patient_interactions        -- Timeline of all patient touchpoints
patient_files               -- Documents linked to patients
patient_tasks               -- Task assignments per patient
patient_consents            -- LGPD consent records per patient per purpose

-- AI & Conversations
ai_conversations            -- Internal AI chat sessions
ai_chat_settings            -- Per-tenant AI configuration
whatsapp_conversations      -- All VIZZ conversations (JSONB messages, stage, scheduling data)
voice_transcripts           -- Voice recorder transcriptions

-- Scheduling & Finance
appointments                -- Consultations (date, time, confirmed, attended)
consultation_payments       -- Payment tracking per consultation

-- Tasks
tasks                       -- Internal task management
task_categories / task_tags / task_projects / task_tag_links / task_assignees / task_attachments / task_comments

-- System
plugins                     -- Feature flags / plugin registry
tenant_plugins              -- Per-tenant plugin activation
tenant_integrations         -- CRM and calendar integration credentials
audit_logs                  -- LGPD audit trail (who accessed what, when)
```

### whatsapp_conversations — Key Column Detail

```sql
CREATE TABLE whatsapp_conversations (
  id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  phone             TEXT NOT NULL,
  tenant_id         VARCHAR,
  patient_name      TEXT,
  scheduling_stage  TEXT,        -- none / initial_contact / collecting_nome /
                                 -- collecting_cpf / collecting_email /
                                 -- awaiting_payment / scheduled / consulta_realizada / lost
  scheduling_data   JSONB,       -- cpf, email, procedure, payment_method,
                                 -- franAlertedAt, franConfirmed, attendanceCheckSent,
                                 -- pendingDoubts[], followUpCount
  messages          JSONB,       -- array of {role, content, timestamp}, max 30
  follow_up_at      TIMESTAMP,   -- next scheduled follow-up
  follow_up_count   INTEGER,     -- 0–5 (stops at 5)
  last_message_at   TIMESTAMP,
  last_client_ip    TEXT,        -- Meta server IP (audit trail)
  created_at        TIMESTAMP
);
```

---

## Qualification State Machine — Full Stage Map

```
[none]
    │ Patient sends first message
    ▼
[initial_contact]
    │ VIZZ greets, asks: particular or convenio?
    ▼
[asked_particular_or_convenio]
    ├── "convenio" → [asked_convenio_name] → handoff to Fran → [handoff]
    └── "particular" →
            │ VIZZ explains R$500 consultation fee
            ▼
        [asked_valor_ok]
            │ Patient agrees
            ▼
        [collecting_nome]
            │ Name extracted
            ▼
        [collecting_cpf]
            ├── CPF valid → next
            └── foreign patient → accept passport format
            ▼
        [collecting_email]
            ▼
        [collecting_telefone]
            ▼
        [awaiting_payment]
            │ Patient sends PIX receipt
            ├── Payment detected →
            │       [payment_confirmed]
            │           │ Fran alerted (2h SLA)
            │           ▼
            │       [scheduled]
            │           │ Attendance check at 20h on consultation day
            │           ▼
            │       [consulta_realizada]  ← stops all follow-ups
            │
            └── Patient goes silent → follow-up cadence runs
                    5 attempts max → [lost]
```

Stage transitions detected by:
1. **Regex patterns** on patient message text (fast, deterministic)
2. **Claude Haiku** for ambiguous inputs ("não sei", "talvez", etc.)
3. **Keyword detection** for payment confirmation (PIX receipt patterns)

---

## System Prompt Architecture

Every patient message generates a system prompt with these sections injected dynamically:

```
1. PERSONA DEFINITION
   — VIZZ's name, role, clinic name, zero-emoji rule, regional tone (RS/Brazil)

2. CLINICAL KNOWLEDGE BASE
   — 12+ procedures with descriptions, prices, recovery times, contraindications
   — Sourced from: mamoplastia, lipoaspiração, lipoaspiração HD, cruroplastia,
     rinoplastia, blefaroplastia, abdominoplastia, lifting facial, otoplastia,
     ninfoplastia, ginecomastia, lipoenxertia

3. CURRENT PATIENT CONTEXT
   — Name (if known), procedure interest, payment method
   — CPF / email / phone collection status
   — Prior CRM record if patient exists

4. STAGE-SPECIFIC INSTRUCTIONS
   — What to do and NOT do in the current stage
   — Specific phrasing constraints per stage

5. CONVERSATION HISTORY
   — Last N messages (up to 30) formatted as role/content pairs

6. BEHAVIORAL CONSTRAINTS
   — Do not repeat questions already answered
   — Emit [VIZZ_DOUBT:text] when uncertain
   — Emit [VIZZ_LOST] if patient is clearly disengaging
   — Greet with correct time-of-day (Brasília timezone)

7. LEARNED BEHAVIORS
   — Accumulated feedback from real conversations stored in DB
   — Synced to Google Sheets (learnings tab)
```

Total prompt size: **~3,000–4,500 tokens** per patient message.

---

## Multi-Tenancy

The system supports multiple clinics (tenants) from a single deployment:

- All DB queries scoped by `tenant_id`
- WhatsApp phone numbers mapped to tenant UUIDs via `META_WHATSAPP_TENANT_MAP` env var
- Each tenant has isolated: patients, conversations, staff accounts, settings
- Currently deployed for one tenant: Vizzotto Cirurgia Plástica

---

## Security

| Layer | Implementation |
|-------|---------------|
| Webhook integrity | HMAC-SHA256 signature on all inbound Meta payloads |
| Session auth | Express sessions with PostgreSQL session store |
| Role-based access | `super_admin` / `admin` / `user` — enforced per endpoint |
| Admin phone whitelist | Hardcoded set of whitelisted numbers for VIZZ admin mode |
| LGPD compliance | Consent records, soft-delete, anonymization, audit log, data export |
| Token validation | WhatsApp access token checked against Meta API on every server startup |

---

## Frontend — Internal Dashboard

The system includes a full web dashboard (not patient-facing):

| Tech | Details |
|------|---------|
| Framework | React 18 + TypeScript |
| Build | Vite |
| Routing | Wouter |
| Server state | TanStack Query (React Query) |
| UI | Tailwind CSS + shadcn/ui + Radix UI primitives |
| Auth | Context-based with protected routes |

Key dashboard pages:
- **CRM Pipeline** — Kanban board with lead stages, Excel export
- **Patient 360** — Full patient record with all interactions, files, tasks
- **VIZZ History** — Three-panel conversation viewer (list / transcript / lead summary card)
- **VIZZ Real Conversations** — Read-only view of active patient conversations from DB
- **Appointments** — Scheduling with no-show analytics
- **AI Chat** — Internal AI assistant with full patient context
- **Voice Recorder** — Audio transcription and analysis
- **Google Drive** — File browser and patient folder management
- **Finance** — DRE, transactions, payment tracking
- **Admin** — User management, VIZZ training data, token status

---

## Deployment

| Aspect | Details |
|--------|---------|
| Host | Replit (always-on) |
| Process | Single Node.js process (Express + background timers) |
| Port | 5000 (internal), exposed via Replit proxy |
| Database | Neon serverless PostgreSQL (connection pooling built-in) |
| Restart | Replit workflow — `fuser -k 5000/tcp && npm run dev` |
| Cost | ~$45/month (Replit Core $25 + Claude Pro $20) |

---

## Known Technical Challenges Solved

### 1. Neon HTTP Adapter Silent Bug
`INSERT ... RETURNING *` returned empty arrays without throwing. `null` timestamps serialized as `""`. Fixed by bypassing Drizzle ORM for all `whatsapp_conversations` writes — raw SQL with explicit `TIMESTAMP` casts, followed by a separate `SELECT` to retrieve the inserted row.

### 2. Parallel GitHub Push Race Condition
The GitHub Contents API rejects concurrent commits to the same branch. Fixed by sequential file uploads (one `PUT` at a time) instead of `Promise.all()`.

### 3. Name Extraction Bypass
When a CRM patient record already existed, name extraction was skipped — causing VIZZ to ask for names already known. Fixed by making extraction unconditional and updating the CRM record when a new name is found in the conversation.

### 4. Audio Transcription Language Lock
Early Groq Whisper calls had `language: "pt"` hardcoded, causing poor transcription for English and Spanish speakers. Fixed by removing the `language` parameter — Whisper auto-detects PT/EN/ES accurately.

### 5. Conversation Recovery After Restart
In-memory conversation cache lost on server restart. Fixed by: persist every message to PostgreSQL synchronously, load all active conversations from DB on startup, merge with Google Sheets data as secondary source.
