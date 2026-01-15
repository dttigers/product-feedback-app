# Product Feedback App

> Centralized feedback collection and management for personal web applications

## Vision

A self-hosted tool to collect user feedback across multiple web apps you build. Users submit feedback (bugs, feature requests, ratings) via API, and you manage it all from a single admin dashboard with tagging, prioritization, and email notifications.

## Problem

When running multiple web apps, user feedback is scattered or lost. You need a unified system to:
- Capture feedback from any app via API
- See all feedback in one place
- Organize and prioritize what to work on
- Know when new feedback arrives

## Target Users

- **Feedback submitters:** End users of your web apps (identified by user ID passed from host app)
- **Admin:** You — the only person accessing the dashboard

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] REST API for submitting feedback from web apps
- [ ] Support multiple feedback types: bug reports, feature requests, general feedback
- [ ] Text content field for feedback details
- [ ] Optional screenshot upload (user-provided)
- [ ] Optional rating/satisfaction score
- [ ] User ID passthrough from host app to identify submitter
- [ ] App ID to separate feedback by source application
- [ ] Admin dashboard for viewing and managing feedback
- [ ] GitHub OAuth authentication for admin access
- [ ] Tagging/categorization system for feedback items
- [ ] Priority levels (high/medium/low) for triage
- [ ] SMTP email notifications on new feedback submission
- [ ] Docker deployment configuration

### Out of Scope

- Multi-tenancy / multiple admin users — this is personal tooling
- Billing or subscription management
- Public-facing feedback portal — API only
- Automatic screenshot capture — user uploads only
- Voting/upvoting by other users
- Integration with external issue trackers (Jira, Linear, etc.)

## Technical Constraints

### Stack

| Layer | Technology | Rationale |
|-------|------------|-----------|
| Backend | Node.js (Express or Fastify) | Simple, good ecosystem, JavaScript throughout |
| Database | MariaDB | User preference, solid relational DB |
| Frontend | React | Admin dashboard SPA |
| Auth | Passport.js + GitHub OAuth | Single admin, simple OAuth flow |
| File Storage | Local filesystem or S3-compatible | Screenshot uploads |
| Email | SMTP | Notification delivery |
| Deployment | Docker | Self-hosted containers |

### Non-Functional

- Self-hosted on Docker
- Single admin user (no complex RBAC)
- API should be simple to integrate from any web app

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| MariaDB over PostgreSQL | User preference | — Pending |
| GitHub OAuth for admin | Simplest OAuth for single developer | — Pending |
| SMTP over transactional email service | Self-hosted preference, more control | — Pending |
| No public feedback portal | API-first approach, feedback comes from apps | — Pending |

## Open Questions

- Screenshot storage: local filesystem vs S3-compatible? (can decide during implementation)
- Email notification frequency: immediate per-feedback or daily digest?

---
*Last updated: 2026-01-15 after initialization*
