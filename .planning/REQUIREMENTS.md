# Requirements: Product Feedback App

**Defined:** 2026-01-19
**Core Value:** Centralized feedback collection and management for personal web applications

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Feedback Submission

- [ ] **API-01**: App can submit feedback via REST API with content and type
- [ ] **API-02**: Feedback includes source app identifier (which app submitted it)
- [ ] **API-03**: Feedback includes user ID passthrough from host app
- [ ] **API-04**: Feedback has type field (bug, feature, general)
- [ ] **API-05**: Feedback receives unique ID and timestamps on creation

### Admin Dashboard

- [ ] **DASH-01**: Admin can view list of all feedback items
- [ ] **DASH-02**: Admin can view individual feedback details
- [ ] **DASH-03**: Admin can filter feedback by source app
- [ ] **DASH-04**: Admin can filter feedback by type
- [ ] **DASH-05**: Admin can sort feedback by date

### Authentication

- [ ] **AUTH-01**: Admin can sign in with GitHub OAuth
- [ ] **AUTH-02**: Admin session persists across browser refresh
- [ ] **AUTH-03**: Only configured GitHub user ID can access admin

### Notifications

- [ ] **NOTF-01**: Admin receives email when new feedback is submitted
- [ ] **NOTF-02**: Email includes feedback content, type, and source app

### Deployment

- [ ] **DEPLOY-01**: Application runs via Docker container
- [ ] **DEPLOY-02**: Configuration via environment variables
- [ ] **DEPLOY-03**: docker-compose for easy local/production setup

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Feedback Submission

- **API-06**: Screenshot upload support
- **API-07**: Rating/satisfaction score field
- **API-08**: Metadata field (arbitrary JSON)

### Admin Dashboard

- **DASH-06**: Status workflow (new → open → in_progress → resolved → closed)
- **DASH-07**: Priority levels (high/medium/low)
- **DASH-08**: Tagging/categorization system
- **DASH-09**: Bulk actions
- **DASH-10**: Search by keyword
- **DASH-11**: Admin notes field

### Authentication

- **AUTH-04**: API keys for app authentication (currently open API)

### Deployment

- **DEPLOY-04**: Health check endpoints
- **DEPLOY-05**: Database migrations system

### Data Management

- **DATA-01**: Export to CSV/JSON
- **DATA-02**: Archiving system

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Multi-tenancy / multiple admins | Personal tooling — single admin |
| Public feedback portal | API-only, feedback comes from apps |
| Voting/upvoting | No public users |
| Issue tracker integration (Jira, Linear) | Complexity, can add later via webhooks |
| Embed widget | API-first approach |
| Real-time updates | WebSocket complexity, polling sufficient |
| Mobile app | Responsive web sufficient |

## Traceability

Which phases cover which requirements. Updated by create-roadmap.

| Requirement | Phase | Status |
|-------------|-------|--------|
| API-01 | — | Pending |
| API-02 | — | Pending |
| API-03 | — | Pending |
| API-04 | — | Pending |
| API-05 | — | Pending |
| DASH-01 | — | Pending |
| DASH-02 | — | Pending |
| DASH-03 | — | Pending |
| DASH-04 | — | Pending |
| DASH-05 | — | Pending |
| AUTH-01 | — | Pending |
| AUTH-02 | — | Pending |
| AUTH-03 | — | Pending |
| NOTF-01 | — | Pending |
| NOTF-02 | — | Pending |
| DEPLOY-01 | — | Pending |
| DEPLOY-02 | — | Pending |
| DEPLOY-03 | — | Pending |

**Coverage:**
- v1 requirements: 18 total
- Mapped to phases: 0
- Unmapped: 18 (will be mapped by create-roadmap)

---
*Requirements defined: 2026-01-19*
*Last updated: 2026-01-19 after initial definition*
