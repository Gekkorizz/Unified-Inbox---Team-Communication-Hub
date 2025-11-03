# Unified Multi-Channel Inbox 📬

A full-stack unified communication platform that aggregates messages from SMS, WhatsApp, Email, and Social Media into a single inbox for seamless team-based customer engagement.

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Next.js](https://img.shields.io/badge/Next.js-14+-black)
![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-blue)
![Prisma](https://img.shields.io/badge/Prisma-Latest-2D3748)

## 🎯 Project Overview

This platform eliminates context-switching for sales and support teams by centralizing all customer communications in one place. Send messages, schedule automations, collaborate with your team, and track engagement metrics—all from a single, intuitive interface.

**Key Features:**
- 📨 Unified inbox across SMS, WhatsApp, Email, and Social Media
- 👥 Team collaboration with real-time presence and @mentions
- 📅 Message scheduling and automation
- 📊 Analytics dashboard with engagement metrics
- 🔒 Role-based access control (Viewer/Editor/Admin)
- 🎨 Modern, responsive UI similar to OpenPhone

## 🏗️ Tech Stack

### Core
- **Frontend/Backend:** Next.js 14+ (App Router, TypeScript)
- **Database:** PostgreSQL with Prisma ORM
- **Authentication:** Better Auth with Google OAuth
- **Styling:** Tailwind CSS
- **State Management:** React Query

### Integrations
- **Twilio:** SMS & WhatsApp messaging
- **Resend:** Email delivery (optional)
- **Twitter API v2:** Direct messages (optional)
- **Facebook Graph API:** Messenger (optional)

### Code Quality
- ESLint & Prettier
- TypeScript strict mode
- Zod validation
- JSDoc documentation

## 📋 Table of Contents

- [Quick Start](#-quick-start)
- [Environment Setup](#-environment-setup)
- [Database Schema](#-database-schema)
- [Architecture](#-architecture)
- [API Documentation](#-api-documentation)
- [Integration Comparison](#-integration-comparison)
- [Key Decisions](#-key-architectural-decisions)
- [Deployment](#-deployment)
- [Contributing](#-contributing)

## 🚀 Quick Start

### Prerequisites

- Node.js 18+ and npm/yarn
- PostgreSQL 14+ (local or cloud)
- Twilio account with trial number
- Git

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/unified-inbox.git
   cd unified-inbox
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Set up environment variables**
   ```bash
   cp .env.example .env
   # Edit .env with your credentials (see Environment Setup below)
   ```

4. **Initialize database**
   ```bash
   npx prisma generate
   npx prisma db push
   npx prisma db seed  # Optional: seed with test data
   ```

5. **Start development server**
   ```bash
   npm run dev
   ```

6. **Open your browser**
   ```
   Navigate to http://localhost:3000
   ```

### One-Command Setup

```bash
npm run setup  # Runs installation, DB setup, and starts dev server
```

## 🔐 Environment Setup

Create a `.env` file in the root directory:

```env
# Database
DATABASE_URL="postgresql://username:password@localhost:5432/unified_inbox"

# Better Auth
BETTER_AUTH_SECRET="your-secret-key-min-32-chars"
BETTER_AUTH_URL="http://localhost:3000"
GOOGLE_CLIENT_ID="your-google-client-id"
GOOGLE_CLIENT_SECRET="your-google-client-secret"

# Twilio (Required)
TWILIO_ACCOUNT_SID="ACxxxxxxxxxxxxxxxxxxxxx"
TWILIO_AUTH_TOKEN="your-auth-token"
TWILIO_PHONE_NUMBER="+1234567890"
TWILIO_WHATSAPP_NUMBER="whatsapp:+14155238886"  # Twilio Sandbox

# Email (Optional)
RESEND_API_KEY="re_xxxxxxxxxxxxx"

# Social Media (Optional)
TWITTER_API_KEY="your-twitter-api-key"
TWITTER_API_SECRET="your-twitter-api-secret"
TWITTER_BEARER_TOKEN="your-bearer-token"
FACEBOOK_APP_ID="your-facebook-app-id"
FACEBOOK_APP_SECRET="your-facebook-app-secret"

# Application
NODE_ENV="development"
NEXT_PUBLIC_APP_URL="http://localhost:3000"
```

### Getting API Keys

#### Twilio Setup
1. Sign up at [twilio.com/try-twilio](https://twilio.com/try-twilio)
2. Get a trial phone number (SMS + WhatsApp enabled)
3. Copy Account SID and Auth Token from dashboard
4. Enable WhatsApp Sandbox: Console → Messaging → Try it out → Send WhatsApp message
5. Configure webhook URL: `https://yourdomain.com/api/webhooks/twilio`

#### Google OAuth Setup
1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project
3. Enable Google+ API
4. Create OAuth 2.0 credentials
5. Add authorized redirect URI: `http://localhost:3000/api/auth/callback/google`

#### Resend Setup (Optional)
1. Sign up at [resend.com](https://resend.com)
2. Verify your domain or use onboarding email
3. Generate API key from dashboard

## 💾 Database Schema

### Entity Relationship Diagram

```mermaid
erDiagram
    User ||--o{ Message : sends
    User ||--o{ Note : writes
    User }o--|| Team : belongs_to
    Contact ||--o{ Message : receives
    Contact ||--o{ Note : about
    Message ||--o{ Attachment : has
    Team ||--o{ Contact : manages
    
    User {
        string id PK
        string email UK
        string name
        string role
        string teamId FK
        datetime createdAt
    }
    
    Contact {
        string id PK
        string phone UK
        string email UK
        json socialHandles
        string teamId FK
        datetime lastContactedAt
    }
    
    Message {
        string id PK
        string contactId FK
        string userId FK
        string channel
        string content
        string status
        string direction
        datetime timestamp
        json metadata
    }
    
    Note {
        string id PK
        string contactId FK
        string userId FK
        string content
        boolean isPrivate
        json mentions
        datetime createdAt
    }
    
    ScheduledMessage {
        string id PK
        string contactId FK
        string channel
        string content
        datetime scheduledFor
        string status
    }
    
    Team {
        string id PK
        string name
        json settings
    }
    
    Attachment {
        string id PK
        string messageId FK
        string url
        string type
        int size
    }
```

### Key Tables

**User**: Authentication, roles (viewer/editor/admin), team membership

**Contact**: Unified contact information with phone, email, social handles, auto-merge duplicates

**Message**: Normalized messages across all channels with status tracking

**Note**: Internal team notes with privacy controls and @mentions

**ScheduledMessage**: Message automation queue with cron-like scheduling

**Team**: Multi-team support with isolated data

**Attachment**: Media files linked to messages (MMS, WhatsApp media)

### Database Commands

```bash
# Generate Prisma Client
npx prisma generate

# Create migration
npx prisma migrate dev --name init

# Push schema changes (dev only)
npx prisma db push

# Open Prisma Studio
npx prisma studio

# Reset database
npx prisma migrate reset
```

## 🏛️ Architecture

### Project Structure

```
unified-inbox/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   └── register/
│   ├── (dashboard)/
│   │   ├── inbox/
│   │   ├── contacts/
│   │   ├── analytics/
│   │   └── settings/
│   ├── api/
│   │   ├── auth/
│   │   ├── messages/
│   │   ├── contacts/
│   │   ├── webhooks/
│   │   │   ├── twilio/
│   │   │   ├── facebook/
│   │   │   └── twitter/
│   │   └── scheduled/
│   └── layout.tsx
├── components/
│   ├── ui/              # Shadcn components
│   ├── inbox/
│   ├── contacts/
│   ├── composer/
│   └── analytics/
├── lib/
│   ├── integrations/
│   │   ├── factory.ts   # Channel factory pattern
│   │   ├── twilio.ts
│   │   ├── resend.ts
│   │   ├── twitter.ts
│   │   └── facebook.ts
│   ├── db.ts            # Prisma client
│   ├── auth.ts          # Better Auth config
│   ├── validations.ts   # Zod schemas
│   └── utils.ts
├── prisma/
│   ├── schema.prisma
│   ├── migrations/
│   └── seed.ts
├── public/
├── types/
│   └── index.ts
└── package.json
```

### Data Flow

```mermaid
sequenceDiagram
    participant User
    participant NextJS
    participant Prisma
    participant PostgreSQL
    participant Twilio
    participant WebSocket
    
    User->>NextJS: Send message via UI
    NextJS->>Prisma: Validate & save message
    Prisma->>PostgreSQL: INSERT message
    NextJS->>Twilio: Send via API
    Twilio-->>User: Deliver to recipient
    
    Twilio->>NextJS: Webhook (incoming message)
    NextJS->>Prisma: Save incoming message
    NextJS->>WebSocket: Broadcast to team
    WebSocket-->>User: Real-time update in UI
```

### Integration Factory Pattern

```typescript
// lib/integrations/factory.ts
interface MessagePayload {
  to: string;
  content: string;
  attachments?: string[];
}

interface ChannelAdapter {
  send(payload: MessagePayload): Promise<void>;
  receive(webhookData: any): Promise<Message>;
}

export function createSender(channel: string): ChannelAdapter {
  switch (channel) {
    case 'sms':
    case 'whatsapp':
      return new TwilioAdapter(channel);
    case 'email':
      return new ResendAdapter();
    case 'twitter':
      return new TwitterAdapter();
    default:
      throw new Error(`Unsupported channel: ${channel}`);
  }
}
```

## 📡 API Documentation

### Authentication

All API routes (except webhooks) require authentication via Better Auth session.

#### POST `/api/auth/login`
Login with credentials or OAuth

**Request:**
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response:**
```json
{
  "user": {
    "id": "user_123",
    "email": "user@example.com",
    "role": "editor"
  },
  "token": "session_token"
}
```

### Messages

#### GET `/api/messages`
Fetch messages with filters

**Query Parameters:**
- `contactId`: Filter by contact
- `channel`: Filter by channel (sms|whatsapp|email|twitter|facebook)
- `status`: Filter by status (sent|delivered|read|failed)
- `limit`: Number of messages (default: 50)
- `cursor`: Pagination cursor

**Response:**
```json
{
  "messages": [
    {
      "id": "msg_123",
      "contactId": "contact_456",
      "channel": "whatsapp",
      "content": "Hello!",
      "status": "delivered",
      "direction": "outbound",
      "timestamp": "2025-11-02T10:30:00Z",
      "attachments": []
    }
  ],
  "nextCursor": "cursor_789"
}
```

#### POST `/api/messages/send`
Send a new message

**Request:**
```json
{
  "contactId": "contact_456",
  "channel": "whatsapp",
  "content": "Hello from our team!",
  "attachments": ["https://example.com/image.jpg"],
  "scheduledFor": "2025-11-03T10:00:00Z"  // Optional
}
```

**Response:**
```json
{
  "message": {
    "id": "msg_789",
    "status": "sent",
    "timestamp": "2025-11-02T10:31:00Z"
  }
}
```

### Contacts

#### GET `/api/contacts`
List contacts with search

**Query Parameters:**
- `search`: Search term
- `channel`: Filter by channel
- `limit`: Number of results

#### POST `/api/contacts`
Create or update contact

**Request:**
```json
{
  "phone": "+1234567890",
  "email": "contact@example.com",
  "name": "John Doe",
  "socialHandles": {
    "twitter": "@johndoe",
    "facebook": "john.doe.123"
  }
}
```

### Webhooks

#### POST `/api/webhooks/twilio`
Receive incoming SMS/WhatsApp messages from Twilio

**Webhook Signature Verification:** Validates `X-Twilio-Signature` header

**Twilio Payload:**
```
From=+1234567890
To=+0987654321
Body=Hello!
MediaUrl0=https://...
```

**Response:** `200 OK` with TwiML (optional)

#### POST `/api/webhooks/facebook`
Receive Facebook Messenger messages

**Verification:** Validates `X-Hub-Signature-256` header

#### POST `/api/webhooks/twitter`
Receive Twitter DM events

**Verification:** Validates `X-Twitter-Webhooks-Signature` header

## 📊 Integration Comparison

| Channel | Latency | Cost | Reliability | Media Support | Rate Limits |
|---------|---------|------|-------------|---------------|-------------|
| **SMS (Twilio)** | 50-500ms | $0.0075/msg (US) | 99.95% | MMS (images, video) | 1 msg/sec (trial) |
| **WhatsApp (Twilio)** | 100-800ms | $0.005/msg | 99.9% | Images, videos, docs, audio | 80 msg/sec |
| **Email (Resend)** | 200ms-2s | $0.0001/email | 99.5% | Full HTML, attachments | 100/day (free) |
| **Twitter DM** | 500ms-3s | Free | 98% | Images, GIFs | 300/15min |
| **Facebook Messenger** | 300ms-2s | Free | 99% | Images, videos, files | Variable |

### Cost Analysis (Monthly for 10,000 messages)

- **SMS:** $75 (US domestic)
- **WhatsApp:** $50 (business-initiated)
- **Email:** $1 (Resend)
- **Social Media:** Free

### Reliability Considerations

**SMS/WhatsApp (Twilio):**
- ✅ Carrier-grade infrastructure
- ✅ Delivery receipts
- ⚠️ Phone number verification required for trial
- ⚠️ Geographic restrictions

**Email (Resend):**
- ✅ High deliverability rates
- ⚠️ Spam filters can block
- ⚠️ Domain warming required for volume

**Social Media:**
- ✅ Direct user engagement
- ⚠️ API rate limits can be restrictive
- ⚠️ Requires user opt-in/approval
- ⚠️ Platform policy changes risk

## 🎯 Key Architectural Decisions

### 1. Message Normalization Strategy
**Decision:** Single `Message` table with `channel` discriminator

**Rationale:**
- Simplifies querying across channels
- Enables unified inbox view without complex joins
- Channel-specific metadata stored in JSON column
- Easier to add new channels (just add enum value)

**Trade-offs:**
- Some denormalization (channel-specific fields in JSON)
- Slightly larger table size
- ✅ Better than: Separate table per channel (complex joins, schema drift)

### 2. Real-time Collaboration via Server-Sent Events (SSE)
**Decision:** Use SSE instead of WebSockets for real-time updates

**Rationale:**
- Simpler implementation in Next.js App Router
- Automatic reconnection in browser
- Works with standard HTTP/2
- Sufficient for unidirectional updates (server → client)

**Trade-offs:**
- One-way communication (adequate for our use case)
- ✅ Better than: WebSockets (more complex, requires separate server)

### 3. Webhook Security Pattern
**Decision:** Signature verification for all webhook endpoints

**Implementation:**
```typescript
// lib/webhooks/verify.ts
export async function verifyTwilioSignature(
  signature: string,
  url: string,
  params: Record<string, string>
): Promise<boolean> {
  const authToken = process.env.TWILIO_AUTH_TOKEN;
  const data = Object.keys(params)
    .sort()
    .map(key => `${key}${params[key]}`)
    .join('');
  
  const hmac = crypto
    .createHmac('sha1', authToken)
    .update(url + data)
    .digest('base64');
    
  return signature === hmac;
}
```

**Rationale:**
- Prevents replay attacks
- Ensures webhooks are from legitimate sources
- Industry standard (HMAC-SHA256)

### 4. Scheduling Implementation
**Decision:** Postgres-based queue with cron job polling

**Rationale:**
- Simple to implement and debug
- No external dependencies (Redis, BullMQ)
- Sufficient for moderate volume (<1000 scheduled msgs/day)
- Transactional consistency with message data

**Trade-offs:**
- Not suitable for high-frequency scheduling
- ✅ Better than: In-memory timers (lost on restart)
- 🔄 Future: Migrate to BullMQ if volume increases

### 5. Contact Deduplication via Fuzzy Matching
**Decision:** Use Levenshtein distance for name matching + exact match on phone/email

**Algorithm:**
```typescript
// lib/contacts/deduplicate.ts
export async function findDuplicates(contact: Contact): Promise<Contact[]> {
  // Exact match on phone or email
  const exactMatches = await prisma.contact.findMany({
    where: {
      OR: [
        { phone: contact.phone },
        { email: contact.email }
      ]
    }
  });
  
  if (exactMatches.length > 0) return exactMatches;
  
  // Fuzzy match on name (Levenshtein distance < 3)
  const allContacts = await prisma.contact.findMany();
  return allContacts.filter(c => 
    levenshtein(c.name, contact.name) < 3
  );
}
```

**Rationale:**
- Prevents duplicate contacts from typos
- User confirms merges (not automatic)
- Balance between precision and recall

### 6. Role-Based Access Control (RBAC)
**Decision:** Three roles with Prisma middleware enforcement

**Roles:**
- **Viewer:** Read-only access to inbox and contacts
- **Editor:** Can send messages and edit contacts
- **Admin:** Full access including team settings and user management

**Implementation:**
```typescript
// lib/auth/rbac.ts
export const permissions = {
  viewer: ['read:messages', 'read:contacts'],
  editor: ['read:messages', 'read:contacts', 'write:messages', 'write:contacts'],
  admin: ['*']
};

// Middleware on API routes
export async function requirePermission(permission: string) {
  const session = await getSession();
  const userPermissions = permissions[session.user.role];
  
  if (!userPermissions.includes('*') && !userPermissions.includes(permission)) {
    throw new UnauthorizedError();
  }
}
```

**Rationale:**
- Simple to understand and implement
- Covers 90% of use cases
- Easily extensible to fine-grained permissions later

### 7. Optimistic UI Updates with React Query
**Decision:** Use React Query mutations with optimistic updates

**Example:**
```typescript
const sendMessage = useMutation({
  mutationFn: (message) => api.sendMessage(message),
  onMutate: async (newMessage) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['messages'] });
    
    // Snapshot previous value
    const previousMessages = queryClient.getQueryData(['messages']);
    
    // Optimistically update
    queryClient.setQueryData(['messages'], (old) => [...old, newMessage]);
    
    return { previousMessages };
  },
  onError: (err, newMessage, context) => {
    // Rollback on error
    queryClient.setQueryData(['messages'], context.previousMessages);
  },
  onSettled: () => {
    // Refetch to ensure consistency
    queryClient.invalidateQueries({ queryKey: ['messages'] });
  }
});
```

**Rationale:**
- Instant UI feedback (perceived performance)
- Automatic error handling with rollback
- Built-in retry logic

## 🚀 Deployment

### Vercel Deployment (Recommended)

1. **Push to GitHub**
   ```bash
   git push origin main
   ```

2. **Connect to Vercel**
   - Import project from GitHub
   - Add environment variables
   - Deploy

3. **Configure Webhooks**
   - Update Twilio webhook URL to production domain
   - Update Facebook/Twitter webhook URLs

### Docker Deployment

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npx prisma generate
RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

```bash
# Build and run
docker build -t unified-inbox .
docker run -p 3000:3000 --env-file .env unified-inbox
```

### Database Migration in Production

```bash
# Apply migrations
npx prisma migrate deploy

# Or use Prisma Cloud (automatic)
```

## 🧪 Testing

### Run Tests

```bash
# Unit tests
npm run test

# E2E tests
npm run test:e2e

# Integration tests
npm run test:integration
```

### Test Coverage

```bash
npm run test:coverage
```

### Manual Testing Checklist

- [ ] Send SMS message
- [ ] Receive SMS via webhook
- [ ] Send WhatsApp message
- [ ] Receive WhatsApp via webhook
- [ ] Schedule message for future
- [ ] Create and view contact
- [ ] Add note with @mention
- [ ] View analytics dashboard
- [ ] Test role permissions (viewer/editor/admin)
- [ ] Verify webhook signatures

## 📈 Performance Considerations

### Database Indexing

```prisma
model Message {
  @@index([contactId, timestamp])
  @@index([channel, status])
  @@index([userId])
}

model Contact {
  @@index([phone])
  @@index([email])
  @@index([teamId])
}
```

### Caching Strategy

- React Query caches API responses (5 min default)
- Server-side rendering for initial page load
- Static generation for public pages

### Optimization Tips

- Use `React.memo()` for expensive components
- Implement virtual scrolling for long message lists
- Lazy load images and media
- Debounce search inputs (300ms)

## 🐛 Troubleshooting

### Common Issues

**Twilio webhook not receiving messages:**
- Check webhook URL is publicly accessible (use ngrok for local dev)
- Verify webhook signature validation is correct
- Check Twilio console logs

**Database connection errors:**
- Verify `DATABASE_URL` is correct
- Check PostgreSQL is running
- Run `npx prisma db push` to sync schema

**Authentication not working:**
- Verify `BETTER_AUTH_SECRET` is set (min 32 chars)
- Check `BETTER_AUTH_URL` matches your domain
- Clear browser cookies and try again

**Real-time updates not working:**
- Check SSE endpoint is accessible
- Verify CORS settings for production
- Check browser console for connection errors

## 📚 Additional Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [Prisma Documentation](https://www.prisma.io/docs)
- [Twilio API Reference](https://www.twilio.com/docs/api)
- [Better Auth Documentation](https://better-auth.com/docs)

## 🤝 Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Code Style

- Follow ESLint configuration
- Run Prettier before committing
- Write JSDoc comments for public functions
- Add tests for new features

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👥 Authors

- Your Name - [@yourusername](https://github.com/yourusername)

## 🙏 Acknowledgments

- Attack Capital for the assignment brief
- Twilio for communication APIs
- Next.js team for the amazing framework
- Open source community

---

**Built with ❤️ for Attack Capital Assignment**

For questions or issues, please contact: [your-email@example.com]