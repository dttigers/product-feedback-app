# Feedback Collection Pitfalls Research

> Common mistakes, failure patterns, and prevention strategies for self-hosted feedback tools with Node.js, MariaDB, GitHub OAuth, SMTP, and file uploads.

**Research Method:** Knowledge synthesis from documented post-mortems, community discussions, and experienced developer warnings. WebSearch unavailable during research.

**Confidence Scale:**
- **HIGH** - Well-documented, frequently reported, personally witnessed in multiple projects
- **MEDIUM** - Commonly discussed, logical failure mode, some documented cases
- **LOW** - Edge case but worth mentioning, anecdotal reports

---

## Critical Pitfalls (Top 7)

### 1. File Upload Path Traversal & Type Bypass

**What Goes Wrong:**
Attackers upload files with manipulated names like `../../../etc/passwd` or disguised executables (`malware.jpg.exe`). Screenshot uploads become a vector for server compromise or serving malicious content to other users.

**Why It Happens:**
- Trusting user-provided filenames
- Only checking file extension, not MIME type or magic bytes
- Storing uploads in web-accessible directories
- Not sanitizing paths before file operations

**How to Avoid:**
```javascript
// NEVER use original filename directly
const safeFilename = crypto.randomUUID() + '.png';

// Validate MIME type AND magic bytes
const fileType = await fileTypeFromBuffer(buffer);
if (!['image/png', 'image/jpeg', 'image/gif'].includes(fileType?.mime)) {
  throw new Error('Invalid file type');
}

// Store outside web root, serve through API with Content-Disposition
app.get('/api/screenshots/:id', async (req, res) => {
  res.setHeader('Content-Disposition', 'inline');
  res.setHeader('Content-Type', 'image/png');
  res.setHeader('X-Content-Type-Options', 'nosniff');
  // Stream from storage...
});
```

**Warning Signs:**
- Using `req.files[0].originalname` anywhere
- Uploads accessible via direct URL to filesystem
- No file type validation in upload handler

**Phase to Address:** Phase 1 (API endpoints) - Build it right from the start

**Confidence:** HIGH - This is OWASP Top 10, extensively documented

---

### 2. GitHub OAuth State Parameter Missing/Weak

**What Goes Wrong:**
CSRF attacks during OAuth flow. Attacker initiates OAuth, captures redirect URL, tricks victim into completing flow, gains admin access to victim's session.

**Why It Happens:**
- Skipping `state` parameter because "it works without it"
- Using predictable state values
- Not validating state on callback
- Storing state in easily-accessible client-side storage

**How to Avoid:**
```javascript
// Generate cryptographically secure state
const state = crypto.randomBytes(32).toString('hex');
req.session.oauthState = state;

// Include in authorization URL
const authUrl = `https://github.com/login/oauth/authorize?client_id=${CLIENT_ID}&state=${state}&redirect_uri=${CALLBACK_URL}`;

// STRICTLY validate on callback
app.get('/auth/github/callback', (req, res) => {
  if (req.query.state !== req.session.oauthState) {
    return res.status(403).send('State mismatch - possible CSRF');
  }
  delete req.session.oauthState;
  // Continue with token exchange...
});
```

**Warning Signs:**
- OAuth URL doesn't include `state` parameter
- State is simple incrementing number or timestamp
- Callback handler doesn't check state first

**Phase to Address:** Phase 2 (Authentication) - Critical security, no shortcuts

**Confidence:** HIGH - GitHub's own docs emphasize this, many documented attacks

---

### 3. SMTP Connection Pool Exhaustion

**What Goes Wrong:**
Email notifications stop working under load or after running for days. Server logs show "ECONNRESET" or timeouts. New feedback submissions succeed but no emails go out.

**Why It Happens:**
- Creating new SMTP transport for every email
- Not handling connection errors/retries
- SMTP server limits concurrent connections
- Nodemailer connections timing out without reconnect

**How to Avoid:**
```javascript
// Single transporter instance with connection pooling
const transporter = nodemailer.createTransport({
  host: process.env.SMTP_HOST,
  port: 587,
  pool: true,           // Enable pooling
  maxConnections: 5,    // Limit concurrent
  maxMessages: 100,     // Messages per connection before reconnect
  rateDelta: 1000,      // Rate limit: 1 per second
  rateLimit: 10,        // Max 10 per rateDelta
});

// Verify connection on startup
transporter.verify((error) => {
  if (error) {
    console.error('SMTP connection failed:', error);
    // Alert admin, don't crash
  }
});

// Queue emails instead of sending directly
async function queueNotification(feedback) {
  await db.insert('email_queue', { feedback_id: feedback.id, status: 'pending' });
}

// Background worker processes queue with retries
```

**Warning Signs:**
- `createTransport()` called inside request handler
- No error handling around `sendMail()`
- Emails work in dev, fail in production
- Memory usage grows over time

**Phase to Address:** Phase 4 (Notifications) - Use queue pattern from start

**Confidence:** HIGH - Extremely common in production Node.js apps

---

### 4. MariaDB Connection Pool Leak

**What Goes Wrong:**
After hours/days of operation, database queries start timing out. "Too many connections" errors. App becomes unresponsive, requires restart.

**Why It Happens:**
- Not releasing connections after use
- Error paths skip connection release
- Transactions left open on errors
- Connection pool too small for concurrent requests

**How to Avoid:**
```javascript
// Use pool.query() for simple queries - auto-releases
await pool.query('SELECT * FROM feedback WHERE id = ?', [id]);

// For transactions, ALWAYS use try/finally
async function updateWithTransaction(feedbackId, data) {
  const conn = await pool.getConnection();
  try {
    await conn.beginTransaction();
    await conn.query('UPDATE feedback SET ...', [...]);
    await conn.query('INSERT INTO audit_log ...', [...]);
    await conn.commit();
  } catch (error) {
    await conn.rollback();
    throw error;
  } finally {
    conn.release(); // ALWAYS release, even on error
  }
}

// Monitor pool health
setInterval(() => {
  console.log('Pool stats:', {
    active: pool.activeConnections(),
    idle: pool.idleConnections(),
    total: pool.totalConnections()
  });
}, 60000);
```

**Warning Signs:**
- `getConnection()` without corresponding `release()`
- No `finally` block in database transaction code
- Works fine in testing, fails after hours in production
- Memory/connection count grows steadily

**Phase to Address:** Phase 1 (Database setup) - Establish patterns early

**Confidence:** HIGH - Classic Node.js database pitfall

---

### 5. Feedback Submission API Without Rate Limiting

**What Goes Wrong:**
Bot spams thousands of feedback submissions. Database fills up, email notifications flood inbox, legitimate feedback buried. Potential DoS vector.

**Why It Happens:**
- "It's internal tooling, who would attack it?"
- Rate limiting deferred for "later"
- API exposed without any authentication
- No per-IP or per-user-id tracking

**How to Avoid:**
```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');

// Per-IP rate limit
const submitLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 10, // 10 submissions per window
  message: { error: 'Too many submissions, try again later' },
  standardHeaders: true,
  // For production, use Redis store for distributed limiting
});

// Additional per-app-id limiting
const appLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 100, // 100 per app per hour
  keyGenerator: (req) => req.body.app_id || req.ip,
});

app.post('/api/feedback', submitLimiter, appLimiter, feedbackHandler);

// Also: validate app_id against registered apps
```

**Warning Signs:**
- No middleware before feedback POST handler
- Feedback table growing unexpectedly
- Email notifications overwhelming inbox
- No app_id validation against whitelist

**Phase to Address:** Phase 1 (API endpoints) - Security from day one

**Confidence:** HIGH - Every public-ish API gets probed

---

### 6. Screenshot Storage Filling Disk Silently

**What Goes Wrong:**
Server disk fills up, app crashes or becomes read-only. No warning until catastrophic failure. Often discovered when DB writes fail.

**Why It Happens:**
- No file size limits on upload
- No total storage quota
- No cleanup of orphaned files
- Logs and uploads on same partition
- No monitoring/alerting on disk space

**How to Avoid:**
```javascript
// Enforce file size limit
const multer = require('multer');
const upload = multer({
  limits: {
    fileSize: 5 * 1024 * 1024, // 5MB max
    files: 1
  },
  fileFilter: (req, file, cb) => {
    // Type checking here too
  }
});

// Track storage usage in database
await db.query(`
  INSERT INTO storage_usage (date, bytes_used)
  SELECT CURDATE(), SUM(file_size) FROM screenshots
  ON DUPLICATE KEY UPDATE bytes_used = VALUES(bytes_used)
`);

// Alert when approaching limit
const STORAGE_LIMIT = 10 * 1024 * 1024 * 1024; // 10GB
if (currentUsage > STORAGE_LIMIT * 0.8) {
  sendAdminAlert('Storage 80% full');
}

// Cleanup job for deleted feedback
// DELETE FROM screenshots WHERE feedback_id NOT IN (SELECT id FROM feedback)
```

**Warning Signs:**
- No `limits` in multer config
- No disk space monitoring
- Files stored but never cleaned up
- Docker volume not sized appropriately

**Phase to Address:** Phase 1 (File uploads) + Phase 5 (Production hardening)

**Confidence:** HIGH - Classic ops failure mode

---

### 7. Session Secret Hardcoded or Weak

**What Goes Wrong:**
Session tokens can be forged. Attacker gains admin access without OAuth. All sessions invalidated on redeploy if using random secret without persistence.

**Why It Happens:**
- Copy-paste from tutorial with example secret
- `process.env.SESSION_SECRET || 'keyboard cat'` pattern
- Generating random secret on each server start
- Not understanding what session secret does

**How to Avoid:**
```javascript
// Generate proper secret once, store securely
// openssl rand -hex 64

// In code - FAIL if missing, never fallback
if (!process.env.SESSION_SECRET || process.env.SESSION_SECRET.length < 64) {
  console.error('SESSION_SECRET must be set and be at least 64 characters');
  process.exit(1);
}

app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    sameSite: 'strict',
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
  }
}));
```

**Warning Signs:**
- Secret visible in code repository
- Secret less than 32 characters
- Fallback default in code
- Sessions break after redeploy

**Phase to Address:** Phase 2 (Authentication) - Security foundation

**Confidence:** HIGH - Extremely common, often undetected until breach

---

## Technical Debt Patterns

### "Just Make It Work" Debt

| Pattern | Short-term | Long-term Pain | Prevention |
|---------|------------|----------------|------------|
| Inline SQL everywhere | Fast to write | Impossible to maintain | Use query builder/ORM from start |
| No input validation | Fewer lines of code | Security holes, bad data | Zod/Joi schemas on all inputs |
| Synchronous file ops | Simpler code | Blocked event loop | Always use async fs operations |
| Console.log debugging | Quick feedback | Log noise in production | Structured logging from day one |
| No database migrations | Faster schema changes | Can't recreate prod DB | Knex/Prisma migrations mandatory |

### Accumulating Debt Signs
- "TODO" comments older than 2 weeks
- Duplicate code for similar operations
- Functions over 50 lines
- Files over 500 lines
- No tests for critical paths (auth, payments, data mutation)

**Confidence:** HIGH

---

## Integration Gotchas

### GitHub OAuth Specific

| Gotcha | Problem | Solution | Confidence |
|--------|---------|----------|------------|
| Token expiration | GitHub tokens don't expire by default, but refresh tokens do | Store token expiry, implement refresh flow | MEDIUM |
| Scope creep | Requesting too many scopes reduces user trust | Request only `read:user` for basic auth | HIGH |
| Rate limiting | 5000 requests/hour per user token | Cache user info, don't fetch on every request | HIGH |
| Org membership checking | App works, then user leaves org | Re-verify on sensitive operations | MEDIUM |
| Callback URL mismatch | Works locally, fails in prod | Exact match required, check trailing slashes | HIGH |
| Client secret exposure | Secret in frontend code | NEVER expose, server-side token exchange only | HIGH |

### SMTP/Nodemailer Specific

| Gotcha | Problem | Solution | Confidence |
|--------|---------|----------|------------|
| Gmail auth | "Less secure apps" deprecated | Use OAuth2 or app-specific password | HIGH |
| TLS certificate errors | Self-signed certs in Docker | `tls: { rejectUnauthorized: false }` only in dev | HIGH |
| HTML email rendering | Looks good in Gmail, broken in Outlook | Use MJML or email-safe HTML, test multiple clients | HIGH |
| Attachment size | Large screenshots bounce | Compress images, consider links instead | MEDIUM |
| SPF/DKIM/DMARC | Emails go to spam | Configure DNS records, or use relay service | HIGH |
| Connection refused | SMTP port blocked by cloud provider | Use port 587, check firewall rules | HIGH |

### MariaDB/Node.js Specific

| Gotcha | Problem | Solution | Confidence |
|--------|---------|----------|------------|
| Timezone confusion | Dates wrong by hours | Set `timezone: 'Z'` in connection, store UTC | HIGH |
| BigInt handling | JavaScript Number overflow for large IDs | Configure `supportBigNumbers: true` | MEDIUM |
| Prepared statement caching | Memory growth | Limit cache size or use simple queries | MEDIUM |
| Character set | Emojis in feedback truncated/corrupted | `charset: 'utf8mb4'` on connection AND tables | HIGH |
| Connection timeout | Long-running app loses connection | `waitForConnections: true`, reconnection logic | HIGH |
| SQL injection via IN clause | `WHERE id IN (${ids})` | Use parameterized queries, even for IN | HIGH |

---

## Performance Traps

### Database Performance

| Trap | Symptom | Prevention | Phase |
|------|---------|------------|-------|
| Missing indexes on query columns | Slow list views | Add indexes on `app_id`, `status`, `created_at`, `priority` | Phase 1 |
| N+1 queries for tags | Feedback list makes 100s of queries | JOIN or batch load tags | Phase 3 |
| Full table scans for search | Search times out | Consider full-text index or limit results | Phase 3 |
| No pagination | List view crashes browser | Mandatory offset/limit from day one | Phase 1 |
| Large BLOB in main table | All queries slow | Store screenshots in separate table/storage | Phase 1 |

### API Performance

| Trap | Symptom | Prevention | Phase |
|------|---------|------------|-------|
| Sync email sending | Feedback submission takes 3+ seconds | Queue emails, respond immediately | Phase 4 |
| No response compression | Large payloads slow on mobile | Enable gzip middleware | Phase 1 |
| Unbounded queries | Admin dashboard times out | Always limit + paginate | Phase 3 |
| No caching of static data | Repeated DB hits for app list | Cache reference data with TTL | Phase 3 |

**Confidence:** HIGH - Standard performance patterns

---

## Security Mistakes (Feedback System Specific)

### Input Handling

| Mistake | Attack Vector | Prevention |
|---------|---------------|------------|
| No XSS sanitization on feedback display | Stored XSS in admin dashboard | Sanitize on input AND escape on output |
| Feedback content in notification email | HTML injection in email | Plain text emails or strict sanitization |
| User ID trusted from client | Submit feedback as any user | Validate user ID against session/token if available |
| Direct object reference | `/api/feedback/123` accessible to anyone | App-specific API keys, validate ownership |

### Authentication & Authorization

| Mistake | Attack Vector | Prevention |
|---------|---------------|------------|
| Auth check only on frontend | Direct API access bypasses auth | Middleware auth check on ALL admin routes |
| Session fixation | Attacker sets session ID pre-auth | Regenerate session ID after successful login |
| No logout invalidation | Stolen session works forever | Clear session server-side on logout |
| API key in URL | Keys logged in access logs | Use Authorization header |

### File Upload Security

| Mistake | Attack Vector | Prevention |
|---------|---------------|------------|
| Execute uploads | PHP/JS file executed by server | Store outside webroot, serve through API |
| No virus scanning | Malware distribution | ClamAV scan or disable high-risk types |
| Image metadata preserved | EXIF location data exposed | Strip metadata with sharp/imagemagick |
| Predictable URLs | Enumerate all screenshots | Random UUIDs, signed URLs if sensitive |

**Confidence:** HIGH

---

## UX Pitfalls

### Feedback Submission

| Pitfall | User Impact | Prevention |
|---------|-------------|------------|
| No submission confirmation | Users submit multiple times | Clear success state, disable button |
| Lost form state on error | Frustrated users abandon | Preserve form data, show specific errors |
| Large screenshot upload blocks UI | Appears frozen | Progress indicator, background upload |
| No character limit indication | Truncated feedback surprises | Show limit and count |

### Admin Dashboard

| Pitfall | User Impact | Prevention |
|---------|-------------|------------|
| No loading states | Clicks don't seem to work | Skeleton loaders, button disabled states |
| Bulk actions without confirmation | Accidental mass delete | Confirm destructive actions |
| Filter state lost on navigation | Repetitive filtering | URL params or local storage for filters |
| No feedback preview | Have to open each item | Expandable rows or preview pane |
| Notification settings hidden | Can't turn off email flood | Prominent settings in nav |

**Confidence:** MEDIUM - Somewhat subjective

---

## "Looks Done But Isn't" Checklist

Before considering any phase complete, verify these often-forgotten items:

### Authentication (Phase 2)
- [ ] OAuth callback validates state parameter
- [ ] Session regenerates after login
- [ ] Logout clears server-side session
- [ ] Session cookie has secure, httpOnly, sameSite flags
- [ ] Failed login attempts are rate-limited
- [ ] Admin-only routes have middleware protection

### File Uploads (Phase 1)
- [ ] File size limit enforced server-side
- [ ] File type validated by magic bytes, not just extension
- [ ] Filename is regenerated, never user-provided
- [ ] Files stored outside web root
- [ ] Content-Type and Content-Disposition headers set on download
- [ ] Old/orphaned files cleaned up

### Email Notifications (Phase 4)
- [ ] SMTP connection is pooled, not per-request
- [ ] Emails queued, not sent synchronously
- [ ] Failed sends have retry mechanism
- [ ] Admin can disable notifications
- [ ] Unsubscribe works (even for single user)
- [ ] Email content sanitized against injection

### Database (Phase 1)
- [ ] All tables have appropriate indexes
- [ ] Connection pool is configured with limits
- [ ] Connections released in all error paths
- [ ] Charset is utf8mb4 everywhere
- [ ] Migrations exist for all schema changes
- [ ] Transactions used for multi-step operations

### API Security (Phase 1)
- [ ] Rate limiting on all public endpoints
- [ ] Input validation on all endpoints (Zod/Joi)
- [ ] SQL injection prevented (parameterized queries)
- [ ] XSS prevention on output
- [ ] CORS configured correctly (not `*` in production)
- [ ] Error messages don't leak internal details

### Production Readiness (Phase 5)
- [ ] Health check endpoint exists
- [ ] Structured logging (JSON, not console.log)
- [ ] Graceful shutdown handling
- [ ] Environment variables documented
- [ ] Secrets not in code or logs
- [ ] Disk space monitoring in place

---

## Recovery Strategies

### When Things Go Wrong

| Failure | Detection | Immediate Action | Prevention |
|---------|-----------|------------------|------------|
| Database connection exhausted | "Too many connections" error | Restart app, increase pool limit | Connection pool monitoring |
| SMTP stopped working | No notifications for hours | Check SMTP logs, verify credentials | Health check pings |
| Disk full | Write errors | Clear logs, expand storage | Monitoring + alerts at 80% |
| OAuth token invalid | Login fails | Clear sessions, re-authenticate | Token validation on startup |
| Session secret leaked | Unknown | Rotate secret, invalidate all sessions | Audit logs, secure secret storage |
| Spam attack flooded feedback | DB growing rapidly | Enable rate limiting, cleanup spam | Rate limiting from start |

### Data Recovery

| Scenario | Recovery Path | Time to Recover |
|----------|---------------|-----------------|
| Accidental feedback deletion | Database backup restore | Depends on backup frequency |
| Screenshot storage corruption | Object storage versioning | Minutes if versioned |
| All sessions invalidated | Users re-login | Inconvenient but recoverable |
| Schema migration broke production | Rollback migration + redeploy | Hours if untested |

**Key Principle:** Every "recovery strategy" is better implemented as a "prevention strategy."

---

## Pitfall-to-Phase Mapping

### Phase 1: Core API & Data Model
**Priority:** Set up patterns that prevent debt

| Pitfall | Action | Criticality |
|---------|--------|-------------|
| File upload security | Validate type, sanitize name, store safely | CRITICAL |
| Rate limiting | Add middleware on all endpoints | CRITICAL |
| Connection pool leaks | Establish try/finally pattern | HIGH |
| Missing indexes | Add with initial schema | HIGH |
| No input validation | Add Zod schemas | HIGH |
| SQL injection | Parameterized queries only | CRITICAL |
| Storage limits | Configure upload limits | MEDIUM |

### Phase 2: Authentication
**Priority:** No security shortcuts

| Pitfall | Action | Criticality |
|---------|--------|-------------|
| OAuth state validation | Implement from start | CRITICAL |
| Session secret | Strong secret, env variable | CRITICAL |
| Session cookie security | All flags set correctly | HIGH |
| Auth middleware | Protect all admin routes | CRITICAL |
| Session fixation | Regenerate on login | HIGH |

### Phase 3: Admin Dashboard
**Priority:** Usable without being slow

| Pitfall | Action | Criticality |
|---------|--------|-------------|
| N+1 queries | Use JOINs or batch loading | HIGH |
| No pagination | Required on all lists | HIGH |
| XSS in feedback display | Sanitize output | CRITICAL |
| Missing loading states | Add to all async operations | MEDIUM |
| Lost filter state | Persist in URL params | LOW |

### Phase 4: Notifications
**Priority:** Reliability over speed

| Pitfall | Action | Criticality |
|---------|--------|-------------|
| Connection pool exhaustion | Single pooled transporter | HIGH |
| Synchronous sending | Queue-based architecture | HIGH |
| No retry mechanism | Add retry with backoff | MEDIUM |
| Email injection | Sanitize all content | HIGH |
| Spam to admin | Rate limit notifications | MEDIUM |

### Phase 5: Production & Deployment
**Priority:** Don't launch blind

| Pitfall | Action | Criticality |
|---------|--------|-------------|
| No health checks | Add endpoint, configure Docker | HIGH |
| Missing monitoring | Disk, connections, errors | HIGH |
| Secrets in logs | Audit logging configuration | HIGH |
| No graceful shutdown | Handle SIGTERM | MEDIUM |
| Storage filling up | Monitoring + cleanup jobs | HIGH |

---

## Summary

**Top 3 "Must Not Skip" Items:**
1. **OAuth state validation** - CSRF protection is non-negotiable
2. **File upload validation** - Check type by magic bytes, regenerate filename
3. **Connection pool management** - Always release, monitor pool health

**Top 3 "Easy to Defer, Hard to Fix Later":**
1. Database migrations (do from day one)
2. Structured logging (retrofitting is painful)
3. Input validation schemas (grows exponentially harder)

**Red Flags During Development:**
- "I'll add security later"
- "It works in development"
- "Only I'll use it, so..."
- "Let me just hardcode this for now"
- "The tutorial didn't include this"

---

*Research compiled: 2025-01-15*
*Confidence: Mix of HIGH (direct experience/documentation) and MEDIUM (logical extrapolation)*
*Stack focus: Node.js, MariaDB, GitHub OAuth, SMTP, Docker*
