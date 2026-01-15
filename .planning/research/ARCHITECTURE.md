# Architecture Research: Feedback Collection System

> Research compiled: 2026-01-15
> Tech stack: Node.js (Express/Fastify), MariaDB, React, GitHub OAuth, SMTP, Docker

---

## System Overview

```
                                    FEEDBACK COLLECTION SYSTEM

    ┌─────────────────────────────────────────────────────────────────────────────┐
    │                              EXTERNAL CLIENTS                                │
    │  ┌──────────┐  ┌──────────┐  ┌──────────┐                                   │
    │  │  App 1   │  │  App 2   │  │  App N   │   (Your web applications)         │
    │  └────┬─────┘  └────┬─────┘  └────┬─────┘                                   │
    │       │             │             │                                          │
    │       └─────────────┼─────────────┘                                          │
    │                     │                                                        │
    │                     ▼                                                        │
    │  ┌─────────────────────────────────────┐                                    │
    │  │           API Gateway Layer          │  Rate limiting, API key validation │
    │  │         (Express/Fastify)            │                                    │
    │  └─────────────────┬───────────────────┘                                    │
    │                    │                                                         │
    └────────────────────┼─────────────────────────────────────────────────────────┘
                         │
    ┌────────────────────┼─────────────────────────────────────────────────────────┐
    │                    ▼                            BACKEND SERVICES             │
    │  ┌─────────────────────────────────────┐                                    │
    │  │          REST API Routes             │                                    │
    │  │  POST /api/v1/feedback               │                                    │
    │  │  GET  /api/v1/feedback               │                                    │
    │  │  GET  /api/v1/feedback/:id           │                                    │
    │  └─────────────────┬───────────────────┘                                    │
    │                    │                                                         │
    │     ┌──────────────┼──────────────┬──────────────┐                          │
    │     ▼              ▼              ▼              ▼                          │
    │  ┌──────┐    ┌──────────┐   ┌──────────┐   ┌──────────┐                     │
    │  │Upload│    │ Feedback │   │   Tag    │   │   App    │   Service Layer     │
    │  │Service    │ Service  │   │ Service  │   │ Service  │                     │
    │  └──┬───┘    └────┬─────┘   └────┬─────┘   └────┬─────┘                     │
    │     │             │              │              │                            │
    │     │             └──────────────┼──────────────┘                            │
    │     │                            │                                           │
    │     ▼                            ▼                                           │
    │  ┌──────────┐            ┌──────────────┐                                   │
    │  │  File    │            │   MariaDB    │                                   │
    │  │ Storage  │            │   Database   │                                   │
    │  │(Local/S3)│            │              │                                   │
    │  └──────────┘            └──────────────┘                                   │
    │                                                                              │
    │     ┌──────────────────────────────────┐                                    │
    │     │        Notification Service       │                                    │
    │     │     (SMTP Email on new feedback)  │                                    │
    │     └──────────────────────────────────┘                                    │
    │                                                                              │
    └──────────────────────────────────────────────────────────────────────────────┘
                         │
    ┌────────────────────┼─────────────────────────────────────────────────────────┐
    │                    ▼                            ADMIN DASHBOARD              │
    │  ┌─────────────────────────────────────┐                                    │
    │  │           React SPA                  │                                    │
    │  │  - Feedback list/detail views        │                                    │
    │  │  - Tagging & prioritization          │                                    │
    │  │  - App management                    │                                    │
    │  └─────────────────────────────────────┘                                    │
    │                    │                                                         │
    │                    ▼                                                         │
    │  ┌─────────────────────────────────────┐                                    │
    │  │         GitHub OAuth Flow            │                                    │
    │  │   (Admin authentication only)        │                                    │
    │  └─────────────────────────────────────┘                                    │
    │                                                                              │
    └──────────────────────────────────────────────────────────────────────────────┘
```

**Confidence: HIGH** - This layered architecture is standard for REST APIs with separate admin interfaces.

---

## Component Responsibilities

### 1. API Layer (Public-facing)
| Component | Responsibility | Authentication |
|-----------|----------------|----------------|
| Feedback Submission | Accept feedback from client apps | API Key per app |
| File Upload | Handle screenshot uploads | API Key per app |
| Health Check | System status endpoint | None |

### 2. Admin API Layer (Dashboard-facing)
| Component | Responsibility | Authentication |
|-----------|----------------|----------------|
| Feedback Management | CRUD operations on feedback | GitHub OAuth session |
| Tag Management | Create/edit/delete tags | GitHub OAuth session |
| App Management | Register/manage API keys | GitHub OAuth session |
| Analytics | Basic counts and stats | GitHub OAuth session |

### 3. Service Layer
| Service | Responsibility |
|---------|----------------|
| FeedbackService | Business logic for feedback CRUD, validation |
| UploadService | File validation, storage abstraction, path generation |
| NotificationService | Email composition, SMTP sending, rate limiting |
| TagService | Tag CRUD, feedback-tag associations |
| AppService | App registration, API key generation/validation |
| AuthService | GitHub OAuth flow, session management |

### 4. Data Layer
| Component | Responsibility |
|-----------|----------------|
| Database (MariaDB) | Structured data persistence |
| File Storage | Screenshot/attachment binary storage |

**Confidence: HIGH** - Clear separation of concerns follows established patterns.

---

## Recommended Project Structure

```
feedback-app/
├── docker-compose.yml
├── Dockerfile
├── package.json
├── .env.example
├── README.md
│
├── src/
│   ├── index.js                    # Application entry point
│   ├── app.js                      # Express/Fastify app setup
│   │
│   ├── config/
│   │   ├── index.js                # Config aggregation
│   │   ├── database.js             # MariaDB connection config
│   │   ├── auth.js                 # OAuth & session config
│   │   ├── storage.js              # File storage config
│   │   └── email.js                # SMTP config
│   │
│   ├── routes/
│   │   ├── index.js                # Route aggregation
│   │   ├── api/                    # Public API routes (API key auth)
│   │   │   ├── index.js
│   │   │   ├── feedback.routes.js
│   │   │   └── health.routes.js
│   │   │
│   │   └── admin/                  # Admin routes (OAuth session auth)
│   │       ├── index.js
│   │       ├── auth.routes.js
│   │       ├── feedback.routes.js
│   │       ├── tags.routes.js
│   │       └── apps.routes.js
│   │
│   ├── middleware/
│   │   ├── apiKeyAuth.js           # Validate X-API-Key header
│   │   ├── sessionAuth.js          # Validate OAuth session
│   │   ├── errorHandler.js         # Global error handling
│   │   ├── rateLimiter.js          # Rate limiting
│   │   ├── upload.js               # Multer config for file uploads
│   │   └── validator.js            # Request validation (Joi/Zod)
│   │
│   ├── services/
│   │   ├── feedback.service.js
│   │   ├── upload.service.js
│   │   ├── notification.service.js
│   │   ├── tag.service.js
│   │   ├── app.service.js
│   │   └── auth.service.js
│   │
│   ├── models/                     # Database models/queries
│   │   ├── index.js
│   │   ├── feedback.model.js
│   │   ├── tag.model.js
│   │   ├── app.model.js
│   │   └── session.model.js
│   │
│   ├── db/
│   │   ├── connection.js           # MariaDB pool setup
│   │   └── migrations/             # SQL migration files
│   │       ├── 001_initial_schema.sql
│   │       ├── 002_add_tags.sql
│   │       └── ...
│   │
│   └── utils/
│       ├── logger.js               # Logging utility (pino/winston)
│       ├── apiKeyGenerator.js      # Secure API key generation
│       └── emailTemplates.js       # Email content templates
│
├── uploads/                        # Local file storage (if not S3)
│   └── .gitkeep
│
├── client/                         # React admin dashboard
│   ├── package.json
│   ├── vite.config.js
│   ├── index.html
│   └── src/
│       ├── main.jsx
│       ├── App.jsx
│       ├── api/                    # API client functions
│       │   └── client.js
│       ├── components/
│       │   ├── Layout/
│       │   ├── FeedbackList/
│       │   ├── FeedbackDetail/
│       │   ├── TagManager/
│       │   └── AppManager/
│       ├── pages/
│       │   ├── Dashboard.jsx
│       │   ├── FeedbackPage.jsx
│       │   ├── SettingsPage.jsx
│       │   └── LoginPage.jsx
│       ├── hooks/
│       │   └── useFeedback.js
│       └── context/
│           └── AuthContext.jsx
│
└── scripts/
    ├── migrate.js                  # Run database migrations
    └── seed.js                     # Optional: seed test data
```

**Confidence: HIGH** - Standard Node.js monorepo structure with clear separation.

---

## Architectural Patterns

### API Design

#### Public API (Feedback Submission)

```
POST /api/v1/feedback
Headers:
  X-API-Key: <app-specific-key>
  Content-Type: multipart/form-data  (if screenshot)
               application/json      (if no files)

Body:
{
  "type": "bug" | "feature" | "feedback",
  "content": "Description text...",
  "userId": "user-123-from-host-app",  // Optional: passed from host app
  "rating": 4,                          // Optional: 1-5
  "metadata": {                         // Optional: arbitrary JSON
    "page": "/settings",
    "browser": "Chrome 120"
  }
}
+ screenshot file (optional)

Response: 201 Created
{
  "id": "fb_abc123",
  "status": "received",
  "createdAt": "2026-01-15T10:30:00Z"
}
```

#### Admin API (Dashboard)

```
# List feedback with filters
GET /admin/api/feedback
  ?app=app_xyz
  &type=bug
  &priority=high
  &tag=ui-issue
  &status=open
  &page=1
  &limit=25

# Get single feedback
GET /admin/api/feedback/:id

# Update feedback (tags, priority, status)
PATCH /admin/api/feedback/:id
{
  "priority": "high",
  "status": "in-progress",
  "tags": ["ui-issue", "needs-investigation"]
}

# Delete feedback
DELETE /admin/api/feedback/:id

# Tag management
GET    /admin/api/tags
POST   /admin/api/tags         { "name": "ui-issue", "color": "#FF5733" }
PATCH  /admin/api/tags/:id
DELETE /admin/api/tags/:id

# App management
GET    /admin/api/apps
POST   /admin/api/apps         { "name": "My Blog", "domain": "blog.example.com" }
POST   /admin/api/apps/:id/rotate-key   # Generate new API key
DELETE /admin/api/apps/:id
```

**Confidence: HIGH** - RESTful patterns are well-established.

### Authentication Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        PUBLIC API AUTH (API Keys)                       │
│                                                                         │
│   Client App                    Feedback API                            │
│       │                              │                                  │
│       │  POST /api/v1/feedback       │                                  │
│       │  X-API-Key: fb_live_xxx      │                                  │
│       │─────────────────────────────>│                                  │
│       │                              │                                  │
│       │                        ┌─────┴─────┐                            │
│       │                        │ Lookup key │                           │
│       │                        │ in apps    │                           │
│       │                        │ table      │                           │
│       │                        └─────┬─────┘                            │
│       │                              │                                  │
│       │     201 { id: "fb_123" }     │  (key valid + app active)        │
│       │<─────────────────────────────│                                  │
│       │                              │                                  │
│       │     401 Unauthorized         │  (key invalid/inactive)          │
│       │<─────────────────────────────│                                  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                      ADMIN AUTH (GitHub OAuth)                          │
│                                                                         │
│   Browser                      Server                     GitHub        │
│      │                           │                           │          │
│      │  GET /admin/auth/login    │                           │          │
│      │──────────────────────────>│                           │          │
│      │                           │                           │          │
│      │  302 Redirect to GitHub   │                           │          │
│      │<──────────────────────────│                           │          │
│      │                           │                           │          │
│      │  GitHub OAuth consent screen                          │          │
│      │──────────────────────────────────────────────────────>│          │
│      │                           │                           │          │
│      │  302 Redirect to /admin/auth/callback?code=xxx        │          │
│      │<──────────────────────────────────────────────────────│          │
│      │                           │                           │          │
│      │  GET /admin/auth/callback │                           │          │
│      │──────────────────────────>│                           │          │
│      │                           │  Exchange code for token  │          │
│      │                           │──────────────────────────>│          │
│      │                           │                           │          │
│      │                           │  User profile (id, login) │          │
│      │                           │<──────────────────────────│          │
│      │                           │                           │          │
│      │                     ┌─────┴─────┐                     │          │
│      │                     │ Verify    │                     │          │
│      │                     │ GitHub ID │                     │          │
│      │                     │ matches   │                     │          │
│      │                     │ ADMIN_ID  │                     │          │
│      │                     └─────┬─────┘                     │          │
│      │                           │                           │          │
│      │  Set session cookie       │                           │          │
│      │  302 Redirect to /admin   │                           │          │
│      │<──────────────────────────│                           │          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

**Single Admin Verification:**
```javascript
// config/auth.js
module.exports = {
  github: {
    clientId: process.env.GITHUB_CLIENT_ID,
    clientSecret: process.env.GITHUB_CLIENT_SECRET,
    callbackURL: '/admin/auth/callback'
  },
  // Only this GitHub user ID can access admin
  allowedAdminId: process.env.ADMIN_GITHUB_ID  // e.g., "12345678"
};

// In OAuth callback:
if (profile.id !== config.auth.allowedAdminId) {
  return res.status(403).json({ error: 'Not authorized' });
}
```

**Confidence: HIGH** - GitHub OAuth with Passport.js is well-documented.

### File Handling Pattern

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         FILE UPLOAD FLOW                                │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐  │
│   │  1. Request arrives with multipart/form-data                     │  │
│   └────────────────────────────┬────────────────────────────────────┘  │
│                                │                                        │
│                                ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────┐  │
│   │  2. Multer middleware parses file                                │  │
│   │     - Validates file type (image/png, image/jpeg, image/gif)     │  │
│   │     - Enforces size limit (e.g., 5MB)                            │  │
│   │     - Stores to temp location                                    │  │
│   └────────────────────────────┬────────────────────────────────────┘  │
│                                │                                        │
│                                ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────┐  │
│   │  3. UploadService processes file                                 │  │
│   │     - Generates unique filename: {feedbackId}_{timestamp}.{ext}  │  │
│   │     - Moves to permanent storage (local or S3)                   │  │
│   │     - Returns storage path/URL                                   │  │
│   └────────────────────────────┬────────────────────────────────────┘  │
│                                │                                        │
│                                ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────┐  │
│   │  4. FeedbackService saves feedback with screenshot_path          │  │
│   └─────────────────────────────────────────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘

Storage abstraction for flexibility:

// services/upload.service.js
class UploadService {
  constructor(storageAdapter) {
    this.storage = storageAdapter;  // LocalStorage or S3Storage
  }

  async save(file, feedbackId) {
    const filename = `${feedbackId}_${Date.now()}${path.extname(file.originalname)}`;
    const storagePath = await this.storage.put(file.path, filename);
    return storagePath;
  }

  async getUrl(storagePath) {
    return this.storage.getUrl(storagePath);
  }

  async delete(storagePath) {
    return this.storage.delete(storagePath);
  }
}

// adapters/local-storage.js
class LocalStorage {
  constructor(basePath) {
    this.basePath = basePath;  // e.g., /app/uploads
  }

  async put(tempPath, filename) {
    const dest = path.join(this.basePath, filename);
    await fs.rename(tempPath, dest);
    return filename;  // Store relative path in DB
  }

  getUrl(filename) {
    return `/uploads/${filename}`;  // Served statically
  }
}

// adapters/s3-storage.js
class S3Storage {
  constructor(bucket, client) {
    this.bucket = bucket;
    this.client = client;
  }

  async put(tempPath, filename) {
    await this.client.putObject({
      Bucket: this.bucket,
      Key: `screenshots/${filename}`,
      Body: fs.createReadStream(tempPath)
    });
    return `screenshots/${filename}`;
  }

  getUrl(key) {
    return `https://${this.bucket}.s3.amazonaws.com/${key}`;
  }
}
```

**Recommendation:** Start with local storage, add S3 adapter later if needed.

**Confidence: HIGH** - Multer + storage abstraction is standard pattern.

---

## Data Flow Diagrams

### Feedback Submission Flow

```
┌───────────────────────────────────────────────────────────────────────────────┐
│                        FEEDBACK SUBMISSION DATA FLOW                          │
│                                                                               │
│  ┌──────────┐                                                                 │
│  │ Your App │                                                                 │
│  │ (Client) │                                                                 │
│  └────┬─────┘                                                                 │
│       │                                                                       │
│       │  POST /api/v1/feedback                                                │
│       │  {type, content, userId?, rating?, screenshot?}                       │
│       ▼                                                                       │
│  ┌─────────────────────────────────────────────────────────────────────────┐ │
│  │                        API GATEWAY                                       │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                   │ │
│  │  │ Rate Limiter │─>│ API Key Auth │─>│  Validator   │                   │ │
│  │  │  (optional)  │  │              │  │  (Joi/Zod)   │                   │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘                   │ │
│  └─────────────────────────────┬───────────────────────────────────────────┘ │
│                                │                                              │
│                                ▼                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐ │
│  │                     FEEDBACK SERVICE                                     │ │
│  │                                                                          │ │
│  │   1. Create feedback record (status: 'new', priority: 'medium')         │ │
│  │   2. If screenshot: call UploadService.save()                           │ │
│  │   3. Insert into database                                                │ │
│  │   4. Emit 'feedback:created' event                                       │ │
│  │                                                                          │ │
│  └─────────────────┬─────────────────────────────────┬─────────────────────┘ │
│                    │                                 │                        │
│                    ▼                                 ▼                        │
│  ┌──────────────────────────┐        ┌────────────────────────────────────┐  │
│  │        MariaDB           │        │     NOTIFICATION SERVICE           │  │
│  │  ┌────────────────────┐  │        │                                    │  │
│  │  │ INSERT INTO        │  │        │  1. Format email template          │  │
│  │  │ feedback (...)     │  │        │  2. Send via SMTP                  │  │
│  │  └────────────────────┘  │        │  3. Log notification sent          │  │
│  └──────────────────────────┘        └────────────────────────────────────┘  │
│                                                                               │
│       │                                                                       │
│       │  201 Created { id, status, createdAt }                               │
│       ▼                                                                       │
│  ┌──────────┐                                                                 │
│  │ Your App │                                                                 │
│  │ (Client) │                                                                 │
│  └──────────┘                                                                 │
│                                                                               │
└───────────────────────────────────────────────────────────────────────────────┘
```

### Admin Dashboard Flow

```
┌───────────────────────────────────────────────────────────────────────────────┐
│                         ADMIN DASHBOARD DATA FLOW                             │
│                                                                               │
│  ┌──────────────┐                                                             │
│  │    Admin     │                                                             │
│  │   Browser    │                                                             │
│  └──────┬───────┘                                                             │
│         │                                                                     │
│         │  1. Navigate to /admin                                              │
│         ▼                                                                     │
│  ┌──────────────────────────────────────────────────────────────────────────┐│
│  │  REACT SPA (served as static files from /admin or separate port)         ││
│  │                                                                           ││
│  │  ┌─────────────────────────────────────────────────────────────────────┐ ││
│  │  │  AuthContext checks session validity                                 │ ││
│  │  │                                                                      │ ││
│  │  │  If no session:                                                      │ ││
│  │  │    -> Redirect to /admin/auth/login                                  │ ││
│  │  │    -> GitHub OAuth flow                                              │ ││
│  │  │    -> Callback sets httpOnly session cookie                          │ ││
│  │  │    -> Redirect back to /admin                                        │ ││
│  │  └─────────────────────────────────────────────────────────────────────┘ ││
│  └──────────────────────────────────────────────────────────────────────────┘│
│         │                                                                     │
│         │  2. Authenticated: fetch feedback list                              │
│         │  GET /admin/api/feedback?status=open&page=1                         │
│         ▼                                                                     │
│  ┌──────────────────────────────────────────────────────────────────────────┐│
│  │  SESSION AUTH MIDDLEWARE                                                  ││
│  │    - Verify session cookie                                                ││
│  │    - Attach user to request                                               ││
│  └─────────────────────────────┬────────────────────────────────────────────┘│
│                                │                                              │
│                                ▼                                              │
│  ┌──────────────────────────────────────────────────────────────────────────┐│
│  │  ADMIN API ROUTES                                                         ││
│  │    - Query MariaDB with filters                                           ││
│  │    - Join tags, app info                                                  ││
│  │    - Paginate results                                                     ││
│  └─────────────────────────────┬────────────────────────────────────────────┘│
│                                │                                              │
│         │                      │                                              │
│         │  { data: [...], pagination: {...} }                                │
│         ▼                                                                     │
│  ┌──────────────────────────────────────────────────────────────────────────┐│
│  │  REACT DASHBOARD                                                          ││
│  │    - Render feedback list                                                 ││
│  │    - Filter/sort controls                                                 ││
│  │    - Click item -> detail view                                            ││
│  │    - Update priority/status/tags -> PATCH /admin/api/feedback/:id         ││
│  └──────────────────────────────────────────────────────────────────────────┘│
│                                                                               │
└───────────────────────────────────────────────────────────────────────────────┘
```

**Confidence: HIGH** - Standard SPA + REST API pattern.

---

## Database Schema Recommendations

```sql
-- ============================================================================
-- FEEDBACK COLLECTION SYSTEM - DATABASE SCHEMA
-- MariaDB / MySQL compatible
-- ============================================================================

-- -----------------------------------------------------------------------------
-- APPS: Registered applications that can submit feedback
-- -----------------------------------------------------------------------------
CREATE TABLE apps (
    id              VARCHAR(36) PRIMARY KEY,           -- UUID
    name            VARCHAR(100) NOT NULL,             -- "My Blog"
    domain          VARCHAR(255),                      -- "blog.example.com" (for reference)
    api_key         VARCHAR(64) NOT NULL UNIQUE,       -- "fb_live_xxxxxxxx..."
    is_active       BOOLEAN DEFAULT TRUE,
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    INDEX idx_api_key (api_key),
    INDEX idx_is_active (is_active)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- -----------------------------------------------------------------------------
-- FEEDBACK: Core feedback items submitted from apps
-- -----------------------------------------------------------------------------
CREATE TABLE feedback (
    id              VARCHAR(36) PRIMARY KEY,           -- UUID (e.g., "fb_abc123...")
    app_id          VARCHAR(36) NOT NULL,              -- Which app submitted this

    -- Content
    type            ENUM('bug', 'feature', 'feedback') NOT NULL DEFAULT 'feedback',
    content         TEXT NOT NULL,                     -- Main feedback text
    rating          TINYINT UNSIGNED,                  -- 1-5 satisfaction score (nullable)

    -- Source user (from host app, not our user)
    user_id         VARCHAR(255),                      -- External user ID from host app
    user_email      VARCHAR(255),                      -- Optional email if provided

    -- Management fields (set by admin)
    status          ENUM('new', 'open', 'in_progress', 'resolved', 'closed')
                    NOT NULL DEFAULT 'new',
    priority        ENUM('low', 'medium', 'high') NOT NULL DEFAULT 'medium',
    admin_notes     TEXT,                              -- Private notes from admin

    -- File attachment
    screenshot_path VARCHAR(500),                      -- Path/key to uploaded screenshot

    -- Metadata (arbitrary JSON from client)
    metadata        JSON,                              -- { "page": "/settings", "browser": "..." }

    -- Timestamps
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    resolved_at     TIMESTAMP NULL,

    FOREIGN KEY (app_id) REFERENCES apps(id) ON DELETE CASCADE,

    INDEX idx_app_id (app_id),
    INDEX idx_type (type),
    INDEX idx_status (status),
    INDEX idx_priority (priority),
    INDEX idx_created_at (created_at),
    INDEX idx_app_status (app_id, status),            -- Common filter combo
    INDEX idx_app_type_status (app_id, type, status)  -- Dashboard queries
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- -----------------------------------------------------------------------------
-- TAGS: Labels for categorizing feedback
-- -----------------------------------------------------------------------------
CREATE TABLE tags (
    id              VARCHAR(36) PRIMARY KEY,           -- UUID
    name            VARCHAR(50) NOT NULL UNIQUE,       -- "ui-bug", "performance"
    color           VARCHAR(7) DEFAULT '#6B7280',      -- Hex color for UI
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_name (name)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- -----------------------------------------------------------------------------
-- FEEDBACK_TAGS: Many-to-many relationship
-- -----------------------------------------------------------------------------
CREATE TABLE feedback_tags (
    feedback_id     VARCHAR(36) NOT NULL,
    tag_id          VARCHAR(36) NOT NULL,
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY (feedback_id, tag_id),
    FOREIGN KEY (feedback_id) REFERENCES feedback(id) ON DELETE CASCADE,
    FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE,

    INDEX idx_tag_id (tag_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- -----------------------------------------------------------------------------
-- SESSIONS: Express-session store (if using database sessions)
-- -----------------------------------------------------------------------------
CREATE TABLE sessions (
    session_id      VARCHAR(128) PRIMARY KEY,
    expires         INT UNSIGNED NOT NULL,
    data            MEDIUMTEXT,

    INDEX idx_expires (expires)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- -----------------------------------------------------------------------------
-- NOTIFICATIONS_LOG: Track sent notifications (optional, for debugging)
-- -----------------------------------------------------------------------------
CREATE TABLE notifications_log (
    id              VARCHAR(36) PRIMARY KEY,
    feedback_id     VARCHAR(36) NOT NULL,
    type            ENUM('email') DEFAULT 'email',
    recipient       VARCHAR(255) NOT NULL,
    subject         VARCHAR(255),
    sent_at         TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status          ENUM('sent', 'failed') DEFAULT 'sent',
    error_message   TEXT,

    FOREIGN KEY (feedback_id) REFERENCES feedback(id) ON DELETE CASCADE,
    INDEX idx_feedback_id (feedback_id),
    INDEX idx_sent_at (sent_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Schema Design Rationale

| Decision | Rationale |
|----------|-----------|
| UUID primary keys | URL-friendly, no auto-increment exposure, easier for API responses |
| VARCHAR(36) for IDs | Standard UUID length, works well with MariaDB |
| ENUM for status/type | Database-enforced constraints, clear valid values |
| JSON for metadata | Flexible client-provided data without schema changes |
| Soft reference for user | External user ID is just a string - not our user system |
| Separate tags table | Reusable across feedback, easy to manage colors |
| Notification log | Debugging email issues, optional but helpful |

**Confidence: HIGH** - Schema covers all requirements with room for extension.

### Common Queries

```sql
-- Dashboard: List feedback with filters and pagination
SELECT
    f.*,
    a.name as app_name,
    GROUP_CONCAT(t.name) as tag_names
FROM feedback f
JOIN apps a ON f.app_id = a.id
LEFT JOIN feedback_tags ft ON f.id = ft.feedback_id
LEFT JOIN tags t ON ft.tag_id = t.id
WHERE f.status IN ('new', 'open')
  AND f.app_id = ?
GROUP BY f.id
ORDER BY
    CASE f.priority
        WHEN 'high' THEN 1
        WHEN 'medium' THEN 2
        WHEN 'low' THEN 3
    END,
    f.created_at DESC
LIMIT 25 OFFSET 0;

-- Stats for dashboard header
SELECT
    COUNT(*) as total,
    SUM(CASE WHEN status = 'new' THEN 1 ELSE 0 END) as new_count,
    SUM(CASE WHEN status = 'open' THEN 1 ELSE 0 END) as open_count,
    SUM(CASE WHEN priority = 'high' AND status NOT IN ('resolved', 'closed') THEN 1 ELSE 0 END) as high_priority
FROM feedback
WHERE app_id = ?;
```

---

## Scaling Considerations

For a personal/single-admin tool, scaling concerns are minimal. Here's a realistic assessment:

### What NOT to Over-Engineer

| Pattern | Skip For Now | Add If/When |
|---------|--------------|-------------|
| Database sharding | Skip | Never (for personal tool) |
| Read replicas | Skip | Never |
| Message queues (Redis/RabbitMQ) | Skip | Very high feedback volume |
| Microservices | Skip | Never |
| Kubernetes | Skip | Already using Docker is fine |
| CDN for screenshots | Skip | Hundreds of daily uploads |
| Full-text search (Elasticsearch) | Skip | Thousands of feedback items + search needs |
| Caching layer (Redis) | Skip | Dashboard feels slow |

### Practical Scaling Steps (in order)

1. **Current architecture handles:** ~10,000 feedback items, ~100/day submissions

2. **If dashboard gets slow:**
   - Add database indexes (already in schema above)
   - Add simple in-memory cache for stats queries (5-minute TTL)

3. **If file storage fills up:**
   - Switch from local to S3 (storage adapter pattern makes this easy)
   - Add cleanup job for old resolved feedback screenshots

4. **If email sending is slow:**
   - Move to async (simple setTimeout or setImmediate)
   - Later: Add a simple job queue (better-queue or similar)

**Confidence: HIGH** - Don't optimize prematurely for a personal tool.

---

## Anti-Patterns to Avoid

### 1. Over-Abstraction

```javascript
// BAD: Don't do this for a simple tool
class FeedbackRepositoryFactory {
  createRepository(type) {
    switch(type) {
      case 'mysql': return new MySQLFeedbackRepository();
      case 'postgres': return new PostgresFeedbackRepository();
      case 'mongo': return new MongoFeedbackRepository();
    }
  }
}

// GOOD: Just use MariaDB directly
class FeedbackRepository {
  constructor(db) {
    this.db = db;  // MariaDB connection
  }

  async findById(id) {
    const [rows] = await this.db.query('SELECT * FROM feedback WHERE id = ?', [id]);
    return rows[0];
  }
}
```

### 2. Premature Async Complexity

```javascript
// BAD: Don't add message queues for simple notifications
import { Queue } from 'bull';
const emailQueue = new Queue('email-notifications', redisConfig);

// GOOD: Simple async function is fine
async function sendNotification(feedback) {
  // Fire and forget - don't await if you don't need to
  sendEmail(feedback).catch(err => logger.error('Email failed:', err));
}
```

### 3. Overly Generic API

```javascript
// BAD: GraphQL for a simple CRUD app with one user
app.use('/graphql', graphqlHTTP({
  schema: buildSchema(`...`)
}));

// GOOD: Simple REST endpoints
router.get('/feedback', listFeedback);
router.get('/feedback/:id', getFeedback);
router.patch('/feedback/:id', updateFeedback);
```

### 4. Complex State Management

```javascript
// BAD: Redux + middleware for simple admin dashboard
const store = createStore(
  rootReducer,
  applyMiddleware(thunk, logger, analytics)
);

// GOOD: React Query or simple useState/useReducer
function useFeedback() {
  return useQuery('feedback', () => api.getFeedback());
}
```

### 5. Multiple Auth Strategies When One Suffices

```javascript
// BAD: Supporting every OAuth provider
passport.use(new GitHubStrategy(...));
passport.use(new GoogleStrategy(...));
passport.use(new TwitterStrategy(...));
passport.use(new LocalStrategy(...));

// GOOD: Just GitHub OAuth for single admin
passport.use(new GitHubStrategy({
  clientID: config.github.clientId,
  clientSecret: config.github.clientSecret
}, verifyCallback));
```

### 6. Microservices for Monolith Problems

```
BAD:
┌────────────┐   ┌────────────┐   ┌────────────┐
│  Feedback  │   │   Email    │   │    File    │
│  Service   │◄─►│  Service   │◄─►│  Service   │
└────────────┘   └────────────┘   └────────────┘
      ▲               ▲               ▲
      └───────────────┼───────────────┘
                      │
              ┌──────────────┐
              │  API Gateway │
              └──────────────┘

GOOD:
┌───────────────────────────────────────────┐
│              Single Node.js App           │
│  ┌─────────┐ ┌─────────┐ ┌─────────────┐ │
│  │Feedback │ │  Email  │ │    File     │ │
│  │ Module  │ │ Module  │ │   Module    │ │
│  └─────────┘ └─────────┘ └─────────────┘ │
└───────────────────────────────────────────┘
```

**Confidence: HIGH** - These are common pitfalls for developer tools.

---

## Integration Points

### 1. GitHub OAuth Integration

```javascript
// config/auth.js
module.exports = {
  github: {
    clientId: process.env.GITHUB_CLIENT_ID,
    clientSecret: process.env.GITHUB_CLIENT_SECRET,
    callbackURL: process.env.GITHUB_CALLBACK_URL || '/admin/auth/callback',
    scope: ['read:user']  // Minimal scope needed
  },
  allowedAdminId: process.env.ADMIN_GITHUB_ID,
  session: {
    secret: process.env.SESSION_SECRET,
    maxAge: 7 * 24 * 60 * 60 * 1000  // 7 days
  }
};

// services/auth.service.js
const passport = require('passport');
const GitHubStrategy = require('passport-github2').Strategy;

passport.use(new GitHubStrategy({
    clientID: config.github.clientId,
    clientSecret: config.github.clientSecret,
    callbackURL: config.github.callbackURL
  },
  async (accessToken, refreshToken, profile, done) => {
    // Only allow the configured admin
    if (profile.id !== config.allowedAdminId) {
      return done(null, false, { message: 'Not authorized' });
    }

    // Return minimal user object
    return done(null, {
      id: profile.id,
      username: profile.username,
      displayName: profile.displayName,
      avatarUrl: profile.photos?.[0]?.value
    });
  }
));

// Serialize just the GitHub ID
passport.serializeUser((user, done) => done(null, user.id));
passport.deserializeUser((id, done) => {
  // For single admin, just return the ID
  done(null, { id });
});
```

**GitHub App Setup:**
1. Go to GitHub Settings > Developer Settings > OAuth Apps
2. Create new OAuth App
3. Set callback URL to `https://yourdomain.com/admin/auth/callback`
4. Copy Client ID and Client Secret to `.env`
5. Get your GitHub user ID: `curl https://api.github.com/users/YOUR_USERNAME`

### 2. SMTP Email Integration

```javascript
// config/email.js
module.exports = {
  smtp: {
    host: process.env.SMTP_HOST,
    port: parseInt(process.env.SMTP_PORT, 10) || 587,
    secure: process.env.SMTP_SECURE === 'true',  // true for 465, false for others
    auth: {
      user: process.env.SMTP_USER,
      pass: process.env.SMTP_PASS
    }
  },
  from: process.env.EMAIL_FROM || 'Feedback App <feedback@example.com>',
  adminEmail: process.env.ADMIN_EMAIL
};

// services/notification.service.js
const nodemailer = require('nodemailer');

class NotificationService {
  constructor(config) {
    this.config = config;
    this.transporter = nodemailer.createTransport(config.smtp);
  }

  async sendNewFeedbackNotification(feedback, app) {
    const subject = `[${app.name}] New ${feedback.type}: ${this.truncate(feedback.content, 50)}`;

    const html = `
      <h2>New Feedback Received</h2>
      <p><strong>App:</strong> ${app.name}</p>
      <p><strong>Type:</strong> ${feedback.type}</p>
      <p><strong>Priority:</strong> ${feedback.priority}</p>
      ${feedback.rating ? `<p><strong>Rating:</strong> ${feedback.rating}/5</p>` : ''}
      ${feedback.userId ? `<p><strong>User ID:</strong> ${feedback.userId}</p>` : ''}
      <hr>
      <p>${feedback.content}</p>
      <hr>
      <p><a href="${process.env.APP_URL}/admin/feedback/${feedback.id}">View in Dashboard</a></p>
    `;

    try {
      await this.transporter.sendMail({
        from: this.config.from,
        to: this.config.adminEmail,
        subject,
        html
      });
      return { success: true };
    } catch (error) {
      console.error('Email send failed:', error);
      return { success: false, error: error.message };
    }
  }

  truncate(str, length) {
    return str.length > length ? str.substring(0, length) + '...' : str;
  }
}
```

**Email Configuration Options:**

| Provider | SMTP Host | Port | Notes |
|----------|-----------|------|-------|
| Gmail | smtp.gmail.com | 587 | Needs App Password |
| Mailgun | smtp.mailgun.org | 587 | Free tier available |
| SendGrid | smtp.sendgrid.net | 587 | Free tier available |
| Self-hosted | Your server | 25/587 | postfix/exim |
| Mailtrap | smtp.mailtrap.io | 587 | For testing |

### 3. File Storage Integration

```javascript
// config/storage.js
module.exports = {
  type: process.env.STORAGE_TYPE || 'local',  // 'local' or 's3'

  local: {
    uploadDir: process.env.UPLOAD_DIR || './uploads',
    serveUrl: '/uploads'  // Static file serving path
  },

  s3: {
    bucket: process.env.S3_BUCKET,
    region: process.env.S3_REGION || 'us-east-1',
    accessKeyId: process.env.S3_ACCESS_KEY_ID,
    secretAccessKey: process.env.S3_SECRET_ACCESS_KEY,
    endpoint: process.env.S3_ENDPOINT  // For S3-compatible (MinIO, DigitalOcean Spaces)
  }
};

// services/upload.service.js
const multer = require('multer');
const path = require('path');
const { v4: uuidv4 } = require('uuid');

// Multer config
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, config.storage.local.uploadDir);
  },
  filename: (req, file, cb) => {
    const ext = path.extname(file.originalname);
    cb(null, `${uuidv4()}${ext}`);
  }
});

const fileFilter = (req, file, cb) => {
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];
  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error('Invalid file type. Only images allowed.'), false);
  }
};

const upload = multer({
  storage,
  fileFilter,
  limits: {
    fileSize: 5 * 1024 * 1024  // 5MB
  }
});

module.exports = { upload };
```

**Confidence: HIGH** - Standard integration patterns.

---

## Environment Configuration

```bash
# .env.example

# =============================================================================
# APPLICATION
# =============================================================================
NODE_ENV=production
PORT=3000
APP_URL=https://feedback.yourdomain.com

# =============================================================================
# DATABASE (MariaDB)
# =============================================================================
DB_HOST=localhost
DB_PORT=3306
DB_NAME=feedback_app
DB_USER=feedback_user
DB_PASSWORD=your_secure_password

# =============================================================================
# AUTHENTICATION (GitHub OAuth)
# =============================================================================
GITHUB_CLIENT_ID=your_github_client_id
GITHUB_CLIENT_SECRET=your_github_client_secret
GITHUB_CALLBACK_URL=https://feedback.yourdomain.com/admin/auth/callback

# Your GitHub numeric user ID (get from: curl https://api.github.com/users/YOUR_USERNAME)
ADMIN_GITHUB_ID=12345678

# Session secret (generate with: openssl rand -base64 32)
SESSION_SECRET=your_session_secret_here

# =============================================================================
# EMAIL (SMTP)
# =============================================================================
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_SECURE=false
SMTP_USER=your_smtp_user
SMTP_PASS=your_smtp_password
EMAIL_FROM=Feedback App <feedback@yourdomain.com>
ADMIN_EMAIL=you@example.com

# =============================================================================
# FILE STORAGE
# =============================================================================
STORAGE_TYPE=local
UPLOAD_DIR=./uploads

# For S3/S3-compatible storage (optional)
# S3_BUCKET=your-bucket
# S3_REGION=us-east-1
# S3_ACCESS_KEY_ID=your_access_key
# S3_SECRET_ACCESS_KEY=your_secret_key
# S3_ENDPOINT=https://s3.amazonaws.com

# =============================================================================
# SECURITY
# =============================================================================
# CORS origins for API (comma-separated)
CORS_ORIGINS=https://app1.yourdomain.com,https://app2.yourdomain.com

# Rate limiting
RATE_LIMIT_WINDOW_MS=60000
RATE_LIMIT_MAX_REQUESTS=100
```

---

## Summary

### Key Architectural Decisions

| Decision | Choice | Confidence |
|----------|--------|------------|
| API structure | REST with versioning (/api/v1/) | HIGH |
| Auth pattern | API keys for submission, GitHub OAuth for admin | HIGH |
| Database | MariaDB with standard relational schema | HIGH |
| File storage | Local with S3 abstraction for future | HIGH |
| Notifications | Direct SMTP, no queue | HIGH |
| Frontend | React SPA with simple state management | HIGH |
| Deployment | Single Docker container (or docker-compose) | HIGH |

### Implementation Priority

1. **Phase 1 - Core API**
   - Database schema + migrations
   - App registration + API key generation
   - Feedback submission endpoint
   - Basic validation

2. **Phase 2 - Admin Foundation**
   - GitHub OAuth setup
   - Session management
   - Basic admin API (list/view feedback)

3. **Phase 3 - Dashboard**
   - React admin UI
   - Feedback list with filters
   - Detail view with editing

4. **Phase 4 - Polish**
   - File upload support
   - Email notifications
   - Tags and priority management
   - Docker deployment config

---

*Research completed: 2026-01-15*
*Confidence assessment: Overall HIGH - patterns are well-established for this type of tool*
