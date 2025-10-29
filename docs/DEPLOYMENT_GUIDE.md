# AI Super Assistant Platform - Production Deployment Guide

**Version**: 1.0  
**Last Updated**: October 29, 2025  
**Target Environment**: Production

---

## Table of Contents

1. [Infrastructure Requirements](#1-infrastructure-requirements)
2. [Supabase Setup](#2-supabase-setup)
3. [Environment Configuration](#3-environment-configuration)
4. [Database Migration Steps](#4-database-migration-steps)
5. [Edge Function Deployment](#5-edge-function-deployment)
6. [Frontend Deployment](#6-frontend-deployment)
7. [Monitoring Setup](#7-monitoring-setup)
8. [Backup Procedures](#8-backup-procedures)
9. [Disaster Recovery](#9-disaster-recovery)
10. [Scaling Guidelines](#10-scaling-guidelines)

---

## 1. Infrastructure Requirements

### 1.1 Minimum System Requirements

#### Production Server (Frontend)
- **CPU**: 2+ cores
- **RAM**: 4GB minimum, 8GB recommended
- **Storage**: 20GB SSD
- **Network**: 100 Mbps bandwidth
- **OS**: Linux (Ubuntu 22.04 LTS or similar)

#### Database (Supabase)
- **Plan**: Supabase Pro tier recommended for production
- **Storage**: 8GB+ database storage
- **Connections**: Up to 500 concurrent connections
- **Region**: Choose closest to your primary user base

#### Edge Functions
- **Plan**: Included in Supabase Pro
- **Memory**: 256MB per function (default)
- **Timeout**: 300 seconds per invocation
- **Concurrent Executions**: 100+ (scales automatically)

### 1.2 Required Services

- **Supabase Account**: Pro tier ($25/month minimum)
- **Domain Name**: Custom domain for production
- **SSL Certificate**: Provided by Supabase and hosting platform
- **CDN**: CloudFlare or similar (optional but recommended)

### 1.3 Required Tools

```bash
# Node.js and pnpm
node --version  # v18+ required
pnpm --version  # v8+ required

# Supabase CLI (optional but recommended)
npm install -g supabase

# PostgreSQL Client (for database management)
sudo apt-get install postgresql-client

# Git
git --version
```

### 1.4 Third-Party API Keys (Optional)

For full AI functionality:
- **OpenAI API Key** (GPT-4, GPT-3.5 models)
- **Anthropic API Key** (Claude models)
- **Google AI API Key** (Gemini models)

For browser automation (advanced):
- **Playwright Cloud** or self-hosted automation service

---

## 2. Supabase Setup

### 2.1 Create New Supabase Project

1. **Sign up/Login to Supabase**
   ```
   https://supabase.com/dashboard
   ```

2. **Create New Project**
   - Click "New Project"
   - Organization: Select or create organization
   - Name: `ai-super-assistant-prod`
   - Database Password: Generate strong password (save securely)
   - Region: Select closest to users (e.g., `us-east-1`, `eu-west-1`)
   - Pricing Plan: **Pro** (recommended for production)

3. **Wait for Project Provisioning** (2-3 minutes)

### 2.2 Retrieve Supabase Credentials

Navigate to: **Project Settings → API**

Required credentials:
```bash
# Project URL
SUPABASE_URL=https://xxxxxxxxxxxxx.supabase.co

# Anon/Public Key (safe for frontend)
SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Service Role Key (KEEP SECRET - backend only)
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Database URL (for direct connections)
DATABASE_URL=postgresql://postgres:[PASSWORD]@db.xxxxxxxxxxxxx.supabase.co:5432/postgres
```

⚠️ **Security Note**: Never commit `SUPABASE_SERVICE_ROLE_KEY` to version control!

### 2.3 Configure Authentication

1. **Navigate to**: Authentication → Providers

2. **Enable Email Authentication**
   - Toggle: Enable Email provider
   - Confirm email: Enable
   - Secure email change: Enable
   - Email templates: Customize (optional)

3. **Configure Auth Settings**
   ```
   Authentication → Settings
   - Site URL: https://yourdomain.com
   - Redirect URLs: https://yourdomain.com/*
   - JWT expiry: 3600 seconds (default)
   ```

4. **Optional: Enable OAuth Providers**
   - Google OAuth
   - GitHub OAuth
   - Microsoft OAuth

### 2.4 Configure Storage Buckets

Create storage buckets for file uploads:

```sql
-- Via SQL Editor or use Supabase Dashboard UI
INSERT INTO storage.buckets (id, name, public) 
VALUES 
  ('avatars', 'avatars', true),
  ('automation-results', 'automation-results', false),
  ('ticket-attachments', 'ticket-attachments', false);
```

Or via Dashboard:
```
Storage → New Bucket
- Name: avatars
- Public: Yes
- File size limit: 5MB
- Allowed MIME types: image/*
```

---

## 3. Environment Configuration

### 3.1 Frontend Environment Variables

Create `/workspace/ai-super-assistant/.env.production`:

```bash
# Supabase Configuration
VITE_SUPABASE_URL=https://xxxxxxxxxxxxx.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Application Configuration
VITE_APP_NAME=AI Super Assistant
VITE_APP_VERSION=1.0.0
VITE_APP_ENV=production

# Feature Flags (optional)
VITE_ENABLE_AI_CHAT=true
VITE_ENABLE_AUTOMATION=true
VITE_ENABLE_IT_SUPPORT=true
VITE_ENABLE_ANALYTICS=true
```

### 3.2 Edge Function Environment Variables

Configure via: **Edge Functions → Function Settings → Environment Variables**

```bash
# Supabase Access
SUPABASE_URL=https://xxxxxxxxxxxxx.supabase.co
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# AI API Keys (Optional)
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_AI_API_KEY=...

# Email Service (Optional)
SMTP_HOST=smtp.sendgrid.net
SMTP_PORT=587
SMTP_USER=apikey
SMTP_PASS=...

# External Services
AUTOMATION_SERVICE_URL=https://automation-service.example.com
WEBHOOK_SECRET=...
```

### 3.3 Validate Environment Setup

```bash
# Check frontend environment
cd /workspace/ai-super-assistant
cat .env.production | grep VITE_

# Verify no secrets in frontend .env
grep -i "service_role" .env.production && echo "⚠️ WARNING: Service role key in frontend!"
```

---

## 4. Database Migration Steps

### 4.1 Pre-Migration Checklist

- [ ] Supabase project created and accessible
- [ ] Database credentials retrieved
- [ ] PostgreSQL client installed
- [ ] Database schema files reviewed

### 4.2 Apply Database Schema

**Method 1: Via Supabase Dashboard (Recommended)**

1. Navigate to: **SQL Editor → New Query**

2. Copy contents from `/workspace/database-schema.sql`

3. Paste and click **Run**

4. Verify success:
   ```sql
   SELECT table_name 
   FROM information_schema.tables 
   WHERE table_schema = 'public' 
   ORDER BY table_name;
   ```

   Expected tables (19 total):
   - profiles
   - conversations
   - messages
   - automation_workflows
   - automation_jobs
   - support_tickets
   - ticket_comments
   - assets
   - monitoring_metrics
   - clients
   - projects
   - project_team
   - usage_metrics
   - invoices
   - workflows
   - webhooks
   - integrations
   - analytics_events
   - ai_model_configs

**Method 2: Via psql Command Line**

```bash
# Set connection details
export PGHOST=db.xxxxxxxxxxxxx.supabase.co
export PGPORT=5432
export PGDATABASE=postgres
export PGUSER=postgres
export PGPASSWORD='your-database-password'

# Apply schema
psql < /workspace/database-schema.sql

# Verify
psql -c "\dt"
```

### 4.3 Apply Row Level Security Policies

1. Navigate to: **SQL Editor → New Query**

2. Copy contents from `/workspace/rls-policies.sql`

3. Paste and click **Run**

4. Verify RLS is enabled:
   ```sql
   SELECT tablename, rowsecurity 
   FROM pg_tables 
   WHERE schemaname = 'public';
   ```

   All tables should have `rowsecurity = true`

### 4.4 Create Database Indexes (Performance Optimization)

```sql
-- Create indexes for frequently queried columns
CREATE INDEX IF NOT EXISTS idx_messages_conversation_id ON messages(conversation_id);
CREATE INDEX IF NOT EXISTS idx_messages_created_at ON messages(created_at DESC);
CREATE INDEX IF NOT EXISTS idx_conversations_user_id ON conversations(user_id);
CREATE INDEX IF NOT EXISTS idx_automation_jobs_workflow_id ON automation_jobs(workflow_id);
CREATE INDEX IF NOT EXISTS idx_automation_jobs_status ON automation_jobs(status);
CREATE INDEX IF NOT EXISTS idx_support_tickets_user_id ON support_tickets(user_id);
CREATE INDEX IF NOT EXISTS idx_support_tickets_status ON support_tickets(status);
CREATE INDEX IF NOT EXISTS idx_analytics_events_user_id ON analytics_events(user_id);
CREATE INDEX IF NOT EXISTS idx_analytics_events_created_at ON analytics_events(created_at DESC);
```

### 4.5 Seed Initial Data (Optional)

```sql
-- Create default AI model configurations
INSERT INTO ai_model_configs (user_id, provider, model_name, temperature, max_tokens, is_default)
VALUES 
  (auth.uid(), 'openai', 'gpt-4', 0.7, 2000, true),
  (auth.uid(), 'claude', 'claude-3-opus', 0.7, 2000, false),
  (auth.uid(), 'gemini', 'gemini-pro', 0.7, 2000, false);
```

### 4.6 Verify Migration Success

```sql
-- Check table counts
SELECT 
  schemaname,
  COUNT(*) as table_count
FROM pg_tables 
WHERE schemaname = 'public'
GROUP BY schemaname;

-- Check RLS policies
SELECT 
  schemaname,
  tablename,
  COUNT(*) as policy_count
FROM pg_policies 
WHERE schemaname = 'public'
GROUP BY schemaname, tablename
ORDER BY tablename;
```

---

## 5. Edge Function Deployment

### 5.1 Available Edge Functions

1. **ai-chat-message**: Handles AI conversation processing
2. **create-support-ticket**: Creates IT support tickets
3. **execute-automation**: Executes browser automation workflows

### 5.2 Deploy via Supabase CLI (Recommended)

```bash
# Install Supabase CLI
npm install -g supabase

# Login to Supabase
supabase login

# Link to your project
cd /workspace
supabase link --project-ref xxxxxxxxxxxxx

# Deploy all functions
supabase functions deploy ai-chat-message
supabase functions deploy create-support-ticket
supabase functions deploy execute-automation

# Verify deployment
supabase functions list
```

### 5.3 Deploy via Supabase Dashboard (Manual)

For each function:

1. Navigate to: **Edge Functions → Create Function**

2. Function details:
   - Name: `ai-chat-message` (or function name)
   - Import: Choose "Create from scratch"

3. Copy code from: `/workspace/supabase/functions/ai-chat-message/index.ts`

4. Click **Deploy**

5. Repeat for all three functions

### 5.4 Test Edge Functions

**Test ai-chat-message:**
```bash
curl -X POST \
  https://xxxxxxxxxxxxx.supabase.co/functions/v1/ai-chat-message \
  -H "Authorization: Bearer YOUR_ANON_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "conversation_id": "test-conv-id",
    "content": "Hello, world!",
    "model_provider": "openai",
    "model_name": "gpt-4"
  }'
```

**Test create-support-ticket:**
```bash
curl -X POST \
  https://xxxxxxxxxxxxx.supabase.co/functions/v1/create-support-ticket \
  -H "Authorization: Bearer YOUR_ANON_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Test Ticket",
    "description": "Testing ticket creation",
    "priority": "medium",
    "category": "incident"
  }'
```

**Test execute-automation:**
```bash
curl -X POST \
  https://xxxxxxxxxxxxx.supabase.co/functions/v1/execute-automation \
  -H "Authorization: Bearer YOUR_ANON_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "workflow_id": "test-workflow-id"
  }'
```

### 5.5 Configure Function Secrets

Via Dashboard: **Edge Functions → Function Settings → Secrets**

```bash
# Add each secret
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_AI_API_KEY=...
```

Or via CLI:
```bash
supabase secrets set OPENAI_API_KEY=sk-...
supabase secrets set ANTHROPIC_API_KEY=sk-ant-...
supabase secrets set GOOGLE_AI_API_KEY=...
```

### 5.6 Enable Function Logs

1. Navigate to: **Edge Functions → [Function Name] → Logs**
2. Enable logging: **On**
3. Retention: 7 days (configurable)

---

## 6. Frontend Deployment

### 6.1 Build Production Bundle

```bash
# Navigate to frontend directory
cd /workspace/ai-super-assistant

# Install dependencies
pnpm install --frozen-lockfile

# Build for production
pnpm run build

# Verify build
ls -lh dist/
```

Expected output:
```
dist/
├── assets/
│   ├── index-[hash].js      (~606KB)
│   └── index-[hash].css     (~18KB)
├── index.html
└── vite.svg
```

### 6.2 Optimize Build (Optional)

```bash
# Analyze bundle size
pnpm add -D rollup-plugin-visualizer

# Add to vite.config.ts
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    react(),
    visualizer({ open: true })
  ]
});

# Build and analyze
pnpm run build
```

### 6.3 Deploy to Static Hosting

**Option A: Vercel**
```bash
# Install Vercel CLI
npm install -g vercel

# Deploy
cd /workspace/ai-super-assistant
vercel --prod

# Set environment variables in Vercel dashboard
```

**Option B: Netlify**
```bash
# Install Netlify CLI
npm install -g netlify-cli

# Deploy
cd /workspace/ai-super-assistant
netlify deploy --prod --dir=dist

# Configure environment variables
netlify env:set VITE_SUPABASE_URL "https://xxxxxxxxxxxxx.supabase.co"
netlify env:set VITE_SUPABASE_ANON_KEY "eyJhbG..."
```

**Option C: Custom Server (Nginx)**
```bash
# Copy build to web server
scp -r dist/* user@server:/var/www/ai-assistant/

# Nginx configuration
server {
    listen 80;
    server_name yourdomain.com;
    root /var/www/ai-assistant;
    index index.html;

    # SPA fallback
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
}

# Enable and reload Nginx
sudo ln -s /etc/nginx/sites-available/ai-assistant /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

**Option D: MiniMax Space (Current)**
```bash
# Already deployed at: https://eyajoj1llnnr.space.minimax.io
# To redeploy with updated credentials:
cd /workspace/ai-super-assistant
pnpm run build
# Use MiniMax deploy tool
```

### 6.4 Configure Custom Domain

1. **Add CNAME record** in your DNS provider:
   ```
   Type: CNAME
   Name: www
   Value: your-deployment-platform.com
   ```

2. **Update Supabase allowed domains**:
   - Navigate to: **Authentication → URL Configuration**
   - Add: `https://yourdomain.com`

3. **Enable SSL/TLS** (automatic with most platforms)

### 6.5 Post-Deployment Verification

```bash
# Test homepage
curl -I https://yourdomain.com

# Test API connectivity (from browser console)
const { data, error } = await supabase.auth.getSession();
console.log('Session:', data);

# Test routing
curl -I https://yourdomain.com/ai-chat
curl -I https://yourdomain.com/pricing
```

---

## 7. Monitoring Setup

### 7.1 Supabase Built-in Monitoring

**Database Monitoring**
- Navigate to: **Database → Logs**
- Monitor: Query performance, slow queries, connection pool

**Edge Function Monitoring**
- Navigate to: **Edge Functions → [Function] → Logs**
- Monitor: Invocations, errors, execution time

**API Monitoring**
- Navigate to: **API → Logs**
- Monitor: Request volume, error rates, response times

### 7.2 Set Up Alerts

1. Navigate to: **Project Settings → Integrations**

2. **Configure Webhook Alerts**:
   ```json
   {
     "webhook_url": "https://your-monitoring-service.com/webhook",
     "events": [
       "database.cpu_high",
       "database.disk_full",
       "function.error_rate_high",
       "auth.suspicious_activity"
     ]
   }
   ```

3. **Email Alerts**:
   - Add team email addresses
   - Configure alert thresholds

### 7.3 Application Performance Monitoring (APM)

**Option 1: Sentry**
```bash
# Install Sentry
pnpm add @sentry/react @sentry/tracing

# Configure in src/main.tsx
import * as Sentry from "@sentry/react";

Sentry.init({
  dsn: "YOUR_SENTRY_DSN",
  environment: "production",
  tracesSampleRate: 1.0,
});
```

**Option 2: LogRocket**
```bash
# Install LogRocket
pnpm add logrocket

# Configure
import LogRocket from 'logrocket';
LogRocket.init('your-app-id');
```

### 7.4 Custom Logging

Create logging utility (`src/utils/logger.ts`):
```typescript
export const logger = {
  info: (message: string, data?: any) => {
    if (import.meta.env.PROD) {
      // Send to logging service
      console.log(message, data);
    }
  },
  error: (message: string, error?: Error) => {
    if (import.meta.env.PROD) {
      // Send to error tracking service
      console.error(message, error);
    }
  }
};
```

### 7.5 Health Check Endpoint

Create edge function `health-check/index.ts`:
```typescript
Deno.serve(async (req) => {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    version: '1.0.0'
  };
  
  return new Response(JSON.stringify(health), {
    headers: { 'Content-Type': 'application/json' }
  });
});
```

Monitor with:
```bash
curl https://xxxxxxxxxxxxx.supabase.co/functions/v1/health-check
```

### 7.6 Performance Metrics Dashboard

```sql
-- Create analytics view
CREATE OR REPLACE VIEW analytics_summary AS
SELECT 
  DATE_TRUNC('hour', created_at) as hour,
  event_type,
  COUNT(*) as event_count,
  AVG(EXTRACT(EPOCH FROM (updated_at - created_at))) as avg_duration
FROM analytics_events
WHERE created_at > NOW() - INTERVAL '24 hours'
GROUP BY hour, event_type
ORDER BY hour DESC;

-- Query for dashboard
SELECT * FROM analytics_summary;
```

---

## 8. Backup Procedures

### 8.1 Automated Database Backups

Supabase Pro includes automated daily backups.

**Verify backup schedule**:
1. Navigate to: **Database → Backups**
2. Verify: Daily backups enabled
3. Retention: 7 days (Pro plan)

### 8.2 Manual Database Backup

```bash
# Export full database
pg_dump -h db.xxxxxxxxxxxxx.supabase.co \
  -U postgres \
  -d postgres \
  -F c \
  -f backup_$(date +%Y%m%d_%H%M%S).dump

# Export specific tables
pg_dump -h db.xxxxxxxxxxxxx.supabase.co \
  -U postgres \
  -d postgres \
  -t conversations \
  -t messages \
  -F c \
  -f conversations_backup_$(date +%Y%m%d).dump

# Export as SQL
pg_dump -h db.xxxxxxxxxxxxx.supabase.co \
  -U postgres \
  -d postgres \
  --clean --if-exists \
  -f backup_$(date +%Y%m%d_%H%M%S).sql
```

### 8.3 Backup Storage Buckets

```bash
# Install Supabase CLI storage tools
supabase storage download --all --bucket avatars --output ./backups/avatars/

# Or use S3-compatible tools
aws s3 sync s3://your-supabase-storage/avatars ./backups/avatars/ \
  --endpoint-url https://xxxxxxxxxxxxx.supabase.co/storage/v1/s3
```

### 8.4 Backup Edge Functions

```bash
# Clone function code
mkdir -p backups/functions
cp -r /workspace/supabase/functions/* backups/functions/

# Export function configurations
supabase functions list > backups/functions-list.txt
```

### 8.5 Automated Backup Script

Create `scripts/backup.sh`:
```bash
#!/bin/bash

BACKUP_DIR="/backups/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

# Database backup
pg_dump -h $PGHOST -U $PGUSER -d $PGDATABASE \
  -F c -f "$BACKUP_DIR/database.dump"

# Storage backup (if configured)
supabase storage download --all --output "$BACKUP_DIR/storage/"

# Compress
tar -czf "$BACKUP_DIR.tar.gz" "$BACKUP_DIR"

# Upload to S3 (optional)
aws s3 cp "$BACKUP_DIR.tar.gz" s3://your-backup-bucket/

# Cleanup old backups (keep 30 days)
find /backups -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR.tar.gz"
```

Schedule with cron:
```bash
# Edit crontab
crontab -e

# Add daily backup at 2 AM
0 2 * * * /path/to/scripts/backup.sh >> /var/log/backup.log 2>&1
```

### 8.6 Verify Backup Integrity

```bash
# Test database restore (dry run)
pg_restore --list backup_20251029.dump

# Verify backup size
ls -lh backup_20251029.dump

# Test restore to temp database
createdb test_restore
pg_restore -d test_restore backup_20251029.dump
psql -d test_restore -c "\dt"
dropdb test_restore
```

---

## 9. Disaster Recovery

### 9.1 Recovery Time Objective (RTO)

**Target RTO**: 4 hours (maximum downtime)

**Recovery Priorities**:
1. Database restoration (1 hour)
2. Edge functions deployment (30 minutes)
3. Frontend deployment (30 minutes)
4. Verification and testing (2 hours)

### 9.2 Recovery Point Objective (RPO)

**Target RPO**: 24 hours (maximum data loss)

With daily backups, maximum data loss is 24 hours.

For critical applications, consider:
- Point-in-Time Recovery (PITR) - available on Supabase Pro
- Enables recovery to any point within retention period

### 9.3 Database Recovery Procedures

**Scenario 1: Accidental Data Deletion**

```sql
-- If within transaction, rollback
ROLLBACK;

-- If committed, restore from backup
-- 1. Restore to temporary database
createdb recovery_temp
pg_restore -d recovery_temp backup_latest.dump

-- 2. Export specific data
pg_dump -d recovery_temp -t conversations --data-only > conversations_data.sql

-- 3. Import to production
psql -d postgres < conversations_data.sql

-- 4. Verify
SELECT COUNT(*) FROM conversations;

-- 5. Cleanup
dropdb recovery_temp
```

**Scenario 2: Database Corruption**

```bash
# 1. Create new Supabase project
# 2. Restore latest backup

pg_restore -h db.new-project.supabase.co \
  -U postgres \
  -d postgres \
  -c \
  backup_latest.dump

# 3. Update application environment variables
# 4. Verify data integrity
psql -h db.new-project.supabase.co -U postgres -d postgres -c "\dt"

# 5. Switch DNS/update frontend config
# 6. Monitor for issues
```

**Scenario 3: Complete Project Loss**

```bash
# Full restoration procedure

# 1. Create new Supabase project (15 minutes)
#    - Same region as original
#    - Same plan tier

# 2. Restore database (30 minutes)
pg_restore -h db.new-project.supabase.co \
  -U postgres \
  -d postgres \
  --clean --if-exists \
  backup_latest.dump

# 3. Apply RLS policies
psql -h db.new-project.supabase.co \
  -U postgres \
  -d postgres \
  < rls-policies.sql

# 4. Redeploy edge functions (20 minutes)
supabase link --project-ref new-project-ref
supabase functions deploy ai-chat-message
supabase functions deploy create-support-ticket
supabase functions deploy execute-automation

# 5. Restore storage buckets (if backed up)
supabase storage restore --bucket avatars --from ./backups/storage/avatars/

# 6. Update frontend environment (10 minutes)
#    - Update VITE_SUPABASE_URL
#    - Update VITE_SUPABASE_ANON_KEY
#    - Rebuild and redeploy

# 7. Verify all systems (30 minutes)
#    - Test authentication
#    - Test database queries
#    - Test edge functions
#    - Test file uploads
```

### 9.4 Edge Function Recovery

```bash
# Redeploy all functions from source
cd /workspace
supabase functions deploy ai-chat-message
supabase functions deploy create-support-ticket
supabase functions deploy execute-automation

# Restore function secrets
supabase secrets set OPENAI_API_KEY=sk-...
supabase secrets set ANTHROPIC_API_KEY=sk-ant-...

# Verify
supabase functions list
```

### 9.5 Frontend Recovery

```bash
# Rebuild from source
cd /workspace/ai-super-assistant
git pull origin main
pnpm install --frozen-lockfile
pnpm run build

# Redeploy
# (Use appropriate deployment method from Section 6)

# Verify
curl -I https://yourdomain.com
```

### 9.6 Disaster Recovery Checklist

```markdown
## Pre-Disaster Preparation
- [ ] Recent backups available (< 24 hours old)
- [ ] Backup integrity verified
- [ ] Recovery procedures documented
- [ ] Emergency contact list updated
- [ ] Service credentials stored securely
- [ ] DNS configuration documented
- [ ] Third-party API keys accessible

## During Disaster
- [ ] Assess scope of issue
- [ ] Notify stakeholders
- [ ] Document incident timeline
- [ ] Activate recovery team
- [ ] Begin recovery procedures
- [ ] Monitor recovery progress

## Post-Disaster
- [ ] Verify all services restored
- [ ] Test critical functionality
- [ ] Review data integrity
- [ ] Update documentation
- [ ] Conduct post-mortem analysis
- [ ] Implement preventive measures
```

### 9.7 Communication Plan

**Stakeholder Notification Templates**:

```
INCIDENT ALERT - [SEVERITY]
Time: [TIMESTAMP]
Issue: [DESCRIPTION]
Impact: [USER IMPACT]
ETA: [ESTIMATED RECOVERY TIME]
Status Page: [URL]
```

```
RECOVERY UPDATE
Time: [TIMESTAMP]
Progress: [PERCENTAGE]
Completed: [STEPS COMPLETED]
Remaining: [STEPS REMAINING]
Next Update: [TIME]
```

```
INCIDENT RESOLVED
Time: [TIMESTAMP]
Duration: [TOTAL DOWNTIME]
Root Cause: [BRIEF DESCRIPTION]
Actions Taken: [SUMMARY]
Post-Mortem: [WHEN AVAILABLE]
```

---

## 10. Scaling Guidelines

### 10.1 Horizontal Scaling

**Database Scaling** (Supabase handles automatically)
- Upgrade to higher tier for more resources
- Consider read replicas for read-heavy workloads

**Edge Functions Scaling**
- Automatic scaling based on load
- No configuration needed
- Monitor invocation metrics

**Frontend Scaling**
- Use CDN for global distribution
- Enable caching for static assets
- Consider multi-region deployment

### 10.2 Vertical Scaling Triggers

**Database Upgrade Triggers**:
- CPU utilization > 80% sustained
- Memory utilization > 90%
- Connection pool exhausted
- Query response time > 1 second

**Supabase Plan Upgrades**:
```
Free → Pro: Production usage begins
Pro → Team: > 100K active users
Team → Enterprise: > 1M active users
```

### 10.3 Performance Optimization

**Database Optimization**:
```sql
-- Add missing indexes
CREATE INDEX CONCURRENTLY idx_name ON table(column);

-- Analyze query performance
EXPLAIN ANALYZE SELECT * FROM conversations WHERE user_id = 'xxx';

-- Update table statistics
ANALYZE conversations;

-- Vacuum to reclaim space
VACUUM ANALYZE;
```

**Frontend Optimization**:
```typescript
// Code splitting
const AIChat = lazy(() => import('./pages/AIChat'));
const BrowserAutomation = lazy(() => import('./pages/BrowserAutomation'));

// Memoization
const MemoizedComponent = React.memo(ExpensiveComponent);

// Virtual scrolling for long lists
import { FixedSizeList } from 'react-window';
```

**Edge Function Optimization**:
```typescript
// Connection pooling
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  Deno.env.get('SUPABASE_URL')!,
  Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!,
  {
    db: { 
      pool: { 
        max: 10, // Limit connections
        idleTimeoutMillis: 30000 
      } 
    }
  }
);

// Caching responses
const cache = new Map();
const getCached = (key: string) => {
  const cached = cache.get(key);
  if (cached && Date.now() - cached.timestamp < 300000) {
    return cached.data;
  }
  return null;
};
```

### 10.4 Caching Strategy

**Browser Caching** (Nginx):
```nginx
location /assets/ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}

location /api/ {
    expires -1;
    add_header Cache-Control "no-store, no-cache, must-revalidate";
}
```

**Application-Level Caching**:
```typescript
// Use React Query for server state
import { useQuery } from '@tanstack/react-query';

const { data } = useQuery({
  queryKey: ['conversations'],
  queryFn: fetchConversations,
  staleTime: 5 * 60 * 1000, // 5 minutes
});
```

**Database Query Caching**:
```sql
-- Materialized views for expensive queries
CREATE MATERIALIZED VIEW user_stats AS
SELECT 
  user_id,
  COUNT(*) as total_conversations,
  SUM(token_count) as total_tokens
FROM conversations c
JOIN messages m ON c.id = m.conversation_id
GROUP BY user_id;

-- Refresh periodically
REFRESH MATERIALIZED VIEW user_stats;
```

### 10.5 Load Testing

**Setup Load Testing**:
```bash
# Install k6
brew install k6  # macOS
# or
sudo apt install k6  # Linux

# Create load test script (load-test.js)
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '2m', target: 100 }, // Ramp up to 100 users
    { duration: '5m', target: 100 }, // Stay at 100 users
    { duration: '2m', target: 0 },   // Ramp down
  ],
};

export default function () {
  const res = http.get('https://yourdomain.com');
  check(res, { 'status is 200': (r) => r.status === 200 });
  sleep(1);
}

# Run test
k6 run load-test.js
```

**Monitor During Load Test**:
- Database CPU/Memory
- Edge function invocations
- Response times
- Error rates

### 10.6 Scaling Checklist

```markdown
## Before Scaling
- [ ] Current metrics documented
- [ ] Bottlenecks identified
- [ ] Scaling plan reviewed
- [ ] Backup created
- [ ] Rollback plan prepared

## During Scaling
- [ ] Monitor key metrics
- [ ] Test critical paths
- [ ] Verify performance improvement
- [ ] Check error logs
- [ ] Monitor costs

## After Scaling
- [ ] Document new configuration
- [ ] Update monitoring thresholds
- [ ] Conduct load testing
- [ ] Review cost implications
- [ ] Plan next scaling tier
```

### 10.7 Cost Optimization

**Database**:
- Archive old data to cold storage
- Implement data retention policies
- Use appropriate column types
- Avoid unnecessary indexes

**Edge Functions**:
- Optimize function execution time
- Reduce function invocations
- Batch operations where possible
- Cache responses

**Storage**:
- Compress images before upload
- Implement automatic cleanup
- Use appropriate storage tiers
- Enable lifecycle policies

**Monitoring Costs**:
```sql
-- Track usage metrics
SELECT 
  DATE_TRUNC('day', created_at) as day,
  COUNT(*) as function_invocations,
  SUM(execution_time_ms) as total_execution_time
FROM analytics_events
WHERE event_type = 'function_invocation'
  AND created_at > NOW() - INTERVAL '30 days'
GROUP BY day
ORDER BY day DESC;
```

---

## Appendix A: Quick Reference Commands

### Database Operations
```bash
# Connect to database
psql -h db.xxxxxxxxxxxxx.supabase.co -U postgres -d postgres

# List tables
\dt

# Describe table
\d table_name

# Execute SQL file
psql -f schema.sql

# Export database
pg_dump -F c -f backup.dump

# Import database
pg_restore -d postgres backup.dump
```

### Supabase CLI
```bash
# Login
supabase login

# Link project
supabase link --project-ref xxxxx

# Deploy function
supabase functions deploy function-name

# View logs
supabase functions logs function-name

# Manage secrets
supabase secrets set KEY=value
supabase secrets list
```

### Frontend Operations
```bash
# Install dependencies
pnpm install

# Development server
pnpm run dev

# Build production
pnpm run build

# Preview production build
pnpm run preview
```

---

## Appendix B: Troubleshooting Guide

### Issue: Database Connection Failed
```bash
# Check connection
psql -h db.xxxxxxxxxxxxx.supabase.co -U postgres -d postgres

# Verify credentials
echo $PGPASSWORD

# Check SSL requirement
psql "sslmode=require host=db.xxx.supabase.co dbname=postgres user=postgres"
```

### Issue: Edge Function Timeout
```typescript
// Increase timeout (max 300s)
// Check function logs for bottlenecks
// Optimize database queries
// Use connection pooling
```

### Issue: Frontend Build Fails
```bash
# Clear cache
rm -rf node_modules .pnpm-store
pnpm install

# Check Node version
node --version  # Should be 18+

# Verbose build
pnpm run build --verbose
```

### Issue: RLS Blocking Queries
```sql
-- Temporarily disable RLS for testing (NOT for production)
ALTER TABLE table_name DISABLE ROW LEVEL SECURITY;

-- Check RLS policies
SELECT * FROM pg_policies WHERE tablename = 'table_name';

-- Debug RLS
SET ROLE authenticated;
SELECT * FROM table_name;  -- Test as authenticated user
```

---

## Appendix C: Security Hardening

### Database Security
```sql
-- Revoke public access
REVOKE ALL ON ALL TABLES IN SCHEMA public FROM PUBLIC;

-- Grant specific permissions
GRANT SELECT, INSERT, UPDATE ON conversations TO authenticated;

-- Audit logging
CREATE TABLE audit_log (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID,
  action TEXT,
  table_name TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### API Security
- Rate limiting enabled in Supabase
- CORS configured properly
- API keys rotated regularly
- Service role key never exposed to frontend

### Frontend Security
- Content Security Policy (CSP)
- XSS protection headers
- HTTPS only
- Secure cookie settings

---

## Support & Maintenance

**Documentation**: This guide  
**Database Schema**: `/workspace/database-schema.sql`  
**RLS Policies**: `/workspace/rls-policies.sql`  
**Edge Functions**: `/workspace/supabase/functions/`  
**Frontend Source**: `/workspace/ai-super-assistant/`  

**Support Channels**:
- Supabase Support: https://supabase.com/support
- Community: https://supabase.com/community
- Documentation: https://supabase.com/docs

---

**End of Deployment Guide**

*Last Updated: October 29, 2025*  
*Version: 1.0*  
*Maintained by: Development Team*
