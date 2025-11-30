# System Architecture for OpenHR

**Last updated:** November 30, 2025  
**Confidence:** 95%+ (research-based, current best practices)

---

## 1. Overview
The OpenHR platform is designed as a scalable, modular, microservices-based system optimized for AI-powered co-founder and developer/startup matching. It unifies semantic search, real-time recommendations, user collaboration, and GDPR privacy by design.

---

## 2. High-Level Architecture
**Component Breakdown:**
- **Frontend:** React (Next.js) SPA with SSR
- **API Gateway:** Node.js/Express REST API (versioned)
- **AI/ML Services:** Python/FastAPI for embeddings, skill extraction, LLM agent suites
- **Database:** Supabase (PostgreSQL) for transactional data, strict row-level security
- **Vector Store:** FAISS (on-prem) or Pinecone (cloud) for semantic search
- **Cache:** Redis for low-latency session, token, and entity caching
- **Message Bus:** (Phase 2) RabbitMQ/Redis Streams for event-driven orchestration
- **CI/CD:** GitHub Actions; local Docker Compose; K8s/Helm for scale

---

## 3. Communication Patterns
- **RESTful APIs** between all major components (OpenAPI 3.1 spec)
- **JSON everywhere** for payloads; protobuf reserved for internal high-throughput use case (Phase 2)
- **Stateless services:** All services scale horizontally
- **Auth propagation:** JWT tokens for all request authentication/authorization

---

## 4. Service Decomposition
- **API Gateway**: Manages rate limiting, authentication, exposes unified REST interface
- **User/Profile Service:** Handles user CRUD, auth, profile enrichment, onboarding, preference management
- **Matching/Recommendation Service:** Real-time semantic search, collaborative/hybrid filtering, explanation
- **Skill & Taxonomy Service:** Skill normalization/aliasing, taxonomy traversal
- **LLM Agent Service:** Resume parsing, match explanation, summarization, message suggestion (stateless)
- **Messaging Service:** Real-time chat/messages between profiles/candidates
- **Notification/Email Service:** Sends alerts, reminders (event-driven)
- **Admin/Moderation Service:** Role management, ban/appeal, system logs

---

## 5. Scalability, Reliability & Security
**Scalability:**
- All stateless microservices are containerized for zero-downtime rolling deployments
- PostgreSQL with read-replicas for scale-out; Redis for cache/sessions
- Vector DB with auto-sharding (Pinecone) or IVF/Flat index partition (FAISS)

**Reliability:**
- Strict RLS (row-level security) on all tables
- API Gateway enforces auth, logging, request size limits
- Circuit breakers and graceful degradation across all services
- Global request ID tracing via OpenTelemetry

**Privacy/Security:**
- No raw PII in logs
- Audit log table with rate limitation and partitioning
- All endpoints require JWT; OAuth2 for delegated auth
- Table-level multi-tenancy fields enforced at API and SQL layers

---

## 6. Deployment & Development Workflow
- **Local:** docker-compose.yml for all services, .env for config; Supabase local mode available
- **Prod:** K8s/Helm for multi-node; cloud deployment (Vercel for frontend, AWS/GCP/Azure for backend/AI)
- **CI/CD:** On push/PR – lint/test/build (GitHub Actions)
- **Monitoring:** Grafana + Prometheus for health/metrics; Sentry for error tracking

---

## 7. Data Flow Example: Match Discovery
1. User (frontend) submits filtering/query
2. API Gateway passes JWT/authn, routes request
3. Matching Service: generates embedding, FAISS/Pinecone semantic search, scores multi-factor
4. Returns top-N matches to frontend
5. Frontend triggers LLM agent for "why you matched" explanation
6. Messages, notifications, audit events stored with full privacy controls

---

## 8. Evolution
- MVP requires only REST, PostgreSQL, Redis, basic FAISS
- Phase 2: Add event-driven message bus for cross-service orchestration, A/B testing
- Phase 3: Support for internationalization (i18n), multi-region deployment, compliance extensions

---

**References:**
- GeeksForGeeks, ZestMinds: Microservices and FastAPI (2025)
- Azure Architecture Center: Orchestration Patterns (2025)
- Leanware: Supabase best practices (2025)
- OpenAPI 3.1 Spec: https://swagger.io/specification/

---

**Next:** Database schema details (database-schema.md)