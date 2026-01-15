# Project Research Summary

**Project:** Product Feedback App
**Domain:** Feedback Collection / Internal Tooling
**Researched:** 2026-01-15
**Confidence:** HIGH

## Executive Summary

This research covers building a self-hosted feedback collection tool for personal web applications. The system collects user feedback (bugs, feature requests, ratings) via REST API from multiple apps, managed through a single admin dashboard with GitHub OAuth authentication.

The recommended approach is a monolithic Node.js architecture with clear separation between public API (for feedback submission) and admin API (for dashboard operations). The stack is well-established: Express.js backend, MariaDB database, React frontend, with Multer for file uploads, Passport.js for GitHub OAuth, and Nodemailer for SMTP notifications.

Key risks center on authentication security (OAuth state validation), file upload vulnerabilities, and connection pool management. All are well-documented patterns with established solutions.

## Key Findings

### Recommended Stack

The 2025 standard stack for this type of tool:

**Core technologies:**
- **Node.js 20 LTS + Express 4.x**: Battle-tested, largest ecosystem, straightforward
- **MariaDB 10.11 LTS + mysql2**: User preference, excellent Node.js support with connection pooling
- **React 18 + Vite 5**: Modern frontend, fast builds, CRA is deprecated
- **TypeScript 5.x**: Type safety across the stack

**Supporting libraries:**
- **Multer + Sharp**: File uploads with image processing
- **Passport.js + passport-github2**: OAuth authentication
- **Nodemailer 6.x**: SMTP email notifications
- **Zod 3.x**: Runtime validation with TypeScript inference
- **Prisma**: ORM with excellent TypeScript integration (or Knex.js for more SQL control)

### Expected Features

**Must have (table stakes):**
- REST API for feedback submission with API key authentication
- Feedback types: bug, feature request, general feedback
- Status workflow: new → open → in_progress → resolved → closed
- Priority levels: high/medium/low
- Source app tracking (multi-app support)
- Admin dashboard with filtering and search
- GitHub OAuth for admin access

**Should have (competitive):**
- Screenshot upload support
- Tagging/categorization system
- Email notifications on new feedback
- Export functionality (CSV/JSON)

**Defer (v2+):**
- Embed widget for easier integration
- Webhooks for external integrations
- Statistics dashboard
- Keyboard shortcuts

### Architecture Approach

Monolithic architecture with clear module separation:

```
┌──────────────────────────────────────────────────────────────────┐
│                        FEEDBACK SYSTEM                            │
├──────────────────────────────────────────────────────────────────┤
│  ┌────────────────┐        ┌────────────────┐                    │
│  │  Public API    │        │   Admin API    │                    │
│  │  (API Key)     │        │ (GitHub OAuth) │                    │
│  └───────┬────────┘        └───────┬────────┘                    │
│          │                         │                              │
│          └─────────┬───────────────┘                              │
│                    ▼                                              │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    SERVICE LAYER                             │ │
│  │  FeedbackService | UploadService | NotificationService       │ │
│  └───────────────────────────┬─────────────────────────────────┘ │
│                              ▼                                    │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐      │
│  │    MariaDB     │  │  File Storage  │  │      SMTP      │      │
│  └────────────────┘  └────────────────┘  └────────────────┘      │
└──────────────────────────────────────────────────────────────────┘
```

**Major components:**
1. **Public API** — Feedback submission from client apps (API key auth)
2. **Admin API** — Dashboard operations (session-based GitHub OAuth)
3. **Service Layer** — Business logic, reusable across both APIs
4. **Data Layer** — MariaDB, filesystem/S3, SMTP

### Critical Pitfalls

Top issues to prevent during implementation:

1. **File Upload Security** — Validate by magic bytes not extension, regenerate filenames, store outside webroot
2. **OAuth State Validation** — CSRF protection via cryptographic state parameter is mandatory
3. **Connection Pool Leaks** — Always release connections in finally blocks, monitor pool health
4. **Rate Limiting** — Add to all public endpoints from day one
5. **Session Secret** — Strong (64+ chars), from environment, fail if missing

## Implications for Roadmap

Based on research, suggested phase structure:

### Phase 1: Foundation & API
**Rationale:** Core data model and API are prerequisites for everything
**Delivers:** Database schema, feedback submission API, file upload handling
**Addresses:** REST API, feedback types, screenshot upload, user/app ID tracking
**Avoids:** Connection pool leaks, file upload vulnerabilities, missing rate limiting

### Phase 2: Authentication
**Rationale:** Admin functionality requires auth; do it before building dashboard
**Delivers:** GitHub OAuth integration, session management, admin middleware
**Addresses:** GitHub OAuth authentication for admin access
**Avoids:** OAuth state CSRF, session fixation, weak secrets

### Phase 3: Admin Dashboard
**Rationale:** Core value for admin — viewing and managing feedback
**Delivers:** React SPA, feedback list with filters, detail view, status/priority management
**Addresses:** Admin dashboard, tagging system, priority levels, filtering/search
**Avoids:** N+1 queries, XSS in feedback display, missing pagination

### Phase 4: Notifications
**Rationale:** Nice-to-have, can ship v1 without but adds significant value
**Delivers:** SMTP email notifications on new feedback
**Addresses:** Email notifications requirement
**Avoids:** Connection pool exhaustion, synchronous sending blocking requests

### Phase 5: Production Hardening
**Rationale:** Deploy with confidence
**Delivers:** Docker configuration, health checks, monitoring setup
**Addresses:** Docker deployment configuration
**Avoids:** Disk filling silently, missing health checks, secrets in logs

### Phase Ordering Rationale

- **Phase 1 first**: Everything depends on the data model and API
- **Phase 2 before 3**: Can't build admin dashboard without auth
- **Phase 4 after core**: Notifications are enhancement, not blocker
- **Phase 5 last**: Polish for production deployment

### Research Flags

Phases likely needing deeper research during planning:
- **Phase 2:** OAuth implementation details specific to Passport.js + GitHub

Phases with standard patterns (skip research-phase):
- **Phase 1:** Standard REST API patterns, well-documented
- **Phase 3:** Standard React dashboard patterns
- **Phase 5:** Standard Docker practices

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Node.js/Express/React are mature, well-documented |
| Features | HIGH | Based on analysis of Canny, Nolt, Fider, UserVoice |
| Architecture | HIGH | Standard patterns for this type of tool |
| Pitfalls | HIGH | Well-documented failure modes, OWASP guidance |

**Overall confidence:** HIGH

### Gaps to Address

- **Screenshot storage decision**: Local filesystem vs S3-compatible — can defer to implementation
- **Email notification frequency**: Immediate vs digest — can defer to implementation
- **Database library choice**: Prisma vs Knex.js — recommend Prisma for TypeScript benefits

## Sources

### Primary (HIGH confidence)
- Node.js official documentation
- Express.js best practices
- MariaDB connector documentation
- GitHub OAuth documentation
- Nodemailer documentation
- OWASP security guidelines

### Secondary (MEDIUM confidence)
- Community patterns from Fider (open source feedback tool)
- General SaaS feedback tool feature analysis

---
*Research completed: 2026-01-15*
*Ready for roadmap: yes*
