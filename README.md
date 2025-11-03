# Unified Inbox

Multi-channel inbox (SMS, WhatsApp, Email) built with Next.js 14, Prisma, and provider SDKs.

## Stack

- Next.js 14 (App Router)
- Prisma + PostgreSQL
- Twilio (SMS, WhatsApp)
- Resend (Email)
- Better Auth (session gating)

## Environment

Create a .env based on .env.example:

- DATABASE_URL=postgres://...
- TWILIO_ACCOUNT_SID=...
- TWILIO_AUTH_TOKEN=...
- TWILIO_PHONE_NUMBER=+1xxxxxxxxxx
- TWILIO_WHATSAPP_NUMBER=whatsapp:+14155238886
- RESEND_API_KEY=...
- RESEND_FROM=onboarding@resend.dev
- NEXT_PUBLIC_APP_URL=http://localhost:3000
- NODE_ENV=development

## Development

- Install: `npm install`
- DB: `npm run db:push && npm run db:seed`
- Dev: `npm run dev` (http://localhost:3000)

## Integration comparison

| Channel  | Provider | Typical latency (p50/p95) | Cost per message (approx) | Reliability/SLA | Notes |
|----------|----------|----------------------------|----------------------------|-----------------|-------|
| SMS      | Twilio   | ~2s / ~5–10s               | Country dependent          | High            | Requires E.164 format. Trial only to verified numbers. Geo permissions must be enabled. |
| WhatsApp | Twilio   | ~1–3s / ~5–8s              | Per session/template       | High            | Must use approved WhatsApp sender. To field must be `whatsapp:+<E.164>`. |
| Email    | Resend   | ~1–3s / ~5–8s              | Per 1k emails (tiered)     | High            | For testing use `onboarding@resend.dev`; for prod use verified domain sender. |

Notes:
- Latency and cost vary by country/route/plan. Measure in your environment for precise numbers.
- Reliability reflects provider maturity and redundancy; still handle errors and retries.

## Key decisions

- Single-provider per channel:
  - SMS/WhatsApp: Twilio (direct API via `twilio` SDK).
  - Email: Resend (`resend` SDK).
- Minimal coupling:
  - Thin provider wrappers: [src/lib/integrations/twilio.ts](cci:7://file:///home/unified/Downloads/Unified_proj/unified-inbox/src/lib/integrations/twilio.ts:0:0-0:0), [src/lib/integrations/resend.ts](cci:7://file:///home/unified/Downloads/Unified_proj/unified-inbox/src/lib/integrations/resend.ts:0:0-0:0).
  - API routes call wrappers and update message status based on provider result.
- Error visibility:
  - Twilio errors surfaced with code/message/status to diagnose trial/format/geo issues.
- Data model:
  - Prisma models for Contact, Message, ScheduledMessage, Attachment, Note.
  - Message `status`: queued/sent/delivered/read/failed; we mark sent/failed at provider-call time.
- Security:
  - No secrets committed. [.env](cci:7://file:///home/unified/Downloads/Unified_proj/unified-inbox/.env:0:0-0:0) ignored; [.env.example](cci:7://file:///home/unified/Downloads/Unified_proj/unified-inbox/.env.example:0:0-0:0) provided.
  - Rotate credentials if exposed.
- Format validation:
  - Phone numbers should be in E.164 format.
  - Emails should be valid and reachable; consider future SMTP bounce handling.
- Scheduling:
  - Scheduled dispatch endpoint processes due messages (batch size 50).

## Deployment

- Ensure server has env vars set (never commit secrets).
- Run migrations/seed if necessary.
- Start: `npm run build && npm start`

## Roadmap

- Messaging Service SID (Twilio) for better routing and compliance.
- Phone normalization (libphonenumber) and stricter validation.
- Delivery status webhooks (mark delivered/read).
- Provider retries/backoff for transient errors.
