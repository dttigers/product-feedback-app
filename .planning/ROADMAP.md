# Roadmap: Product Feedback App

## Overview

Build a self-hosted feedback collection tool in 5 phases: establish the data foundation and API, add admin authentication, build the dashboard UI, add email notifications, then package for Docker deployment.

## Domain Expertise

None

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

- [ ] **Phase 1: Foundation & API** - Database schema and feedback submission REST API
- [ ] **Phase 2: Authentication** - GitHub OAuth for admin access
- [ ] **Phase 3: Admin Dashboard** - React SPA for viewing and managing feedback
- [ ] **Phase 4: Notifications** - SMTP email notifications on new feedback
- [ ] **Phase 5: Deployment** - Docker configuration for self-hosting

## Phase Details

### Phase 1: Foundation & API
**Goal**: Database schema and feedback submission REST API
**Depends on**: Nothing (first phase)
**Requirements**: API-01, API-02, API-03, API-04, API-05
**Research**: Unlikely (established patterns)
**Plans**: TBD

Plans:
- [ ] 01-01: Project setup and database schema
- [ ] 01-02: Feedback submission API endpoint
- [ ] 01-03: App registration and validation

### Phase 2: Authentication
**Goal**: GitHub OAuth for admin access with session persistence
**Depends on**: Phase 1
**Requirements**: AUTH-01, AUTH-02, AUTH-03
**Research**: Likely (OAuth implementation)
**Research topics**: Passport.js + GitHub OAuth setup, session management, admin restriction by GitHub user ID
**Plans**: TBD

Plans:
- [ ] 02-01: GitHub OAuth integration
- [ ] 02-02: Session management and admin middleware

### Phase 3: Admin Dashboard
**Goal**: React SPA for viewing and managing feedback with filters
**Depends on**: Phase 2
**Requirements**: DASH-01, DASH-02, DASH-03, DASH-04, DASH-05
**Research**: Unlikely (standard React patterns)
**Plans**: TBD

Plans:
- [ ] 03-01: Dashboard setup and feedback list view
- [ ] 03-02: Feedback detail view
- [ ] 03-03: Filtering and sorting

### Phase 4: Notifications
**Goal**: SMTP email notifications when new feedback is submitted
**Depends on**: Phase 1 (API must exist to trigger notifications)
**Requirements**: NOTF-01, NOTF-02
**Research**: Unlikely (Nodemailer is straightforward)
**Plans**: TBD

Plans:
- [ ] 04-01: SMTP integration and notification service

### Phase 5: Deployment
**Goal**: Docker configuration for easy self-hosting
**Depends on**: All previous phases
**Requirements**: DEPLOY-01, DEPLOY-02, DEPLOY-03
**Research**: Unlikely (standard Docker practices)
**Plans**: TBD

Plans:
- [ ] 05-01: Dockerfile and docker-compose
- [ ] 05-02: Environment configuration and documentation

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation & API | 0/3 | Not started | - |
| 2. Authentication | 0/2 | Not started | - |
| 3. Admin Dashboard | 0/3 | Not started | - |
| 4. Notifications | 0/1 | Not started | - |
| 5. Deployment | 0/2 | Not started | - |
