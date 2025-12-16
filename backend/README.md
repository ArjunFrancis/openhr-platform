# OpenHR Backend API

**Status**: 🚧 Not yet implemented (skeleton only)

---

## Overview

This directory will contain the Node.js + Express backend API for OpenHR.

### Technology Stack

- **Runtime**: Node.js 18+ with TypeScript
- **Framework**: Express.js
- **Database**: Supabase (PostgreSQL)
- **Authentication**: Supabase Auth (JWT validation)
- **Caching**: Redis
- **Logging**: Winston
- **API Docs**: OpenAPI 3.1 (Swagger)
- **Testing**: Mocha + Chai + Supertest
- **Linting**: ESLint + Prettier

---

## Planned Directory Structure

```
backend/
├── src/
│   ├── api/                  # API routes and controllers
│   │   ├── routes/
│   │   │   ├── auth.routes.ts
│   │   │   ├── profiles.routes.ts
│   │   │   ├── matches.routes.ts
│   │   │   ├── messages.routes.ts
│   │   │   └── index.ts
│   │   ├── controllers/
│   │   │   ├── auth.controller.ts
│   │   │   ├── profiles.controller.ts
│   │   │   ├── matches.controller.ts
│   │   │   └── messages.controller.ts
│   │   └── middlewares/
│   │       ├── auth.middleware.ts
│   │       ├── validation.middleware.ts
│   │       ├── error.middleware.ts
│   │       └── rateLimit.middleware.ts
│   ├── services/             # Business logic layer
│   │   ├── profile.service.ts
│   │   ├── match.service.ts
│   │   ├── message.service.ts
│   │   └── ml.service.ts        # ML Service API client
│   ├── repositories/         # Data access layer
│   │   ├── profile.repository.ts
│   │   ├── match.repository.ts
│   │   └── message.repository.ts
│   ├── db/                   # Database configuration
│   │   ├── supabase.ts
│   │   ├── redis.ts
│   │   └── migrations/       # SQL migrations (if manual)
│   ├── utils/                # Utility functions
│   │   ├── logger.ts
│   │   ├── errors.ts
│   │   ├── validators.ts
│   │   └── helpers.ts
│   ├── types/                # TypeScript types
│   │   ├── api.types.ts
│   │   ├── database.types.ts
│   │   └── supabase.types.ts # Auto-generated from Supabase
│   ├── config/               # Configuration files
│   │   ├── index.ts
│   │   ├── database.config.ts
│   │   └── redis.config.ts
│   ├── app.ts                # Express app setup
│   └── server.ts             # HTTP server entry point
├── tests/                    # Test files
│   ├── unit/
│   ├── integration/
│   └── fixtures/
├── docs/                     # API documentation
│   └── openapi.yaml          # OpenAPI 3.1 spec
├── .env.example              # Example environment variables
├── .eslintrc.json            # ESLint configuration
├── .prettierrc               # Prettier configuration
├── Dockerfile                # Docker container definition
├── package.json              # Dependencies and scripts
├── tsconfig.json             # TypeScript configuration
└── README.md                 # This file
```

---

## Key Features to Implement

### 1. Authentication & Authorization
- [ ] JWT validation middleware (Supabase Auth)
- [ ] OAuth callback handlers (GitHub, Google)
- [ ] Session management
- [ ] Role-based access control (RBAC)
- [ ] API key authentication (for third-party integrations)

### 2. Profile Management
- [ ] CRUD operations for user profiles
- [ ] Skill management endpoints
- [ ] Preference updates
- [ ] Profile enrichment orchestration (call ML service)
- [ ] Public profile view (with privacy controls)

### 3. Matching Engine
- [ ] Fetch candidate pool with filters
- [ ] Call ML service for scoring and ranking
- [ ] Store match records in database
- [ ] Handle like/pass actions
- [ ] Track match history and analytics

### 4. Messaging System
- [ ] Create and list conversations
- [ ] Send and retrieve messages
- [ ] Integrate with Supabase Realtime
- [ ] Mark messages as read
- [ ] Support file attachments (future)

### 5. Notifications
- [ ] In-app notification system
- [ ] Email notifications (via third-party service)
- [ ] Push notifications (future)
- [ ] Notification preferences management

### 6. Admin & Moderation
- [ ] User management endpoints (admin only)
- [ ] Content moderation tools
- [ ] Analytics and reporting
- [ ] Audit logging

---

## Environment Variables

Create a `.env` file with the following:

```bash
# Server
PORT=4000
NODE_ENV=development

# Supabase
SUPABASE_URL=your_supabase_project_url
SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_role_key
SUPABASE_ANON_KEY=your_supabase_anon_key

# Redis
REDIS_URL=redis://localhost:6379

# ML Service
ML_SERVICE_URL=http://localhost:8000

# JWT
JWT_SECRET=your_jwt_secret

# CORS
CORS_ORIGIN=http://localhost:3000

# Rate Limiting
RATE_LIMIT_WINDOW_MS=60000
RATE_LIMIT_MAX_REQUESTS=100

# Logging
LOG_LEVEL=info
```

---

## Installation (When Implemented)

```bash
# Install dependencies
npm install

# Generate Supabase types
npm run generate:types

# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run tests
npm test

# Run tests with coverage
npm run test:coverage

# Lint and format
npm run lint
npm run format
```

---

## Implementation Guide for Coding Agents

### Step 1: Project Setup
1. Initialize Node.js project with TypeScript:
   ```bash
   npm init -y
   npm install express typescript @types/node @types/express
   npm install ts-node nodemon --save-dev
   ```
2. Install dependencies:
   ```bash
   npm install @supabase/supabase-js ioredis winston helmet cors
   npm install joi express-rate-limit express-async-errors
   ```
3. Configure TypeScript (`tsconfig.json`)
4. Set up ESLint and Prettier

### Step 2: Database & Auth Setup
1. Initialize Supabase client (`src/db/supabase.ts`)
2. Generate TypeScript types from Supabase schema:
   ```bash
   npx supabase gen types typescript --project-id <project-id> > src/types/supabase.types.ts
   ```
3. Create auth middleware to validate JWTs
4. Test connection to Postgres and Redis

### Step 3: Core API Endpoints
1. Implement profile endpoints (refer to `docs/architecture/api-specification.md`)
2. Implement match endpoints
3. Implement messaging endpoints
4. Add input validation with Joi
5. Add error handling middleware

### Step 4: ML Service Integration
1. Create ML service client (`src/services/ml.service.ts`)
2. Implement profile enrichment workflow
3. Implement match scoring workflow
4. Add circuit breaker for resilience
5. Cache ML responses in Redis

### Step 5: Realtime Integration
1. Trigger Supabase Realtime events on message creation
2. Implement webhook handlers for async ML results
3. Add presence tracking (optional)

### Step 6: Testing & Documentation
1. Write unit tests for services and repositories
2. Write integration tests for API endpoints
3. Generate OpenAPI spec from route definitions
4. Add Swagger UI for interactive API docs
5. Load testing with Artillery or k6

---

## API Endpoints (Planned)

Refer to [API Specification](../docs/architecture/api-specification.md) for full details.

**Authentication**
- `POST /auth/callback` - OAuth callback handler
- `POST /auth/refresh` - Refresh JWT token
- `POST /auth/logout` - Logout user

**Profiles**
- `GET /profiles/:id` - Get profile by ID
- `PUT /profiles/:id` - Update profile
- `POST /profiles/enrich` - Trigger profile enrichment

**Matches**
- `GET /matches` - List matches for current user
- `POST /matches/:id/like` - Like a candidate
- `POST /matches/:id/pass` - Pass on a candidate

**Messaging**
- `GET /conversations` - List conversations
- `GET /conversations/:id/messages` - Get messages
- `POST /conversations/:id/messages` - Send message

**Skills**
- `GET /skills/search` - Search skills taxonomy
- `GET /skills/normalize` - Normalize skill name

---

## Testing Strategy

### Unit Tests
- Test business logic in services
- Test data access in repositories
- Mock external dependencies (Supabase, Redis, ML service)

### Integration Tests
- Test API endpoints with real database (Supabase local)
- Test authentication and authorization
- Test error handling and validation

### Load Tests
- Simulate 1000+ concurrent users
- Test rate limiting and caching
- Identify performance bottlenecks

---

## Related Documentation

- [System Architecture](../docs/architecture/system-architecture.md)
- [API Specification](../docs/architecture/api-specification.md)
- [Database Schema](../docs/architecture/database-schema.md)
- [Auth & Authorization](../docs/platform/auth-authorization.md)

---

**Status**: Ready for implementation by backend coding agents. All specifications are complete.

**Questions?** Open a GitHub Discussion or issue.