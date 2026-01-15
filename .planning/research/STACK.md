# Technology Stack Research: Feedback Collection System

**Research Date:** January 2025
**Domain:** Feedback Collection / Internal Tooling
**Knowledge Source:** Claude knowledge base (cutoff May 2025)
**Note:** Web search unavailable - recommendations based on established best practices and library stability. Verify versions on npm before implementation.

---

## Executive Summary

Recommended production stack for a self-hosted feedback collection tool:

| Layer | Technology | Version |
|-------|------------|---------|
| Runtime | Node.js | 20 LTS or 22 LTS |
| Backend Framework | Express.js | 4.x (5.x when stable) |
| Database | MariaDB | 10.11+ LTS |
| DB Driver | mysql2 | 3.x |
| ORM | Prisma or Knex.js | Latest |
| Frontend | React | 18.x |
| Build Tool | Vite | 5.x |
| File Uploads | Multer + Sharp | Latest |
| OAuth | Passport.js + passport-github2 | Latest |
| Email | Nodemailer | 6.x |
| Validation | Zod | 3.x |

---

## Core Technologies

### 1. Node.js Runtime

**Recommendation:** Node.js 20 LTS (Jod) or Node.js 22 LTS (when available)

| Version | Status | End of Life | Recommendation |
|---------|--------|-------------|----------------|
| Node 18 | Maintenance LTS | April 2025 | Avoid for new projects |
| Node 20 | Active LTS | April 2026 | **Recommended** |
| Node 22 | Current/LTS | April 2027 | Good choice if LTS released |

**Rationale:**
- LTS versions receive security updates for 30 months
- Node 20 includes native fetch, test runner, and improved ESM support
- Node 20 has stable `--watch` flag for development

**Confidence:** HIGH (Official Node.js release schedule)

```bash
# Installation via nvm (recommended)
nvm install 20
nvm use 20
nvm alias default 20

# Or via Docker
FROM node:20-alpine
```

---

### 2. Backend Framework: Express.js

**Recommendation:** Express 4.21.x (stable) or Express 5.x (if released stable)

**Rationale:**
- Express 4.x is battle-tested with massive ecosystem
- Express 5.x has been in beta/RC for years - verify stability before adopting
- For new projects, Express 4.x remains the safe choice

**Confidence:** HIGH (Express is the de-facto Node.js web framework)

```bash
npm install express@4
npm install --save-dev @types/express  # TypeScript support
```

**Project Structure (2025 Best Practices):**
```
src/
├── routes/           # Route definitions
├── controllers/      # Request handlers
├── services/         # Business logic
├── repositories/     # Database access
├── middleware/       # Custom middleware
├── utils/            # Helpers
├── config/           # Configuration
└── types/            # TypeScript types
```

**Alternatives Considered:**
| Framework | When to Use | Why Not for This Project |
|-----------|-------------|-------------------------|
| Fastify | High-performance APIs | Smaller ecosystem, steeper learning curve |
| NestJS | Enterprise, complex apps | Overkill for feedback tool |
| Hono | Edge/serverless | Less mature for traditional servers |
| Koa | Minimalist | Smaller ecosystem than Express |

---

### 3. Database: MariaDB with mysql2

**Recommendation:** MariaDB 10.11 LTS + mysql2 3.x

**MariaDB Version:**
| Version | Type | Support Until | Notes |
|---------|------|---------------|-------|
| 10.11 | LTS | Feb 2028 | **Recommended** |
| 11.x | Short-term | ~1 year | Innovation releases |

**Confidence:** HIGH (Official MariaDB release schedule)

**Node.js Driver: mysql2**

**Why mysql2 over mysql:**
- Prepared statements support
- Promise-based API (native)
- Better performance
- Active maintenance
- MySQL 8+ and MariaDB compatible

**Confidence:** HIGH (mysql2 is the standard MySQL/MariaDB driver for Node.js)

```bash
npm install mysql2
```

**Connection Pool Setup:**
```javascript
// src/config/database.js
import mysql from 'mysql2/promise';

const pool = mysql.createPool({
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT) || 3306,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  waitForConnections: true,
  connectionLimit: 10,
  maxIdle: 10,
  idleTimeout: 60000,
  queueLimit: 0,
  enableKeepAlive: true,
  keepAliveInitialDelay: 0
});

export default pool;
```

---

### 4. ORM / Query Builder

**Recommendation:** Prisma (for type safety) OR Knex.js (for SQL control)

#### Option A: Prisma (Recommended for TypeScript projects)

**Pros:**
- Excellent TypeScript integration
- Auto-generated types from schema
- Migrations built-in
- Great developer experience

**Cons:**
- Adds abstraction layer
- Learning curve for Prisma schema syntax
- Slightly larger bundle

**Confidence:** HIGH (Prisma is the leading Node.js ORM)

```bash
npm install prisma --save-dev
npm install @prisma/client
npx prisma init
```

**Schema Example (schema.prisma):**
```prisma
datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model Feedback {
  id          Int       @id @default(autoincrement())
  type        String    // 'bug', 'feature', 'rating'
  title       String
  description String    @db.Text
  rating      Int?
  sourceApp   String
  userEmail   String?
  status      String    @default("new")
  priority    String?
  tags        Tag[]
  screenshots Screenshot[]
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
}

model Screenshot {
  id         Int      @id @default(autoincrement())
  feedbackId Int
  feedback   Feedback @relation(fields: [feedbackId], references: [id])
  filename   String
  path       String
  mimeType   String
  size       Int
  createdAt  DateTime @default(now())
}

model Tag {
  id       Int        @id @default(autoincrement())
  name     String     @unique
  feedback Feedback[]
}

model Admin {
  id        Int      @id @default(autoincrement())
  githubId  String   @unique
  username  String
  email     String?
  avatarUrl String?
  createdAt DateTime @default(now())
  lastLogin DateTime?
}
```

#### Option B: Knex.js (More SQL control)

**Pros:**
- Direct SQL control
- Lightweight
- Excellent for complex queries

**Confidence:** HIGH (Knex.js is a mature, stable query builder)

```bash
npm install knex mysql2
```

---

### 5. Frontend: React + Vite

**Recommendation:** React 18.x + Vite 5.x

**React Version:**
- React 18.2.x or 18.3.x (stable)
- React 19 may be available - verify stability before adopting

**Confidence:** HIGH (React 18 is well-established)

**Build Tool: Vite**

**Why Vite over Create React App:**
- CRA is deprecated/unmaintained
- Vite is fast (ESBuild + Rollup)
- Better DX with instant HMR
- Official React documentation recommends Vite

**Confidence:** HIGH (CRA deprecation is confirmed, Vite is recommended)

```bash
npm create vite@latest admin-dashboard -- --template react-ts
cd admin-dashboard
npm install
```

**Admin Dashboard UI Library Options:**

| Library | Best For | Bundle Size | Confidence |
|---------|----------|-------------|------------|
| **shadcn/ui** | Modern, customizable | Tiny (copy components) | HIGH |
| Ant Design | Feature-rich admin panels | Large | HIGH |
| Material UI | Google design system | Medium-Large | HIGH |
| Chakra UI | Accessible, composable | Medium | MEDIUM |

**Recommendation:** shadcn/ui + Tailwind CSS for modern admin dashboards

```bash
# In your Vite React project
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# shadcn/ui setup
npx shadcn-ui@latest init
```

---

## Supporting Libraries

### 6. File Uploads: Multer + Sharp

**Recommendation:** Multer 1.4.x + Sharp (for image processing)

**Multer** - De-facto file upload middleware for Express
**Sharp** - High-performance image processing (resizing screenshots)

**Confidence:** HIGH (Both are industry standards)

```bash
npm install multer sharp
npm install --save-dev @types/multer  # TypeScript
```

**Implementation:**
```javascript
// src/middleware/upload.js
import multer from 'multer';
import path from 'path';
import crypto from 'crypto';

const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, process.env.UPLOAD_DIR || './uploads');
  },
  filename: (req, file, cb) => {
    const uniqueSuffix = crypto.randomBytes(16).toString('hex');
    cb(null, `${uniqueSuffix}${path.extname(file.originalname)}`);
  }
});

const fileFilter = (req, file, cb) => {
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];
  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error('Invalid file type. Only JPEG, PNG, GIF, and WebP are allowed.'), false);
  }
};

export const upload = multer({
  storage,
  fileFilter,
  limits: {
    fileSize: 10 * 1024 * 1024, // 10MB max
    files: 5 // Max 5 files per request
  }
});
```

**Image Processing with Sharp:**
```javascript
// src/services/imageService.js
import sharp from 'sharp';
import path from 'path';

export async function processScreenshot(inputPath, outputDir) {
  const filename = path.basename(inputPath);
  const thumbnailPath = path.join(outputDir, `thumb_${filename}`);

  // Create thumbnail
  await sharp(inputPath)
    .resize(300, 200, { fit: 'inside', withoutEnlargement: true })
    .jpeg({ quality: 80 })
    .toFile(thumbnailPath);

  // Optimize original
  const optimizedPath = path.join(outputDir, `opt_${filename}`);
  await sharp(inputPath)
    .resize(1920, 1080, { fit: 'inside', withoutEnlargement: true })
    .jpeg({ quality: 85 })
    .toFile(optimizedPath);

  return { thumbnailPath, optimizedPath };
}
```

**Alternatives:**
| Library | Use Case | Notes |
|---------|----------|-------|
| Formidable | Streaming uploads | More complex API |
| Busboy | Low-level parsing | Multer uses this internally |
| express-fileupload | Simple API | Less flexible than Multer |

---

### 7. OAuth: Passport.js + passport-github2

**Recommendation:** Passport.js 0.7.x + passport-github2

**Confidence:** HIGH (Passport is the standard Node.js auth middleware)

```bash
npm install passport passport-github2 express-session
npm install --save-dev @types/passport @types/passport-github2 @types/express-session
```

**Session Store:** Use connect-redis or express-mysql-session for production

```bash
npm install connect-redis ioredis
# OR
npm install express-mysql-session
```

**Implementation:**
```javascript
// src/config/passport.js
import passport from 'passport';
import { Strategy as GitHubStrategy } from 'passport-github2';
import { findOrCreateAdmin } from '../services/adminService.js';

passport.serializeUser((user, done) => {
  done(null, user.id);
});

passport.deserializeUser(async (id, done) => {
  try {
    const user = await findAdminById(id);
    done(null, user);
  } catch (err) {
    done(err);
  }
});

passport.use(new GitHubStrategy({
    clientID: process.env.GITHUB_CLIENT_ID,
    clientSecret: process.env.GITHUB_CLIENT_SECRET,
    callbackURL: process.env.GITHUB_CALLBACK_URL,
    scope: ['user:email']
  },
  async (accessToken, refreshToken, profile, done) => {
    try {
      const user = await findOrCreateAdmin({
        githubId: profile.id,
        username: profile.username,
        email: profile.emails?.[0]?.value,
        avatarUrl: profile.photos?.[0]?.value
      });
      done(null, user);
    } catch (err) {
      done(err);
    }
  }
));

export default passport;
```

**Routes:**
```javascript
// src/routes/auth.js
import express from 'express';
import passport from '../config/passport.js';

const router = express.Router();

router.get('/github', passport.authenticate('github', { scope: ['user:email'] }));

router.get('/github/callback',
  passport.authenticate('github', { failureRedirect: '/login' }),
  (req, res) => {
    res.redirect('/dashboard');
  }
);

router.post('/logout', (req, res, next) => {
  req.logout((err) => {
    if (err) return next(err);
    res.redirect('/');
  });
});

export default router;
```

**Alternatives Considered:**
| Library | Use Case | Why Not |
|---------|----------|---------|
| Auth.js (NextAuth) | Next.js projects | Designed for Next.js |
| Grant | Multi-provider OAuth | Less documentation |
| Simple-oauth2 | Low-level OAuth | More manual work |

---

### 8. Email: Nodemailer

**Recommendation:** Nodemailer 6.x

**Confidence:** HIGH (Nodemailer is the only serious Node.js email solution)

```bash
npm install nodemailer
npm install --save-dev @types/nodemailer
```

**Implementation:**
```javascript
// src/services/emailService.js
import nodemailer from 'nodemailer';

const transporter = nodemailer.createTransport({
  host: process.env.SMTP_HOST,
  port: parseInt(process.env.SMTP_PORT) || 587,
  secure: process.env.SMTP_SECURE === 'true', // true for 465, false for other ports
  auth: {
    user: process.env.SMTP_USER,
    pass: process.env.SMTP_PASS
  }
});

export async function sendFeedbackNotification(feedback, recipients) {
  const mailOptions = {
    from: `"Feedback System" <${process.env.SMTP_FROM}>`,
    to: recipients.join(', '),
    subject: `New ${feedback.type}: ${feedback.title}`,
    html: `
      <h2>New Feedback Received</h2>
      <p><strong>Type:</strong> ${feedback.type}</p>
      <p><strong>Title:</strong> ${feedback.title}</p>
      <p><strong>Description:</strong></p>
      <p>${feedback.description}</p>
      <p><strong>Source App:</strong> ${feedback.sourceApp}</p>
      ${feedback.rating ? `<p><strong>Rating:</strong> ${feedback.rating}/5</p>` : ''}
      <hr>
      <p><a href="${process.env.APP_URL}/dashboard/feedback/${feedback.id}">View in Dashboard</a></p>
    `
  };

  return transporter.sendMail(mailOptions);
}

// Verify connection on startup
export async function verifyEmailConnection() {
  try {
    await transporter.verify();
    console.log('SMTP connection verified');
    return true;
  } catch (error) {
    console.error('SMTP connection failed:', error);
    return false;
  }
}
```

**For Email Templates:** Consider using MJML or React Email

```bash
npm install @react-email/components  # Modern approach
# OR
npm install mjml  # Traditional approach
```

---

### 9. Validation: Zod

**Recommendation:** Zod 3.x

**Why Zod over alternatives:**
- TypeScript-first design
- Runtime validation + static types
- Excellent error messages
- No code generation required
- Growing ecosystem

**Confidence:** HIGH (Zod is the leading TypeScript validation library)

```bash
npm install zod
```

**Implementation:**
```javascript
// src/schemas/feedback.js
import { z } from 'zod';

export const feedbackSchema = z.object({
  type: z.enum(['bug', 'feature', 'rating']),
  title: z.string().min(5).max(200),
  description: z.string().min(10).max(10000),
  rating: z.number().int().min(1).max(5).optional(),
  sourceApp: z.string().min(1).max(100),
  userEmail: z.string().email().optional().nullable(),
  tags: z.array(z.string()).optional()
});

export const feedbackUpdateSchema = feedbackSchema.partial().extend({
  status: z.enum(['new', 'in-progress', 'resolved', 'closed']).optional(),
  priority: z.enum(['low', 'medium', 'high', 'critical']).optional()
});

// Express middleware for validation
export function validateBody(schema) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(400).json({
        error: 'Validation failed',
        details: result.error.issues
      });
    }
    req.validatedBody = result.data;
    next();
  };
}
```

**Alternatives:**
| Library | Pros | Cons |
|---------|------|------|
| Joi | Mature, feature-rich | No TypeScript inference |
| Yup | Good for forms | Less TypeScript support |
| class-validator | Decorator-based | Requires classes |
| Ajv | Fast JSON Schema | Verbose schemas |

---

## Development Tools

### 10. TypeScript

**Recommendation:** TypeScript 5.x

**Confidence:** HIGH (TypeScript is standard for production Node.js)

```bash
npm install --save-dev typescript @types/node ts-node tsx
npx tsc --init
```

**tsconfig.json (Node.js 20+):**
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

**For Development:** Use `tsx` for running TypeScript directly
```bash
npm install --save-dev tsx
# Run with: npx tsx src/index.ts
# Watch mode: npx tsx watch src/index.ts
```

---

### 11. Testing

**Recommendation:** Vitest (frontend) + Node.js Test Runner or Jest (backend)

**Frontend Testing:**
```bash
npm install --save-dev vitest @testing-library/react @testing-library/jest-dom jsdom
```

**Backend Testing:**
```bash
# Option A: Native Node.js test runner (Node 20+)
# No installation needed - built into Node.js

# Option B: Jest
npm install --save-dev jest @types/jest ts-jest
```

**Confidence:** HIGH (Vitest is the standard for Vite projects)

---

### 12. Linting & Formatting

**Recommendation:** ESLint 9.x (flat config) + Prettier

```bash
npm install --save-dev eslint @eslint/js typescript-eslint prettier eslint-config-prettier
```

**eslint.config.js (ESLint 9 flat config):**
```javascript
import js from '@eslint/js';
import tseslint from 'typescript-eslint';
import prettier from 'eslint-config-prettier';

export default [
  js.configs.recommended,
  ...tseslint.configs.recommended,
  prettier,
  {
    rules: {
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      '@typescript-eslint/explicit-function-return-type': 'off'
    }
  }
];
```

**Confidence:** HIGH (ESLint 9 flat config is the new standard)

---

### 13. Docker Setup

**Dockerfile (Node.js Backend):**
```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:20-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/prisma ./prisma

# Create non-root user
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
USER nodejs

EXPOSE 3000
CMD ["node", "dist/index.js"]
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  api:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=mysql://user:password@db:3306/feedback
      - GITHUB_CLIENT_ID=${GITHUB_CLIENT_ID}
      - GITHUB_CLIENT_SECRET=${GITHUB_CLIENT_SECRET}
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - uploads:/app/uploads
    restart: unless-stopped

  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - api
    restart: unless-stopped

  db:
    image: mariadb:10.11
    environment:
      - MYSQL_ROOT_PASSWORD=${DB_ROOT_PASSWORD}
      - MYSQL_DATABASE=feedback
      - MYSQL_USER=feedback_user
      - MYSQL_PASSWORD=${DB_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

volumes:
  uploads:
  db_data:
```

**Confidence:** HIGH (Standard Docker practices)

---

## Complete Installation Commands

```bash
# Backend setup
mkdir feedback-backend && cd feedback-backend
npm init -y
npm install express mysql2 @prisma/client passport passport-github2 \
  express-session multer sharp nodemailer zod dotenv cors helmet \
  express-rate-limit
npm install --save-dev typescript @types/node @types/express \
  @types/passport @types/passport-github2 @types/express-session \
  @types/multer @types/nodemailer tsx prisma eslint prettier \
  @eslint/js typescript-eslint eslint-config-prettier

# Frontend setup
npm create vite@latest feedback-frontend -- --template react-ts
cd feedback-frontend
npm install
npm install axios @tanstack/react-query react-router-dom zustand
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
npx shadcn-ui@latest init
```

---

## What NOT to Use

| Technology | Reason | Alternative |
|------------|--------|-------------|
| Create React App | Deprecated, unmaintained | Vite |
| mysql (npm) | Outdated, unmaintained | mysql2 |
| Mongoose | For MongoDB, not MariaDB | Prisma/Knex |
| Sequelize | TypeScript support issues | Prisma |
| bcrypt (pure JS) | Security concerns | bcrypt (native) or argon2 |
| Express 5.x (beta) | Not stable yet | Express 4.x |
| node-fetch | Not needed in Node 18+ | Native fetch |
| moment.js | Deprecated, large | date-fns or Luxon |
| Request (npm) | Deprecated | axios or native fetch |

---

## Security Checklist

- [ ] Helmet.js for HTTP security headers
- [ ] Rate limiting on API endpoints
- [ ] CORS configuration
- [ ] Input validation on all endpoints
- [ ] File upload restrictions (type, size)
- [ ] Session security (secure cookies, httpOnly)
- [ ] SQL injection prevention (use parameterized queries/ORM)
- [ ] XSS prevention (sanitize user input for display)
- [ ] Environment variables for secrets (never commit .env)
- [ ] Non-root Docker user

```bash
npm install helmet cors express-rate-limit
```

```javascript
// Security middleware setup
import helmet from 'helmet';
import cors from 'cors';
import rateLimit from 'express-rate-limit';

app.use(helmet());
app.use(cors({
  origin: process.env.FRONTEND_URL,
  credentials: true
}));
app.use('/api/', rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
}));
```

---

## Confidence Summary

| Category | Items | Confidence |
|----------|-------|------------|
| Core Stack | Node 20, Express 4, React 18, MariaDB 10.11 | HIGH |
| Database | mysql2, Prisma | HIGH |
| File Uploads | Multer, Sharp | HIGH |
| OAuth | Passport.js, passport-github2 | HIGH |
| Email | Nodemailer | HIGH |
| Validation | Zod | HIGH |
| Build Tools | Vite, TypeScript 5 | HIGH |
| UI Framework | shadcn/ui, Tailwind CSS | MEDIUM-HIGH |
| Specific Versions | Exact latest versions | MEDIUM (verify on npm) |

---

## Next Steps

1. Verify latest stable versions on npm before installation
2. Set up GitHub OAuth application for credentials
3. Configure SMTP server or use a service (SendGrid, AWS SES)
4. Design database schema based on Prisma model above
5. Set up CI/CD pipeline (GitHub Actions recommended)

---

*This research document was compiled from established best practices and stable library recommendations. Always verify current versions and check official documentation before production deployment.*
