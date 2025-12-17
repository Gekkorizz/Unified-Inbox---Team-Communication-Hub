# Unified Inbox - Team Communication Hub

A modern, full-stack application for team-based customer engagement across multiple channels (SMS, WhatsApp, Email, Social Media).

## 🏗 Database Schema (ERD)

```mermaid
erDiagram
    Team ||--o{ User : has
    Team ||--o{ Contact : manages
    Team ||--o{ MessageTemplate : owns
    Team ||--o{ TwilioNumber : owns
    
    User ||--o{ Message : sends
    User ||--o{ Note : creates
    User ||--o{ Mention : receives
    User ||--o{ Contact : "assigned to"
    
    Contact ||--o{ Message : receives
    Contact ||--o{ Note : has
    Contact ||--o{ ScheduledMessage : has
    
    Message ||--o{ Attachment : contains
    
    Note ||--o{ Mention : contains
    
    Team {
        string id PK
        string name
        json settings
    }
    
    User {
        string id PK
        string email UK
        string name
        enum role
        string teamId FK
        string avatar
        boolean isOnline
        datetime lastSeen
        datetime createdAt
    }
    
    Contact {
        string id PK
        string name
        string phone UK
        string email UK
        json socialHandles
        string teamId FK
        datetime lastContactedAt
        enum status
        string assignedUserId FK
        string[] tags
        int priority
        datetime createdAt
        datetime updatedAt
    }
    
    Message {
        string id PK
        string contactId FK
        string userId FK
        enum channel
        string content
        enum status
        enum direction
        datetime timestamp
        json metadata
    }
    
    Note {
        string id PK
        string contactId FK
        string userId FK
        string content
        boolean isPrivate
        datetime createdAt
        datetime updatedAt
    }
    
    Mention {
        string id PK
        string noteId FK
        string userId FK
        int position
        int length
    }
    
    MessageTemplate {
        string id PK
        string name
        string content
        enum channel
        string teamId FK
        datetime createdAt
    }
    
    TwilioNumber {
        string id PK
        string phoneNumber UK
        string friendlyName
        boolean isTrial
        string teamId FK
        datetime createdAt
    }
    
    ScheduledMessage {
        string id PK
        string contactId FK
        enum channel
        string content
        datetime scheduledFor
        string status
    }
    
    Attachment {
        string id PK
        string messageId FK
        string url
        string type
        int size
    }
```

## ✨ Features Implemented

### 🎨 Modern UI/UX
- ✅ Professional design system with Tailwind CSS and Radix UI components
- ✅ Responsive layout with sidebar navigation
- ✅ Contact list with search functionality
- ✅ Modern message thread interface with channel badges
- ✅ Real-time message updates via Server-Sent Events
- ✅ Enhanced analytics dashboard with charts and metrics

### 🔧 Core Functionality
- ✅ User authentication system
- ✅ Contact management (create, view, delete)
- ✅ Multi-channel messaging (SMS, WhatsApp, Email)
- ✅ Message scheduling
- ✅ Notes system with private/public visibility
- ✅ Real-time updates and notifications

### 📊 Analytics & Reporting
- ✅ Analytics dashboard with key metrics
- ✅ Message volume tracking
- ✅ Success rate monitoring
- ✅ Response time analytics
- ✅ Channel breakdown visualization

## 🚀 Quick Start

### Prerequisites
- Node.js 18+ 
- PostgreSQL database
- Docker (optional, for database)

### Installation

1. **Clone and setup**
   ```bash
   cd unified-inbox
   npm install
   ```

2. **Environment Setup**
   ```bash
   cp .env.example .env
   ```
   
   Update `.env` with your configuration:
   ```env
   DATABASE_URL="postgresql://user:password@localhost:5432/unified_inbox"
   TWILIO_ACCOUNT_SID="your_twilio_sid"
   TWILIO_AUTH_TOKEN="your_twilio_token"
   TWILIO_PHONE_NUMBER="your_twilio_number"
   RESEND_API_KEY="your_resend_key"
   NEXT_PUBLIC_APP_URL="http://localhost:3000"
   ```

3. **Database Setup**
   ```bash
   npm run db:push
   npm run db:seed
   ```

4. **Start Development Server**
   ```bash
   npm run dev
   ```

5. **Access Application**
   - Main App: http://localhost:3000
   - Analytics: http://localhost:3000/analytics

## ✅ **NEWLY IMPLEMENTED FEATURES**

### 🔧 **Code Quality & Documentation**
- ✅ **Zod Validation**: Complete type-safe API validation for all endpoints
- ✅ **Enhanced Documentation**: Comprehensive API docs with JSDoc comments
- ✅ **ERD Diagram**: Complete database schema visualization (Mermaid)
- ✅ **Orchestration Factory**: Unified channel abstraction layer (`/lib/integrations.ts`)

### 🔒 **Security & Production**
- ✅ **Note Encryption**: Automatic encryption for private notes via Prisma middleware
- ✅ **Audit Trail**: Comprehensive logging for sensitive operations
- ✅ **Performance Monitoring**: Query performance tracking and optimization

### 📊 **Advanced Analytics**
- ✅ **Export Functionality**: CSV/JSON export for messages, contacts, and analytics
- ✅ **Custom Date Ranges**: Flexible analytics filtering
- ✅ **Team Performance Metrics**: Individual and team analytics

### 🤖 **Contact Management**
- ✅ **Auto-merge System**: Fuzzy matching algorithm for duplicate detection
- ✅ **Merge Confirmation UI**: Review and approve contact merges
- ✅ **Duplicate Detection**: Advanced similarity scoring with configurable thresholds

## 🏗 Architecture

### Tech Stack
- **Frontend**: Next.js 14, React, TypeScript, Tailwind CSS, Radix UI
- **Backend**: Next.js API Routes, Prisma ORM
- **Database**: PostgreSQL
- **Real-time**: Server-Sent Events (SSE)
- **Integrations**: Twilio (SMS/WhatsApp), Resend (Email)

### Project Structure
```
unified-inbox/
├── src/
│   ├── app/                 # Next.js app router
│   │   ├── api/            # API routes
│   │   ├── analytics/      # Analytics dashboard
│   │   └── page.tsx        # Main inbox interface
│   ├── components/         # Reusable UI components
│   │   ├── ui/            # Base UI components
│   │   ├── contact-list.tsx
│   │   ├── message-thread.tsx
│   │   └── notes-panel.tsx
│   └── lib/               # Utilities and integrations
│       ├── auth.ts        # Authentication logic
│       ├── db.ts          # Database connection
│       ├── events.ts      # SSE event system
│       └── integrations/  # Third-party integrations
├── prisma/
│   ├── schema.prisma      # Database schema
│   └── seed.js           # Database seeding
└── package.json
```

## 🔧 Development

### Available Scripts
- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run ESLint
- `npm run db:push` - Push schema changes to database
- `npm run db:seed` - Seed database with sample data
- `npm run db:reset` - Reset database with fresh data
- `npm run setup` - Complete setup (db:push + db:seed + dev)
- `npm run setup:full` - Full setup including dependencies
- `npm run test:api` - Test all API endpoints
- `npm run merge:contacts` - Find and merge duplicate contacts
- `npm run export:data` - Export analytics data

### Database Schema
The application uses a normalized schema with the following main entities:
- **Teams**: Multi-tenant organization structure
- **Users**: Team members with role-based access
- **Contacts**: Unified contact records across channels
- **Messages**: Normalized messages from all channels
- **Notes**: Team collaboration notes
- **ScheduledMessages**: Message scheduling system

## 🚀 Deployment

### Quick Deployment Guide

#### **Option 1: Vercel + Supabase (Recommended)**

1. **Set up Supabase Database:**
   ```bash
   # Go to supabase.com and create a new project
   # Copy the PostgreSQL connection string
   ```

2. **Deploy to Vercel:**
   ```bash
   # Go to vercel.com and import your GitHub repo
   # Add environment variables in Vercel dashboard
   ```

3. **Environment Variables for Vercel:**
   ```env
   DATABASE_URL=postgresql://postgres:[password]@[host]:5432/postgres
   TWILIO_ACCOUNT_SID=your_twilio_account_sid
   TWILIO_AUTH_TOKEN=your_twilio_auth_token
   TWILIO_PHONE_NUMBER=your_twilio_phone_number
   RESEND_API_KEY=your_resend_api_key
   NEXT_PUBLIC_APP_URL=https://your-app.vercel.app
   ENCRYPTION_KEY=generate_with_npm_run_deploy:prep
   ```

4. **Initialize Production Database:**
   ```bash
   # After deployment, run these commands:
   npx prisma db push
   npm run db:seed
   ```

#### **Option 2: Railway (Full-Stack Friendly)**

1. **Install Railway CLI:**
   ```bash
   npm install -g @railway/cli
   railway login
   ```

2. **Deploy:**
   ```bash
   railway init
   railway add postgresql
   railway up
   ```

3. **Configure Environment Variables in Railway Dashboard**

#### **Option 3: DigitalOcean App Platform**

1. **Create App from GitHub**
2. **Add Managed PostgreSQL Database**
3. **Configure Environment Variables**

### Deployment Preparation

Run the deployment preparation script:
```bash
npm run deploy:prep
```

This will generate:
- Required environment variables template
- Encryption key for production
- Deployment checklist
- Platform recommendations

### Production Checklist
- [ ] **Database**: Production PostgreSQL (Supabase/Railway/Neon)
- [ ] **Environment Variables**: All required vars configured
- [ ] **Twilio**: Production account with verified phone number
- [ ] **Domain**: Custom domain configured (optional)
- [ ] **SSL**: HTTPS enabled (automatic with most platforms)
- [ ] **Monitoring**: Error tracking and logging
- [ ] **Backups**: Database backup strategy

### Post-Deployment Steps

1. **Test Core Features:**
   ```bash
   # Test API endpoints
   curl https://your-app.com/api/test-db
   curl https://your-app.com/api/contacts
   ```

2. **Set up Webhooks:**
   - Configure Twilio webhook URL: `https://your-app.com/api/webhooks/twilio`
   - Test message sending and receiving

3. **Monitor Application:**
   - Check logs for errors
   - Monitor database performance
   - Test all integrations

### Troubleshooting

**Common Issues:**
- **Database Connection**: Ensure DATABASE_URL is correct
- **Twilio Webhooks**: Verify webhook URL is accessible
- **Environment Variables**: Check all required vars are set
- **Build Errors**: Run `npm run build` locally first

**Debug Commands:**
```bash
# Check database connection
npm run test:api

# View application logs
# (Platform-specific - check your deployment platform docs)
```

## 📝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License.
