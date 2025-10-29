# AI Super Assistant Platform - Developer Guide

## Table of Contents

1. [Getting Started](#getting-started)
2. [Development Setup](#development-setup)
3. [Architecture Overview](#architecture-overview)
4. [Database Schema Documentation](#database-schema-documentation)
5. [API Reference](#api-reference)
6. [Edge Functions Documentation](#edge-functions-documentation)
7. [Frontend Component Library](#frontend-component-library)
8. [Testing Guide](#testing-guide)
9. [Deployment Guide](#deployment-guide)
10. [Troubleshooting](#troubleshooting)

---

## Getting Started

### Prerequisites

- **Node.js**: v18 or higher
- **pnpm**: v8 or higher
- **Git**: Latest version
- **Supabase Account**: Free tier is sufficient for development
- **Text Editor**: VS Code recommended with TypeScript/React extensions

### Quick Start

```bash
# Clone the repository
git clone <repository-url>
cd ai-super-assistant

# Install dependencies
pnpm install

# Create environment file
cp .env.example .env

# Configure your Supabase credentials in .env
# VITE_SUPABASE_URL=your_supabase_url
# VITE_SUPABASE_ANON_KEY=your_anon_key

# Start development server
pnpm run dev
```

Your application will be running at `http://localhost:5173`

---

## Development Setup

### 1. Environment Configuration

Create a `.env` file in the project root:

```env
# Supabase Configuration
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key

# Optional: AI Provider API Keys (for future integration)
VITE_OPENAI_API_KEY=your-openai-key
VITE_ANTHROPIC_API_KEY=your-anthropic-key
VITE_GOOGLE_API_KEY=your-google-key
```

### 2. Database Setup

Apply the database schema and Row Level Security policies:

```bash
# Navigate to Supabase Dashboard > SQL Editor
# Run the following SQL files in order:

# 1. Apply schema
cat database-schema.sql

# 2. Apply RLS policies
cat rls-policies.sql
```

### 3. Project Structure

```
ai-super-assistant/
├── src/
│   ├── components/          # Reusable React components
│   │   ├── Navigation.tsx   # Main navigation component
│   │   └── ErrorBoundary.tsx
│   ├── pages/               # Page components (MPA with React Router)
│   │   ├── HomePage.tsx
│   │   ├── AIChat.tsx
│   │   ├── BrowserAutomation.tsx
│   │   ├── ITSupport.tsx
│   │   ├── BusinessManagement.tsx
│   │   ├── Pricing.tsx
│   │   ├── Documentation.tsx
│   │   └── Contact.tsx
│   ├── contexts/            # React contexts
│   │   └── AuthContext.tsx  # Authentication context
│   ├── hooks/               # Custom React hooks
│   │   └── use-mobile.tsx
│   ├── lib/                 # Utilities and configurations
│   │   ├── supabase.ts      # Supabase client setup
│   │   └── utils.ts         # Helper functions
│   ├── App.tsx              # Main app component with routing
│   ├── main.tsx             # Application entry point
│   └── index.css            # Global styles
├── supabase/
│   └── functions/           # Edge functions
│       ├── ai-chat-message/
│       ├── create-support-ticket/
│       └── execute-automation/
├── database-schema.sql      # Database schema definition
├── rls-policies.sql         # Row Level Security policies
├── tailwind.config.js       # Tailwind CSS configuration
├── vite.config.ts           # Vite configuration
└── package.json             # Project dependencies
```

### 4. Development Commands

```bash
# Start development server
pnpm run dev

# Build for production
pnpm run build

# Preview production build
pnpm run preview

# Lint code
pnpm run lint

# Clean dependencies (if needed)
pnpm run clean
```

---

## Architecture Overview

### Technology Stack

**Frontend:**
- **React 18** - UI framework
- **TypeScript** - Type safety
- **Vite** - Build tool and dev server
- **React Router** - Multi-page routing
- **Tailwind CSS** - Utility-first styling
- **Radix UI** - Accessible component primitives
- **Supabase JS Client** - Backend integration

**Backend:**
- **Supabase** - Backend-as-a-Service platform
  - PostgreSQL database
  - Authentication
  - Edge Functions (Deno runtime)
  - Row Level Security (RLS)
  - Real-time subscriptions

**Design System:**
- Modern Minimalism Premium
- Custom Tailwind configuration with design tokens
- Responsive and accessible components

### Application Flow

```
User Request → React Router → Page Component → Supabase Client → Edge Function → Database
                                                     ↓
                                              Auth Context
                                                     ↓
                                              RLS Policies
```

---

## Database Schema Documentation

### Core Tables

#### 1. Profiles Table

Stores user profile information linked to Supabase Auth.

```sql
CREATE TABLE profiles (
    id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
    email TEXT NOT NULL,
    full_name TEXT,
    avatar_url TEXT,
    role TEXT DEFAULT 'user' CHECK (role IN ('admin', 'user', 'client')),
    organization TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

**Fields:**
- `id` - UUID linked to auth.users
- `email` - User email address
- `full_name` - Optional display name
- `avatar_url` - Profile picture URL
- `role` - User role (admin, user, client)
- `organization` - Company/organization name
- `created_at` / `updated_at` - Timestamps

#### 2. Conversations Table

AI chat conversation sessions.

```sql
CREATE TABLE conversations (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL,
    title TEXT NOT NULL DEFAULT 'New Conversation',
    model_provider TEXT NOT NULL CHECK (model_provider IN ('openai', 'claude', 'gemini')),
    model_name TEXT NOT NULL,
    context_summary TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

**Fields:**
- `id` - Unique conversation identifier
- `user_id` - Owner of the conversation
- `title` - Conversation title
- `model_provider` - AI provider (openai, claude, gemini)
- `model_name` - Specific model used
- `context_summary` - Summary for context management

#### 3. Messages Table

Individual messages within conversations.

```sql
CREATE TABLE messages (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    conversation_id UUID NOT NULL,
    role TEXT NOT NULL CHECK (role IN ('user', 'assistant', 'system')),
    content TEXT NOT NULL,
    token_count INTEGER DEFAULT 0,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

**Fields:**
- `id` - Message identifier
- `conversation_id` - Parent conversation
- `role` - Message sender (user, assistant, system)
- `content` - Message text
- `token_count` - Estimated token usage

#### 4. Automation Workflows Table

Browser automation workflow definitions.

```sql
CREATE TABLE automation_workflows (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL,
    name TEXT NOT NULL,
    description TEXT,
    framework TEXT NOT NULL CHECK (framework IN ('playwright', 'selenium', 'puppeteer')),
    script_code TEXT NOT NULL,
    schedule_cron TEXT,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

**Fields:**
- `id` - Workflow identifier
- `user_id` - Workflow owner
- `name` - Workflow name
- `framework` - Automation framework
- `script_code` - Executable script
- `schedule_cron` - Optional schedule
- `is_active` - Enabled status

#### 5. Automation Jobs Table

Execution records for automation workflows.

```sql
CREATE TABLE automation_jobs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    workflow_id UUID NOT NULL,
    user_id UUID NOT NULL,
    status TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'running', 'completed', 'failed', 'cancelled')),
    started_at TIMESTAMPTZ,
    completed_at TIMESTAMPTZ,
    result_data JSONB,
    error_message TEXT,
    execution_time_ms INTEGER,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### 6. Support Tickets Table

IT support ticket management.

```sql
CREATE TABLE support_tickets (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    ticket_number TEXT UNIQUE NOT NULL,
    client_id UUID,
    created_by UUID NOT NULL,
    assigned_to UUID,
    title TEXT NOT NULL,
    description TEXT NOT NULL,
    priority TEXT NOT NULL DEFAULT 'medium' CHECK (priority IN ('low', 'medium', 'high', 'urgent')),
    status TEXT NOT NULL DEFAULT 'open' CHECK (status IN ('open', 'in_progress', 'pending', 'resolved', 'closed')),
    category TEXT NOT NULL CHECK (category IN ('incident', 'request', 'problem', 'change')),
    resolution_notes TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    resolved_at TIMESTAMPTZ
);
```

#### 7. Ticket Comments Table

Comments and updates on support tickets.

```sql
CREATE TABLE ticket_comments (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    ticket_id UUID NOT NULL,
    user_id UUID NOT NULL,
    comment TEXT NOT NULL,
    is_internal BOOLEAN DEFAULT false,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### 8. Clients Table

Client/customer management.

```sql
CREATE TABLE clients (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name TEXT NOT NULL,
    company_name TEXT NOT NULL,
    email TEXT NOT NULL,
    phone TEXT,
    subscription_tier TEXT DEFAULT 'basic' CHECK (subscription_tier IN ('free', 'basic', 'professional', 'enterprise')),
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### 9. Projects Table

Client project tracking.

```sql
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    client_id UUID NOT NULL,
    name TEXT NOT NULL,
    description TEXT,
    status TEXT NOT NULL DEFAULT 'planning' CHECK (status IN ('planning', 'active', 'on_hold', 'completed', 'cancelled')),
    start_date DATE,
    end_date DATE,
    budget NUMERIC(12, 2),
    created_by UUID NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### Database Indexes

```sql
CREATE INDEX idx_conversations_user_id ON conversations(user_id);
CREATE INDEX idx_messages_conversation_id ON messages(conversation_id);
CREATE INDEX idx_automation_jobs_workflow_id ON automation_jobs(workflow_id);
CREATE INDEX idx_support_tickets_client_id ON support_tickets(client_id);
CREATE INDEX idx_projects_client_id ON projects(client_id);
```

### Row Level Security (RLS) Policies

All tables have RLS enabled. Key policies include:

**Profiles:**
- Users can view and update their own profile
- Service role has full access

**Conversations & Messages:**
- Users can only access their own conversations and messages
- Service role has full access

**Automation:**
- Users can only manage their own workflows and jobs
- Service role has full access

**Support Tickets:**
- Users can access tickets they created or are assigned to
- Service role has full access

**Clients & Projects:**
- All authenticated users can view clients
- Service role has full access

---

## API Reference

### Supabase Client Usage

#### Initialize Client

```typescript
import { supabase } from '@/lib/supabase';
```

#### Authentication

```typescript
// Sign up
const { data, error } = await supabase.auth.signUp({
  email: 'user@example.com',
  password: 'password123',
});

// Sign in
const { data, error } = await supabase.auth.signInWithPassword({
  email: 'user@example.com',
  password: 'password123',
});

// Sign out
const { error } = await supabase.auth.signOut();

// Get current user
const { data: { user } } = await supabase.auth.getUser();
```

#### Database Operations

**Select Data:**

```typescript
// Get all conversations for current user
const { data, error } = await supabase
  .from('conversations')
  .select('*')
  .eq('user_id', user.id)
  .order('created_at', { ascending: false });

// Get conversation with messages
const { data, error } = await supabase
  .from('conversations')
  .select(`
    *,
    messages (*)
  `)
  .eq('id', conversationId)
  .single();
```

**Insert Data:**

```typescript
// Create new conversation
const { data, error } = await supabase
  .from('conversations')
  .insert({
    user_id: user.id,
    title: 'New Chat',
    model_provider: 'openai',
    model_name: 'gpt-4',
  })
  .select()
  .single();

// Create message
const { data, error } = await supabase
  .from('messages')
  .insert({
    conversation_id: conversationId,
    role: 'user',
    content: 'Hello!',
    token_count: 2,
  })
  .select()
  .single();
```

**Update Data:**

```typescript
// Update conversation title
const { data, error } = await supabase
  .from('conversations')
  .update({ title: 'Updated Title' })
  .eq('id', conversationId)
  .select()
  .single();
```

**Delete Data:**

```typescript
// Delete conversation (cascades to messages)
const { error } = await supabase
  .from('conversations')
  .delete()
  .eq('id', conversationId);
```

#### Real-time Subscriptions

```typescript
// Subscribe to new messages
const channel = supabase
  .channel('messages')
  .on(
    'postgres_changes',
    {
      event: 'INSERT',
      schema: 'public',
      table: 'messages',
      filter: `conversation_id=eq.${conversationId}`,
    },
    (payload) => {
      console.log('New message:', payload.new);
    }
  )
  .subscribe();

// Unsubscribe
channel.unsubscribe();
```

#### Storage Operations

```typescript
// Upload file
const { data, error } = await supabase.storage
  .from('avatars')
  .upload(`${user.id}/avatar.png`, file);

// Get public URL
const { data } = supabase.storage
  .from('avatars')
  .getPublicUrl(`${user.id}/avatar.png`);

// Download file
const { data, error } = await supabase.storage
  .from('avatars')
  .download(`${user.id}/avatar.png`);
```

---

## Edge Functions Documentation

Edge Functions run on Deno and provide serverless backend logic.

### 1. AI Chat Message Function

**Endpoint:** `/functions/v1/ai-chat-message`

**Purpose:** Handle AI chat message creation and response generation

**Request:**

```typescript
POST /functions/v1/ai-chat-message
Headers:
  Authorization: Bearer <user-token>
  Content-Type: application/json

Body:
{
  "conversation_id": "uuid",
  "role": "user",
  "content": "Hello, AI!",
  "model_provider": "openai"
}
```

**Response:**

```json
{
  "data": {
    "userMessage": {
      "id": "uuid",
      "conversation_id": "uuid",
      "role": "user",
      "content": "Hello, AI!",
      "token_count": 3,
      "created_at": "2025-10-29T10:00:00Z"
    },
    "aiMessage": {
      "id": "uuid",
      "conversation_id": "uuid",
      "role": "assistant",
      "content": "Hello! How can I help you?",
      "token_count": 6,
      "created_at": "2025-10-29T10:00:01Z"
    }
  }
}
```

**Implementation:**

```typescript
// /supabase/functions/ai-chat-message/index.ts
Deno.serve(async (req) => {
  const corsHeaders = {
    'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Headers': 'authorization, x-client-info, apikey, content-type',
  };

  if (req.method === 'OPTIONS') {
    return new Response(null, { status: 200, headers: corsHeaders });
  }

  try {
    const { conversation_id, content, model_provider } = await req.json();
    
    // Verify authentication
    const authHeader = req.headers.get('authorization');
    // ... authentication logic
    
    // Save user message
    // Call AI API
    // Save AI response
    
    return new Response(JSON.stringify({ data }), {
      headers: { ...corsHeaders, 'Content-Type': 'application/json' }
    });
  } catch (error) {
    return new Response(JSON.stringify({ error: error.message }), {
      status: 500,
      headers: { ...corsHeaders, 'Content-Type': 'application/json' }
    });
  }
});
```

### 2. Create Support Ticket Function

**Endpoint:** `/functions/v1/create-support-ticket`

**Purpose:** Create new IT support tickets with auto-generated ticket numbers

**Request:**

```typescript
POST /functions/v1/create-support-ticket
Headers:
  Authorization: Bearer <user-token>
  Content-Type: application/json

Body:
{
  "title": "Login issue",
  "description": "Cannot access my account",
  "priority": "high",
  "category": "incident",
  "client_id": "uuid"  // optional
}
```

**Response:**

```json
{
  "data": {
    "id": "uuid",
    "ticket_number": "TKT-1730198400000-ABC12",
    "title": "Login issue",
    "description": "Cannot access my account",
    "priority": "high",
    "status": "open",
    "category": "incident",
    "created_by": "uuid",
    "created_at": "2025-10-29T10:00:00Z"
  }
}
```

### 3. Execute Automation Function

**Endpoint:** `/functions/v1/execute-automation`

**Purpose:** Execute browser automation workflows

**Request:**

```typescript
POST /functions/v1/execute-automation
Headers:
  Authorization: Bearer <user-token>
  Content-Type: application/json

Body:
{
  "workflow_id": "uuid"
}
```

### Deploying Edge Functions

```bash
# Install Supabase CLI
npm install -g supabase

# Login to Supabase
supabase login

# Link to your project
supabase link --project-ref your-project-ref

# Deploy a function
supabase functions deploy ai-chat-message

# Deploy all functions
supabase functions deploy
```

### Testing Edge Functions Locally

```bash
# Serve functions locally
supabase functions serve

# Test with curl
curl -i --location --request POST 'http://localhost:54321/functions/v1/ai-chat-message' \
  --header 'Authorization: Bearer YOUR_ANON_KEY' \
  --header 'Content-Type: application/json' \
  --data '{"conversation_id":"test","content":"Hello"}'
```

---

## Frontend Component Library

### Navigation Component

Main navigation with responsive design and route highlighting.

```typescript
import { Navigation } from '@/components/Navigation';

// Usage in App.tsx
<Navigation />
```

**Features:**
- Responsive mobile menu
- Active route highlighting
- Smooth animations
- Accessible keyboard navigation

### Error Boundary

Global error handling component.

```typescript
import { ErrorBoundary } from '@/components/ErrorBoundary';

// Usage
<ErrorBoundary>
  <YourComponent />
</ErrorBoundary>
```

### Auth Context

Authentication state management.

```typescript
import { useAuth } from '@/contexts/AuthContext';

function MyComponent() {
  const { user, signIn, signOut, loading } = useAuth();
  
  if (loading) return <div>Loading...</div>;
  
  return user ? (
    <button onClick={signOut}>Sign Out</button>
  ) : (
    <button onClick={() => signIn(email, password)}>Sign In</button>
  );
}
```

### Page Components

All page components follow a consistent structure:

```typescript
// Example: AIChat.tsx
export default function AIChat() {
  return (
    <div className="min-h-screen bg-background-page">
      {/* Hero Section */}
      <section className="relative min-h-hero">
        <div className="container mx-auto px-4 py-xl">
          <h1 className="text-hero font-bold text-neutral-900">
            AI Chat
          </h1>
          <p className="text-body-large text-neutral-700">
            Description
          </p>
        </div>
      </section>
      
      {/* Content Sections */}
      <section className="py-2xl">
        {/* Section content */}
      </section>
    </div>
  );
}
```

### Design Tokens

Available via Tailwind CSS classes:

**Colors:**
```css
bg-primary-500      /* #0066FF */
text-neutral-900    /* #171717 */
border-neutral-200  /* #E5E5E5*/
```

**Typography:**
```css
text-hero          /* 64px */
text-title         /* 48px */
text-subtitle      /* 32px */
text-body-large    /* 20px */
text-body          /* 16px */
```

**Spacing:**
```css
p-xs    /* 8px */
p-sm    /* 16px */
p-md    /* 24px */
p-lg    /* 32px */
p-xl    /* 48px */
p-2xl   /* 64px */
p-3xl   /* 96px */
```

**Rounded Corners:**
```css
rounded-button  /* 12px */
rounded-card    /* 16px */
rounded-modal   /* 24px */
```

### Common Patterns

**Card Component:**

```typescript
<div className="bg-background-surface rounded-card shadow-card p-lg hover:shadow-card-hover transition-shadow duration-standard">
  <h3 className="text-subtitle font-semibold text-neutral-900 mb-md">
    Card Title
  </h3>
  <p className="text-body text-neutral-700">
    Card content
  </p>
</div>
```

**Button Component:**

```typescript
<button className="bg-primary-500 text-white px-lg py-sm rounded-button font-medium hover:bg-primary-600 transition-colors duration-fast">
  Click Me
</button>
```

**Form Input:**

```typescript
<input
  type="text"
  className="w-full px-md py-sm border border-neutral-200 rounded-input focus:outline-none focus:ring-2 focus:ring-primary-500 transition-shadow"
  placeholder="Enter text"
/>
```

---

## Testing Guide

### Manual Testing Checklist

#### Authentication Flow
- [ ] User can sign up with email/password
- [ ] User can sign in
- [ ] User can sign out
- [ ] Protected routes redirect to login
- [ ] User session persists on refresh

#### AI Chat
- [ ] Create new conversation
- [ ] Send messages
- [ ] Receive AI responses
- [ ] View conversation history
- [ ] Delete conversations

#### Support Tickets
- [ ] Create new ticket
- [ ] View ticket list
- [ ] Update ticket status
- [ ] Add comments
- [ ] Filter by priority/status

#### Browser Automation
- [ ] Create workflow
- [ ] Execute workflow
- [ ] View job results
- [ ] Schedule workflow

### Unit Testing (Future Enhancement)

Set up Jest and React Testing Library:

```bash
pnpm add -D @testing-library/react @testing-library/jest-dom vitest jsdom
```

**Example Test:**

```typescript
// __tests__/Navigation.test.tsx
import { render, screen } from '@testing-library/react';
import { Navigation } from '@/components/Navigation';

describe('Navigation', () => {
  it('renders navigation links', () => {
    render(<Navigation />);
    expect(screen.getByText('AI Chat')).toBeInTheDocument();
    expect(screen.getByText('Browser Automation')).toBeInTheDocument();
  });
});
```

### Integration Testing

Test Edge Functions locally:

```bash
# Start local Supabase
supabase start

# Run function tests
curl -X POST http://localhost:54321/functions/v1/ai-chat-message \
  -H "Authorization: Bearer ${ANON_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"conversation_id":"test","content":"Test message"}'
```

### E2E Testing with Playwright (Future Enhancement)

```typescript
// e2e/auth.spec.ts
import { test, expect } from '@playwright/test';

test('user can sign in', async ({ page }) => {
  await page.goto('http://localhost:5173');
  await page.click('text=Sign In');
  await page.fill('input[type="email"]', 'test@example.com');
  await page.fill('input[type="password"]', 'password123');
  await page.click('button:has-text("Sign In")');
  await expect(page).toHaveURL('/dashboard');
});
```

---

## Deployment Guide

### Production Build

```bash
# Create optimized production build
pnpm run build

# The build output will be in ./dist directory
```

### Deploy to Vercel

```bash
# Install Vercel CLI
npm install -g vercel

# Deploy
vercel

# Deploy to production
vercel --prod
```

**vercel.json:**

```json
{
  "buildCommand": "pnpm run build",
  "outputDirectory": "dist",
  "devCommand": "pnpm run dev",
  "installCommand": "pnpm install",
  "framework": "vite",
  "env": {
    "VITE_SUPABASE_URL": "@supabase_url",
    "VITE_SUPABASE_ANON_KEY": "@supabase_anon_key"
  }
}
```

### Deploy to Netlify

**netlify.toml:**

```toml
[build]
  command = "pnpm run build"
  publish = "dist"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[build.environment]
  NODE_VERSION = "18"
```

### Deploy Edge Functions to Supabase

```bash
# Login
supabase login

# Link project
supabase link --project-ref your-project-ref

# Deploy all functions
supabase functions deploy

# Set environment secrets for functions
supabase secrets set OPENAI_API_KEY=your_key
supabase secrets set ANTHROPIC_API_KEY=your_key
```

### Environment Variables Checklist

**Frontend (.env):**
- `VITE_SUPABASE_URL`
- `VITE_SUPABASE_ANON_KEY`

**Edge Functions (Supabase Secrets):**
- `SUPABASE_URL` (automatically provided)
- `SUPABASE_SERVICE_ROLE_KEY` (automatically provided)
- `OPENAI_API_KEY`
- `ANTHROPIC_API_KEY`
- `GOOGLE_API_KEY`

### Performance Optimization

**Build Optimizations:**

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'react-vendor': ['react', 'react-dom', 'react-router-dom'],
          'supabase-vendor': ['@supabase/supabase-js'],
          'ui-vendor': ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu'],
        }
      }
    },
    chunkSizeWarningLimit: 1000,
  }
});
```

**Image Optimization:**
- Use WebP format
- Implement lazy loading
- Add proper width/height attributes

**Code Splitting:**
- Use React.lazy() for route-based splitting
- Implement dynamic imports for heavy components

---

## Troubleshooting

### Common Issues

#### 1. "Supabase client not configured" Error

**Problem:** Environment variables not loaded

**Solution:**
```bash
# Ensure .env file exists
cp .env.example .env

# Add your Supabase credentials
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key

# Restart dev server
pnpm run dev
```

#### 2. Authentication Not Working

**Problem:** RLS policies blocking access

**Solution:**
```sql
-- Check if RLS is enabled
SELECT tablename, rowsecurity FROM pg_tables 
WHERE schemaname = 'public';

-- Verify policies exist
SELECT * FROM pg_policies WHERE tablename = 'profiles';

-- Re-apply RLS policies if needed
\i rls-policies.sql
```

#### 3. Edge Function Timeout

**Problem:** Function exceeds execution time limit

**Solution:**
- Optimize database queries
- Use connection pooling
- Implement pagination for large datasets
- Add proper indexes

#### 4. Build Fails with TypeScript Errors

**Problem:** Type mismatches or missing types

**Solution:**
```bash
# Clear TypeScript cache
rm -rf node_modules/.vite

# Reinstall dependencies
pnpm install

# Check for type errors
pnpm run build
```

#### 5. CORS Errors in Development

**Problem:** Cross-origin request blocked

**Solution:**
```typescript
// Ensure Edge Functions include CORS headers
const corsHeaders = {
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Headers': 'authorization, x-client-info, apikey, content-type',
};

if (req.method === 'OPTIONS') {
  return new Response(null, { status: 200, headers: corsHeaders });
}
```

#### 6. Real-time Subscriptions Not Working

**Problem:** Changes not reflected in real-time

**Solution:**
```sql
-- Enable real-time for tables
ALTER PUBLICATION supabase_realtime ADD TABLE messages;
ALTER PUBLICATION supabase_realtime ADD TABLE support_tickets;

-- Verify in Supabase Dashboard > Database > Publications
```

#### 7. Database Connection Pool Exhausted

**Problem:** Too many concurrent connections

**Solution:**
- Implement connection pooling
- Use Supabase's built-in pooler
- Close unused connections
- Optimize query patterns

### Debug Mode

Enable verbose logging:

```typescript
// In development
if (import.meta.env.DEV) {
  console.log('Supabase client initialized:', supabase);
}

// For API debugging
const { data, error } = await supabase.from('table').select();
console.log('Query result:', { data, error });
```

### Performance Profiling

```typescript
// Measure component render time
import { Profiler } from 'react';

function onRenderCallback(
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number
) {
  console.log(`${id} (${phase}): ${actualDuration}ms`);
}

<Profiler id="MyComponent" onRender={onRenderCallback}>
  <MyComponent />
</Profiler>
```

### Database Query Optimization

```sql
-- Explain query performance
EXPLAIN ANALYZE 
SELECT * FROM conversations 
WHERE user_id = 'uuid' 
ORDER BY created_at DESC;

-- Check for missing indexes
SELECT tablename, indexname 
FROM pg_indexes 
WHERE schemaname = 'public';

-- Monitor slow queries
SELECT query, mean_exec_time 
FROM pg_stat_statements 
ORDER BY mean_exec_time DESC 
LIMIT 10;
```

### Support Resources

- **Supabase Documentation:** https://supabase.com/docs
- **React Documentation:** https://react.dev
- **Tailwind CSS:** https://tailwindcss.com/docs
- **TypeScript Handbook:** https://www.typescriptlang.org/docs
- **Vite Guide:** https://vitejs.dev/guide

### Reporting Issues

When reporting issues, include:
1. Error message and stack trace
2. Steps to reproduce
3. Browser/Node version
4. Environment (development/production)
5. Relevant code snippets
6. Network/console logs

---

## Additional Resources

### Code Examples Repository

Find more examples in the `/examples` directory:
- Authentication flows
- Database queries
- Edge function templates
- UI component patterns

### API Client Libraries

```typescript
// Example: Custom API client wrapper
import { supabase } from '@/lib/supabase';

export class ConversationAPI {
  static async getAll(userId: string) {
    return supabase
      .from('conversations')
      .select('*')
      .eq('user_id', userId)
      .order('created_at', { ascending: false });
  }

  static async create(userId: string, data: ConversationData) {
    return supabase
      .from('conversations')
      .insert({ user_id: userId, ...data })
      .select()
      .single();
  }

  static async delete(id: string) {
    return supabase
      .from('conversations')
      .delete()
      .eq('id', id);
  }
}
```

### Development Best Practices

1. **Always use TypeScript types** for type safety
2. **Implement error boundaries** for graceful error handling
3. **Use React hooks properly** (no conditional hooks)
4. **Follow accessibility guidelines** (ARIA labels, keyboard navigation)
5. **Optimize images and assets** before deployment
6. **Write meaningful commit messages**
7. **Test on multiple browsers and devices**
8. **Keep dependencies updated**
9. **Document complex logic**
10. **Use environment variables for sensitive data**

### Security Checklist

- [ ] Never commit `.env` files
- [ ] Use RLS policies for all tables
- [ ] Validate user input on frontend and backend
- [ ] Sanitize database queries
- [ ] Implement rate limiting on Edge Functions
- [ ] Use HTTPS in production
- [ ] Enable 2FA for Supabase account
- [ ] Regular security audits
- [ ] Keep dependencies updated
- [ ] Monitor for suspicious activity

---

## Changelog

### Version 1.0.0 (Current)
- Initial release
- 8 marketing pages
- Database schema with 9 tables
- 3 edge functions
- Authentication system
- Modern Minimalism Premium design
- Full TypeScript support
- Responsive design

### Planned Features
- AI provider integrations (OpenAI, Claude, Gemini)
- Real-time collaboration
- Advanced analytics dashboard
- Mobile application
- API rate limiting
- Webhook system
- Batch operations
- Export/import functionality

---

## Contributing

### Development Workflow

1. Create a feature branch
2. Make changes
3. Test thoroughly
4. Submit pull request
5. Code review
6. Merge to main

### Code Style

- Use TypeScript for all new files
- Follow ESLint configuration
- Use Prettier for formatting
- Write meaningful variable names
- Add comments for complex logic
- Keep functions small and focused

### Commit Message Format

```
type(scope): subject

body

footer
```

**Types:** feat, fix, docs, style, refactor, test, chore

**Example:**
```
feat(ai-chat): add streaming response support

Implement server-sent events for streaming AI responses
to improve user experience and reduce perceived latency.

Closes #123
```

---

## License

This project is proprietary and confidential.

## Contact

For questions or support, contact the development team.

---

**Last Updated:** 2025-10-29  
**Version:** 1.0.0  
**Maintained by:** Development Team
