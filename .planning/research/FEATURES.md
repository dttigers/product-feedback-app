# Feedback Collection Tool Features Research

> **Research Date**: January 2025
> **Source**: Analysis based on Canny, UserVoice, Nolt, Fider, and similar tools
> **Scope**: Single-admin personal feedback tool (self-hosted, multi-app)
> **Note**: Based on domain knowledge; web research was unavailable

---

## Table Stakes (Users Expect These)

These features are baseline expectations. Missing any creates friction or makes the tool feel incomplete.

| Feature | Description | Confidence |
|---------|-------------|------------|
| **Feedback submission form** | Simple form with title, description, optional category | HIGH |
| **REST API for submission** | Programmatic feedback ingestion from multiple apps | HIGH |
| **List/browse feedback** | View all feedback with basic sorting (date, status) | HIGH |
| **Status tracking** | Open/In Progress/Completed/Closed states | HIGH |
| **Basic search** | Find feedback by keyword in title/description | HIGH |
| **Filtering by status** | Filter view by current status | HIGH |
| **Source/app identification** | Know which app feedback came from | HIGH |
| **Timestamps** | Created date, updated date | HIGH |
| **Unique identifiers** | Reference feedback by ID | HIGH |
| **Basic auth for admin** | Protect the dashboard | HIGH |

### Why These Are Table Stakes
- Every competitor (Canny, Nolt, Fider) has these
- Users will immediately notice if missing
- Core to the feedback management workflow

---

## Differentiators (Competitive Advantage)

For a **personal self-hosted tool**, differentiation comes from:

### High-Value Differentiators (Personal Use)

| Feature | Value Proposition | Confidence |
|---------|-------------------|------------|
| **Zero-config deployment** | Docker single command, no database setup needed | HIGH |
| **Offline-first / Local storage** | Works without cloud, SQLite embedded | MEDIUM |
| **Simple embed widget** | One script tag to add feedback to any app | HIGH |
| **Keyboard shortcuts** | Power-user efficiency for triage | MEDIUM |
| **Quick archive/bulk actions** | Handle high volume feedback fast | HIGH |
| **Custom fields per app** | Different metadata per source app | MEDIUM |
| **Markdown support** | Rich formatting in descriptions | MEDIUM |
| **Dark mode** | Developer preference for admin tools | MEDIUM |

### Lower-Value for Personal Use (But Common in Market)

| Feature | Why Lower Value for Single Admin | Confidence |
|---------|----------------------------------|------------|
| **Public voting** | No public users in personal tool | HIGH |
| **Roadmap boards** | Overkill for personal project tracking | HIGH |
| **User identity/SSO** | Single admin doesn't need | HIGH |
| **Email notifications** | You're the only user | MEDIUM |
| **Changelog publishing** | No audience to notify | HIGH |

---

## Anti-Features (Commonly Requested But Problematic)

Features that seem valuable but add complexity or maintenance burden:

| Feature | Problem | Alternative | Confidence |
|---------|---------|-------------|------------|
| **AI categorization** | Requires ML infrastructure, API costs, accuracy issues | Manual tags + search | HIGH |
| **Sentiment analysis** | Often inaccurate, adds complexity | Read the feedback | HIGH |
| **Email submission** | Email parsing is fragile, spam issues | API-only submission | HIGH |
| **Screenshot upload** | Storage complexity, security concerns | External image link field | MEDIUM |
| **Multi-tenant** | Massive scope increase for personal tool | Keep single-tenant | HIGH |
| **Plugin system** | Maintenance burden, security surface | Direct code customization | MEDIUM |
| **Real-time updates** | WebSocket complexity | Polling / manual refresh | MEDIUM |
| **Advanced analytics** | YAGNI for personal use | Export to spreadsheet | HIGH |
| **Native mobile app** | Separate codebase to maintain | Responsive web | HIGH |
| **Integrations (Slack, Jira)** | Auth complexity, maintenance | Webhooks out (you build) | MEDIUM |

### When Anti-Features Become Features
- **Screenshot upload**: Becomes valuable if you add simple file storage (S3-compatible)
- **Real-time**: Valuable if you have >100 feedback/day
- **Integrations**: Valuable if you use the same stack consistently

---

## Feature Dependencies (What Requires What)

```
Core Layer (Must Have First)
├── Database schema & models
├── REST API (CRUD operations)
└── Basic auth

Feedback Management Layer (Build on Core)
├── Status workflow ─────────► Requires: Core
├── Filtering/Search ────────► Requires: Core
├── Source app tracking ─────► Requires: Core schema design
└── Categories/Tags ─────────► Requires: Core (simple) or Tag system

User Experience Layer (Build on Management)
├── Dashboard UI ────────────► Requires: Management layer
├── Bulk actions ────────────► Requires: Dashboard UI
├── Keyboard shortcuts ──────► Requires: Dashboard UI
└── Dark mode ───────────────► Requires: Dashboard UI

Integration Layer (Build on Everything)
├── Embed widget ────────────► Requires: API + CORS config
├── Webhooks out ────────────► Requires: Core + Event system
└── Export (CSV/JSON) ───────► Requires: API
```

### Dependency Notes
- **Tags/Categories**: Can be simple (single field) or complex (many-to-many). Start simple.
- **Search**: Full-text search adds DB complexity. LIKE queries work for <10k items.
- **Auth**: Session-based is simpler than JWT for single-admin.

---

## MVP Definition

### Launch With (Week 1-2)

| Feature | Rationale |
|---------|-----------|
| SQLite database | Zero-config, portable, sufficient for personal use |
| Feedback CRUD API | Core functionality |
| Status field (Open/In Progress/Done/Closed) | Minimal workflow |
| Source app identifier | Track which app feedback came from |
| Basic admin dashboard | List, view, update status |
| Single admin auth | Password-based, session |
| Category/tag field | Single string, not relational |
| Date filtering | Last 7/30/90 days |

### Add Later (Week 3-4)

| Feature | Rationale |
|---------|-----------|
| Full-text search | Quality of life as volume grows |
| Embed widget (JS) | Easier than API integration for some apps |
| Bulk status updates | Efficiency for triage |
| Export to CSV | Data portability |
| Dark mode | Polish |
| Keyboard shortcuts | Power user efficiency |

### Future (Month 2+)

| Feature | Rationale |
|---------|-----------|
| Webhooks (outbound) | Integration flexibility |
| Custom fields per source | Advanced customization |
| Archiving system | Keep DB clean, retain history |
| Statistics dashboard | Insights on feedback patterns |
| API rate limiting | If exposing publicly |
| Backup/restore utility | Data safety |

### Explicitly Not Building

| Feature | Reason |
|---------|--------|
| User voting/upvotes | No public users |
| Public roadmap | No audience |
| Multi-tenant | Not needed for personal use |
| Email notifications | Single admin |
| Comment threads | Feedback is one-way |
| User accounts | Single admin only |
| OAuth/SSO | Overkill |
| Mobile app | Responsive web sufficient |

---

## Feature Prioritization Matrix

### Impact vs. Effort Matrix

```
                    LOW EFFORT                          HIGH EFFORT
              ┌─────────────────────────────┬─────────────────────────────┐
              │                             │                             │
              │  ★ QUICK WINS               │  ★ MAJOR PROJECTS           │
              │                             │                             │
  HIGH        │  • Status tracking          │  • Embed widget             │
  IMPACT      │  • Basic filtering          │  • Full-text search         │
              │  • Source app field         │  • Bulk actions             │
              │  • Categories               │  • Export system            │
              │  • Date sorting             │                             │
              │                             │                             │
              ├─────────────────────────────┼─────────────────────────────┤
              │                             │                             │
              │  ★ FILL-INS                 │  ★ AVOID/DEFER              │
              │                             │                             │
  LOW         │  • Dark mode                │  • Real-time updates        │
  IMPACT      │  • Keyboard shortcuts       │  • AI categorization        │
              │  • Markdown preview         │  • Integrations             │
              │                             │  • Analytics dashboard      │
              │                             │  • Screenshot upload        │
              │                             │                             │
              └─────────────────────────────┴─────────────────────────────┘
```

### Priority Order (Recommended)

1. **P0 - Core** (Can't launch without)
   - REST API with CRUD
   - Database models
   - Admin authentication
   - Basic list view

2. **P1 - Functional** (Launch feels incomplete without)
   - Status workflow
   - Source app tracking
   - Filtering & sorting
   - Single feedback view

3. **P2 - Usable** (Day-to-day usage comfort)
   - Search
   - Bulk actions
   - Keyboard shortcuts
   - Export

4. **P3 - Polish** (Nice to have)
   - Dark mode
   - Embed widget
   - Webhooks
   - Statistics

---

## Competitive Analysis Summary

### What Canny Does Well
- Clean UI for feedback boards
- Voting system for prioritization
- Roadmap visualization
- Changelog integration

**Relevant to personal tool**: Clean UI patterns, status workflow

### What UserVoice Does Well
- Enterprise feedback aggregation
- Analytics and reporting
- Integration ecosystem

**Relevant to personal tool**: Almost nothing (enterprise focus)

### What Nolt Does Well
- Simple setup
- Clean design
- Privacy-friendly options

**Relevant to personal tool**: Simplicity as a feature, clean design

### What Fider Does Well
- Open source, self-hostable
- Simple feature set
- Community-driven

**Relevant to personal tool**: Self-hosted model, minimal feature set, reference architecture

---

## Key Insights for Personal Tool

1. **Simplicity is the feature** - Enterprise tools are bloated for personal use
2. **Status workflow is core** - Everything else is secondary
3. **Multi-app support matters** - Distinguishes from single-project tools
4. **No public features needed** - Voting, roadmaps, changelogs are irrelevant
5. **Local-first is valuable** - Self-hosted means no cloud dependency
6. **Export is important** - Data portability for personal tools
7. **API-first enables flexibility** - Let apps submit however they want

---

## Confidence Levels Explained

- **HIGH**: Based on consistent patterns across multiple tools, well-established UX patterns
- **MEDIUM**: Based on reasonable inference, may vary by specific use case
- **LOW**: Speculative, needs validation with actual usage

---

## Sources & References

- Canny.io feature documentation
- Fider.io (open source reference implementation)
- Nolt.io feature pages
- UserVoice product pages
- General SaaS feedback tool patterns (2023-2025)
- Personal experience with feedback tool implementations

> **Limitation**: Live web research was unavailable. Findings based on training data knowledge of these tools. Recommend verifying current feature sets if specific integrations are planned.
