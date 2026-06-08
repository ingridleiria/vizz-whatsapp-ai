# VIZZ — Technical Architecture

## System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    VIZZ Ecosystem                           │
│                                                             │
│  WhatsApp Patient ──► Meta Cloud API ──► Express.js         │
│                                              │               │
│                              ┌───────────────┼──────────┐   │
│                              │           Stage Machine   │   │
│                              │      (Rule-based + AI)    │   │
│                              └───────────────┬──────────┘   │
│                                              │               │
│                    ┌─────────────────────────┤               │
│                    │                         │               │
│              Claude AI                  PostgreSQL           │
│         (Anthropic API)              (Neon Serverless)       │
│                    │                         │               │
│                    └─────────────┬───────────┘               │
│                                  │                           │
│              ┌───────────────────┼──────────────┐            │
│              │                   │              │            │
│        Google Sheets        Admin WA         CRM DB          │
│       (Lead tracking)    (Escalation)     (Patients)         │
└─────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. WhatsApp Webhook (`server/routes.ts`)

Receives all inbound messages from Meta Cloud API. Handles:
- Text messages
- Audio messages (transcribed via Gemini)
- Images/documents (notifies admin)
- Stickers (acknowledged gracefully)

Routes messages to:
- Patient flow (standard leads)
- Admin flow (whitelisted clinic team numbers)

### 2. Stage Machine (`server/whatsapp-ai-service.ts`)

A deterministic state machine that tracks where each patient is in the qualification funnel:

```
none
  └── initial_contact
        └── asked_particular_or_convenio
              ├── asked_convenio_name (→ handoff to Fran)
              └── asked_valor_ok
                    └── collecting_nome
                          └── collecting_cpf
                                └── collecting_email
                                      └── collecting_telefone
                                            └── awaiting_payment
                                                  ├── payment_confirmed
                                                  │     └── scheduled
                                                  │           └── consulta_realizada
                                                  └── lost
```

Each stage transition is detected via:
1. Regex patterns on the patient message
2. Claude AI analysis for ambiguous cases
3. Keyword detection for payment confirmation

### 3. AI Response Generation

Every patient message goes through:

```typescript
buildSystemPrompt(patient, conversation) {
  // Injects: knowledge base, patient history, current stage context,
  //          name-missing alert, scheduling context, time of day,
  //          previous opportunities, learned behaviors
}

generateResponse(systemPrompt, messages) {
  // claude-sonnet-4-5 with temperature 0.7
  // Response parsed for: [VIZZ_DOUBT:], [VIZZ_LOST], [VIZZ_FOLLOWUP]
}
```

### 4. Doubt Escalation System

```
Patient asks unknown question
    │
    ▼
Claude emits [VIZZ_DOUBT:question text]
    │
    ▼
System detects marker, strips it from patient-facing response
    │
    ▼
extractUnansweredQuestionsFromConversation()
  → Finds ALL pending questions in conversation history
    │
    ▼
sendTextMessage(FRAN_PHONE, compiledDoubtMessage)
    │
    ▼
setAdminPendingDoubt(adminKey, leadPhone, doubt)
  → Stored in admin conversation's schedulingData.pendingDoubts
    │
    ▼
Patient receives: "Vou confirmar com o Dr. Mateus e já te retorno..."
    │
    ▼
Fran replies with #XXXXXX code or alone (if single pending doubt)
    │
    ▼
handleFranDoubtResponse() resolves and schedules patient reply
```

### 5. Proactive Follow-Up Scanner

Runs every 15 minutes, checks for leads that:
- Have NOT reached scheduling/payment stages
- Have been silent for the cadence thresholds
- Have not exceeded 5 follow-up attempts

```
Cadence:
  1st: 24h after last contact
  2nd: +24h after 1st (Day 2)
  3rd: +7 days after 2nd
  4th: +30 days after 3rd
  5th: +90 days after 4th (final)
```

### 6. Conversation Persistence

Fixed a critical Neon HTTP adapter bug where:
- `INSERT ... RETURNING *` silently returned empty arrays
- `null` timestamps were serialized as `""` causing PostgreSQL errors

Solution: bypass ORM, use raw SQL with explicit casts:
```sql
INSERT INTO whatsapp_conversations (phone, tenant_id, patient_name, ...)
VALUES ($1, $2, $3, ...)

-- Then separate SELECT to fetch the inserted row
SELECT * FROM whatsapp_conversations WHERE phone = $1 AND tenant_id = $2
```

## Database Schema (Key Tables)

```sql
whatsapp_conversations
  id          UUID PRIMARY KEY
  phone       TEXT NOT NULL          -- patient's WhatsApp number
  tenant_id   VARCHAR                -- clinic identifier
  patient_name TEXT                  -- extracted from conversation
  scheduling_stage TEXT              -- current funnel stage
  scheduling_data  JSONB             -- CPF, email, procedure, payment, etc.
  messages    JSONB                  -- up to 30 most recent messages
  follow_up_at TIMESTAMP             -- next scheduled follow-up
  follow_up_count INTEGER            -- 0-5, stops cadence at 5
  last_message_at TIMESTAMP
  created_at  TIMESTAMP

patients                             -- CRM records (post-qualification)
  id          UUID PRIMARY KEY
  nome        TEXT
  cpf         TEXT
  email       TEXT
  telefone    TEXT
  procedimento TEXT
  status      TEXT                   -- CRM pipeline stage

appointments                         -- Scheduled consultations
  id          UUID PRIMARY KEY
  patient_id  UUID REFERENCES patients
  data_consulta DATE
  horario     TIME
  confirmado  BOOLEAN
  compareceu  BOOLEAN               -- set at 20h on day of consultation
```

## Admin Mode

When Fran, Dr. Mateus, or Ingrid message VIZZ:

```
isAdminPhone(msg.from) → true
    │
    ▼
Skip all patient logic (no CRM entry, no Sheets sync, no email)
    │
    ▼
Load all active conversations from DB
    │
    ▼
buildAdminSystemPrompt(adminName, conversations)
  → VIZZ becomes a management dashboard over WhatsApp
    │
    ▼
generateAdminWhatsAppResponse()
  → Responds with summaries, urgent cases, specific patient details
    │
    ▼
Store in admin_PHONE conversation key (separate from patient keys)
```

## Environment Variables Required

```bash
# WhatsApp (Meta Cloud API)
WHATSAPP_ACCESS_TOKEN=
WHATSAPP_PHONE_NUMBER_ID=
WHATSAPP_APP_SECRET=
WHATSAPP_VERIFY_TOKEN=
META_WHATSAPP_TENANT_MAP=  # phoneNumberId:tenantUuid

# AI
AI_INTEGRATIONS_ANTHROPIC_API_KEY=  # via Replit integration
GEMINI_API_KEY=                      # for audio transcription
GROQ_API_KEY=                        # Whisper fallback

# Database
DATABASE_URL=  # Neon PostgreSQL connection string

# Admin phones (whitelisted)
VIZZ_ADMIN_PHONES=  # JSON: {"5551995200513":"Fran",...}
FRAN_PHONE=5551995200513
ADMIN_WHATSAPP_NUMBER=5551995200513

# Google
# Connected via Replit Google Drive connector (OAuth)

# Email (Resend)
RESEND_API_KEY=
```

## Security & Compliance

- **LGPD** (Brazilian GDPR equivalent): consent management, right to erasure, audit logs
- **Webhook signature validation**: HMAC-SHA256 on all Meta webhook payloads
- **Multi-tenancy**: tenant isolation at DB level, all queries scoped by tenant_id
- **Role-based access**: super_admin / admin / user roles
- **Session-based auth**: Express sessions, no JWT tokens in URLs
