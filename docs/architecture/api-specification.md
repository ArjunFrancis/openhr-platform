# API Specification for OpenHR

**Last updated:** November 30, 2025  
**Confidence:** 95%+ (OpenAPI 3.1 compliance, REST best practices)

---

## 1. Overview

OpenHR exposes a **RESTful API** using OpenAPI 3.1 specification for all public and internal endpoints. The API is versioned, authenticated, and documented via Swagger UI. It supports both frontend consumption and agent/service communication.

**Key Principles:**
- **RESTful design:** Resources (users, profiles, matches, messages) with standard HTTP methods.
- **Versioning:** `/v1/` prefix, semantic versioning for breaking changes.
- **Authentication:** JWT Bearer tokens for all endpoints; OAuth2 for delegated auth.
- **Rate limiting:** 100 requests/minute per user (configurable).
- **Pagination:** Offset-based with `limit` and `offset` parameters.
- **Error handling:** Standard HTTP status codes with JSON error bodies.
- **Documentation:** Auto-generated OpenAPI spec at `/openapi.json`.

---

## 2. Authentication & Authorization

### 2.1 JWT Authentication
All endpoints require a valid JWT in the `Authorization` header:
```
Authorization: Bearer <jwt_token>
```

**Token Flow:**
1. User logs in via `/auth/login` → receives JWT.
2. JWT contains: `user_id`, `role`, `exp`, `iat`.
3. API Gateway validates JWT on each request.
4. Role-based access: admin endpoints require `role: admin`.

**Example Login Request:**
```bash
POST /v1/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "secure_password"
}
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "user_id": "uuid-here"
}
```

### 2.2 OAuth2 (GitHub/LinkedIn)
For delegated profile enrichment:
- `/auth/github` → redirects to GitHub OAuth.
- Callback extracts GitHub data, creates/enriches profile.
- Similar flow for LinkedIn integration.

---

## 3. Core Resources & Endpoints

### 3.1 Users & Profiles

**Create User + Profile (Onboarding):**
```bash
POST /v1/users
Content-Type: application/json
Authorization: Bearer <admin_token>  # Admin only

{
  "email": "user@example.com",
  "password": "secure_password",
  "role": "user",
  "profile": {
    "display_name": "John Doe",
    "bio": "Full-stack developer seeking technical co-founder",
    "location": "San Francisco, CA",
    "looking_for_role": "technical_cofounder"
  }
}
```

**Get Profile:**
```bash
GET /v1/profiles/{profile_id}
Authorization: Bearer <jwt_token>
```

**Response:**
```json
{
  "id": "uuid",
  "display_name": "John Doe",
  "bio": "...",
  "skills": [
    {"id": "uuid", "name": "React", "proficiency": "advanced"},
    {"id": "uuid", "name": "Node.js", "proficiency": "expert"}
  ],
  "location": "San Francisco, CA",
  "github_url": "https://github.com/johndoe",
  "verification_status": "verified"
}
```

**Update Profile:**
```bash
PATCH /v1/profiles/{profile_id}
Authorization: Bearer <jwt_token>
Content-Type: application/json

{
  "bio": "Updated bio",
  "skills": [
    {"skill_id": "uuid", "proficiency": "intermediate"},
    {"skill_id": "uuid", "proficiency": "expert"}
  ]
}
```

**Enrich Profile (AI Agent):**
```bash
POST /v1/profiles/{profile_id}/enrich
Authorization: Bearer <jwt_token>
Content-Type: multipart/form-data

# Form data with resume file
resume=@resume.pdf
github_username=johndoe
```

**Response:**
```json
{
  "status": "success",
  "enriched_data": {
    "skills_extracted": ["React", "Node.js", "Python"],
    "experience_summary": "5+ years full-stack development...",
    "github_stats": {"stars": 42, "forks": 10, "commits": 150}
  }
}
```

### 3.2 Startups & Roles

**Create Startup:**
```bash
POST /v1/startups
Authorization: Bearer <jwt_token>
Content-Type: application/json

{
  "name": "TechStartup",
  "description": "AI-powered talent matching platform",
  "stage": "mvp",
  "industry": "SaaS",
  "location": "Remote",
  "roles": [
    {
      "title": "Technical Co-founder",
      "role_type": "technical",
      "description": "Full-stack development, AI/ML experience",
      "equity_min": 20,
      "equity_max": 40,
      "required_skills": ["React", "Python", "AI"]
    }
  ]
}
```

**Get Startup Roles:**
```bash
GET /v1/startups/{startup_id}/roles?status=open
Authorization: Bearer <jwt_token>
```

**Response:**
```json
[
  {
    "id": "uuid",
    "title": "Technical Co-founder",
    "description": "...",
    "role_type": "technical",
    "is_filled": false,
    "required_skills": [
      {"id": "uuid", "name": "React", "importance": 5},
      {"id": "uuid", "name": "Python", "importance": 4}
    ]
  }
]
```

### 3.3 Matches & Recommendations

**Get Recommendations:**
```bash
GET /v1/matches/recommendations?limit=10&offset=0&role_type=technical&location=remote
Authorization: Bearer <jwt_token>
```

**Response:**
```json
{
  "matches": [
    {
      "id": "uuid",
      "startup_role": {
        "id": "uuid",
        "title": "Technical Co-founder",
        "startup": {
          "name": "TechStartup",
          "stage": "mvp",
          "location": "Remote"
        }
      },
      "score": 0.92,
      "status": "suggested",
      "explanation": "High skill match (React, Python) + remote compatible",
      "score_breakdown": {
        "skill_fit": 0.95,
        "complementary": 0.88,
        "vision_alignment": 0.90,
        "work_style": 0.85
      }
    }
  ],
  "pagination": {
    "limit": 10,
    "offset": 0,
    "total": 42
  }
}
```

**Accept/Reject Match:**
```bash
POST /v1/matches/{match_id}/accept
Authorization: Bearer <jwt_token>
```

**Response:**
```json
{
  "status": "success",
  "match": {
    "id": "uuid",
    "status": "mutual",
    "suggestions": [
      "Hi! I'm excited about your AI platform. I have experience with React and Python...",
      "Hello! Your mission to connect co-founders resonates with me. Let's discuss..."
    ]
  }
}
```

### 3.4 Messaging

**Get Conversations:**
```bash
GET /v1/conversations?limit=20&offset=0
Authorization: Bearer <jwt_token>
```

**Response:**
```json
[
  {
    "id": "uuid",
    "participants": [
      {"profile_id": "uuid", "display_name": "John Doe", "avatar": "url"}
    ],
    "last_message": {
      "content": "Looking forward to discussing the role!",
      "sent_at": "2025-11-30T10:00:00Z",
      "sender": "John Doe"
    },
    "unread_count": 3
  }
]
```

**Send Message:**
```bash
POST /v1/conversations/{conversation_id}/messages
Authorization: Bearer <jwt_token>
Content-Type: application/json

{
  "content": "Hi! Thanks for the match. I'd love to learn more about your startup."
}
```

**Get Messages:**
```bash
GET /v1/conversations/{conversation_id}/messages?limit=50&before=2025-11-30T10:00:00Z
Authorization: Bearer <jwt_token>
```

**Response:**
```json
[
  {
    "id": "uuid",
    "conversation_id": "uuid",
    "sender_profile_id": "uuid",
    "sender_display_name": "John Doe",
    "content": "Hi! Thanks for the match...",
    "sent_at": "2025-11-30T10:00:00Z",
    "status": "delivered"
  }
]
```

### 3.5 Skills & Search

**Search Skills:**
```bash
GET /v1/skills/search?q=react&type=specific&limit=10
Authorization: Bearer <jwt_token>
```

**Response:**
```json
[
  {
    "id": "uuid",
    "canonical_name": "React",
    "description": "JavaScript library for building user interfaces",
    "level": "specific",
    "domain": "Frontend Development",
    "area": "JavaScript Frameworks",
    "related_skills": ["JavaScript", "TypeScript", "Redux"]
  }
]
```

**Add Profile Skills:**
```bash
POST /v1/profiles/{profile_id}/skills
Authorization: Bearer <jwt_token>
Content-Type: application/json

[
  {
    "skill_id": "uuid",
    "proficiency": "advanced",
    "experience_years": 3,
    "last_used": "2025-01-01"
  }
]
```

---

## 4. Internal Agent APIs

Agents communicate via internal REST endpoints (same auth, but localhost-only access).

**Compute Match Score:**
```bash
POST /v1/internal/matches/compute
Authorization: Bearer <internal_jwt>
Content-Type: application/json

{
  "candidate_profile_id": "uuid",
  "startup_role_id": "uuid",
  "user_preferences": {
    "preferred_location": "remote",
    "preferred_roles": ["technical"]
  }
}
```

**Response:**
```json
{
  "match_id": "uuid",
  "score": 0.92,
  "breakdown": {
    "skill_fit": {"weight": 0.4, "score": 0.95, "contribution": 0.38},
    "complementary": {"weight": 0.2, "score": 0.88, "contribution": 0.176}
  }
}
```

**Generate Explanation:**
```bash
POST /v1/internal/llm/explain-match
Authorization: Bearer <internal_jwt>
Content-Type: application/json

{
  "match_id": "uuid",
  "model": "gpt-4o"
}
```

**Response:**
```json
{
  "explanation": "You matched well with TechStartup because your React and Python skills align perfectly with their technical co-founder role. Your 5+ years of experience and remote work preference also fit their MVP stage and distributed team."
}
```

---

## 5. Error Handling

**Standard Error Format:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email is required",
    "details": [
      {"field": "email", "issue": "must be valid email format"}
    ]
  }
}
```

**HTTP Status Codes:**
- 200 OK - Success
- 201 Created - Resource created
- 400 Bad Request - Validation error
- 401 Unauthorized - Missing/invalid token
- 403 Forbidden - Insufficient permissions
- 404 Not Found - Resource doesn't exist
- 429 Too Many Requests - Rate limit exceeded
- 500 Internal Server Error - Server error

---

## 6. OpenAPI Specification

The full OpenAPI 3.1 spec is available at `/openapi.json`. Key features:

**Security Schemes:**
```yaml
securitySchemes:
  bearerAuth:
    type: http
    scheme: bearer
    bearerFormat: JWT
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
security:
  - bearerAuth: []
```

**Pagination Schema:**
```yaml
components:
  schemas:
    Pagination:
      type: object
      properties:
        limit:
          type: integer
        offset:
          type: integer
        total:
          type: integer
        has_more:
          type: boolean
```

**Rate Limiting Headers:**
- `X-RateLimit-Limit`: Maximum requests per period
- `X-RateLimit-Remaining`: Remaining requests
- `X-RateLimit-Reset`: Time when limit resets

---

## 7. Performance & Scalability

**Caching:**
- Profile data: Redis TTL 5 minutes
- Match scores: Redis TTL 1 hour
- Skill taxonomy: Redis TTL 24 hours

**Async Endpoints:**
- Profile enrichment, recommendation generation use async queues (Celery).
- Long-running tasks return job IDs for polling.

**Example Async Response:**
```json
{
  "job_id": "uuid",
  "status": "queued",
  "estimated_time": "30s",
  "poll_url": "/v1/jobs/{job_id}"
}
```

---

**References:**
- OpenAPI 3.1 Specification [web:227]
- Swagger API Design Best Practices [web:230]
- REST API Design Guidelines (Microsoft, 2025)