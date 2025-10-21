<div align="center">

# Streaky - GitHub Streak Guardian

[![GitHub Stars](https://img.shields.io/github/stars/0xReLogic/Streaky?style=social)](https://github.com/0xReLogic/Streaky/stargazers)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Live Demo](https://img.shields.io/badge/demo-live-success)](https://streakyy.vercel.app)

**Never lose your GitHub streak again!**

### [ Main Repository](https://github.com/0xReLogic/Streaky) • [ Live Demo](https://streakyy.vercel.app)

</div>

---

###  Overview

Streaky is a web application that monitors your GitHub contribution streak and sends notifications when you're at risk of breaking it. Built with a distributed architecture to handle concurrent user processing efficiently.

###  The Problem

GitHub streaks represent consistency in coding habits, but it's easy to forget to commit on busy days. Existing solutions either:
- Require manual checking
- Don't support multiple notification platforms
- Lack proper security for credentials
- Can't scale efficiently

###  The Solution

A fully automated system with:
- **Distributed Processing**: Each user processed in isolated Worker instance
- **Multi-Platform Notifications**: Discord & Telegram support
- **Zero-Knowledge Security**: AES-256-GCM encryption, credentials never stored in plaintext
- **Scalable Architecture**: Built on Cloudflare's edge network

###  Technical Architecture

```
┌─────────────────┐
│   Next.js 15    │ ← User Interface
│   (Vercel)      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Cloudflare      │ ← API & Cron Jobs
│ Workers + D1    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Rust Proxy     │ ← Notification Delivery
│   (Koyeb)       │
└─────────────────┘
```

###  Tech Stack

**Frontend**
- Next.js 15 (App Router)
- React 19
- TypeScript
- Tailwind CSS + shadcn/ui
- NextAuth.js v5

**Backend**
- Cloudflare Workers
- Cloudflare D1 (SQLite)
- Hono framework
- Service Bindings for distributed processing

**Infrastructure**
- Rust notification proxy (Axum framework)
- GitHub OAuth
- Idempotent queue system
- Automated daily cron at 12:00 UTC

###  Key Technical Challenges Solved

#### 1. Distributed Cron Processing
**Challenge**: Process thousands of users within Cloudflare Workers' 30-second CPU time limit.

**Solution**: Implemented Service Bindings to spawn separate Worker instances for each user, enabling true parallel processing with isolated execution.

```typescript
// Each user gets their own Worker instance
c.executionCtx.waitUntil(
  c.env.SELF.fetch('http://internal/api/cron/process-user', {
    method: 'POST',
    body: JSON.stringify({ userId })
  })
);
```

#### 2. Idempotent Queue System
**Challenge**: Prevent duplicate processing when cron jobs overlap or retry.

**Solution**: D1-based queue with atomic claim operations and batch tracking.

```sql
-- Atomic claim with status check
UPDATE cron_queue 
SET status = 'processing', claimed_at = CURRENT_TIMESTAMP
WHERE user_id = ? AND status = 'pending'
```

#### 3. Secure Credential Storage
**Challenge**: Store Discord/Telegram webhooks securely without exposing them.

**Solution**: AES-256-GCM encryption with per-user keys derived from server secrets.

```typescript
// Encrypt before storing
const encrypted = await encrypt(webhook, encryptionKey);
// Decrypt only when needed
const decrypted = await decrypt(encrypted, encryptionKey);
```

#### 4. Zero-Knowledge Architecture
**Challenge**: Minimize data exposure and maintain user privacy.

**Solution**: 
- GitHub tokens never stored (OAuth refresh flow)
- Webhooks encrypted at rest
- Notifications sent via isolated Rust proxy
- No logging of sensitive data

###  Project Stats

- **Lines of Code**: ~5,000+ (TypeScript, Rust, Python)
- **Components**: 3 (Web App, CLI, Notification Proxy)
- **Database Tables**: 4 with optimized indexes
- **API Endpoints**: 15+
- **Test Coverage**: Unit tests for critical paths

###  Design Philosophy

- **Clean & Minimal**: Black and white design, no distractions
- **Privacy-First**: Zero-knowledge architecture, encrypted credentials
- **Developer-Friendly**: Comprehensive docs, env examples, contributing guide
- **Production-Ready**: Error handling, logging, monitoring

###  Live Demo

Try it yourself: **[streakyy.vercel.app](https://streakyy.vercel.app)**

###  What I Learned

- Building distributed systems with Cloudflare Workers
- Implementing idempotent operations for reliability
- Designing zero-knowledge architectures
- Managing monorepo with multiple languages (TypeScript, Rust, Python)
- Writing comprehensive documentation for open-source projects
- Setting up GitHub automation (issue templates, auto-labeling)

---

<div align="center">

##  Links

**[ Main Repository](https://github.com/0xReLogic/Streaky)** | **[ Live Demo](https://streakyy.vercel.app)** | **[ Documentation](https://github.com/0xReLogic/Streaky#readme)**

**[ Report Issues](https://github.com/0xReLogic/Streaky/issues)** | **[ Contributing Guide](https://github.com/0xReLogic/Streaky/blob/main/CONTRIBUTING.md)**

---

**If you find this project interesting, consider giving it a star!**

</div>
