# Day 3: System Architecture – Draft Outline

## 1. Executive Summary
This document outlines the recommended architecture and database schema for the OpenHR AI-powered talent matching platform, based on current industry best practices and research (2025). The design is tailored to support scalable, real-time, AI-driven co-founder and developer matching, with robust support for semantic search, recommendation, LLM agent orchestration, and privacy-by-design.

---

## 2. System Architecture Overview
- Modern **microservices-based architecture**
- Polyglot stack: **Node.js/Express** (API Gateway), **Python/FastAPI** (AI/ML services), **React** (Frontend)
- **Supabase/PostgreSQL** as the primary transactional database
- **Redis** for in-memory caching and session management
- **Vector DB** (FAISS, Pinecone) for semantic search
- **Kubernetes** for orchestration (optional at scale)
- **REST + OpenAPI** for all public/internal APIs, minimum OpenAPI 3.1 compliance
- **CI/CD** via GitHub Actions

## Diagram
```
       +----------------+         +---------------------+        +-------------+
       |   React Front  | <-----> |   API Gateway /     | <----> |  PostgreSQL  |
       |    (Next.js)   |  REST   |   Node.js/Express   |  SQL   +-------------+
       +----------------+         +---------------------+        | Supabase    |
                                                |               +-------------+
     +------------------------+           REST/GRPC                | Vector DB   |
     |  AI/ML Service Layer   |  <------------------------------> +-------------+
     |   (Python: FastAPI)    |                                   |  FAISS      |
     | - Embedding Service    |                                   |  Pinecone   |
     | - LLM API Adapter      |                                   +-------------+
     | - Score Engine         |                   | Redis |
     | - Skill Normalization  |    REST           +-------+
     +------------------------+                   |In-memory|
                                                  +-------+
```
---

## 3. Database Schema Design Principles
- **Normalization first:** Normalize for integrity; selectively denormalize for performance after query analysis.
- **Strong typing:** Use explicit types (including enums for status, roles, etc.)
- **Describe relationships:** Foreign keys for all logical links (users-profiles, profile-skills, etc.)
- **Row Level Security (RLS):** Enforce strict RLS per Supabase architecture
- **Multi-tenancy:** Support via org_id/tenant_id on relevant tables
- **Audit fields:** All primary tables include created_at, updated_at

---

## 4. Core Schema (Draft)
### users
```sql
id UUID PRIMARY KEY
email TEXT UNIQUE NOT NULL
hashed_password TEXT NOT NULL
role ENUM('user','admin','moderator')
status ENUM('active','invited','deactivated')
created_at TIMESTAMPTZ DEFAULT now()
updated_at TIMESTAMPTZ DEFAULT now()
```
### profiles
```sql
id UUID PRIMARY KEY (references users.id)
display_name TEXT
bio TEXT
location TEXT
github_url TEXT
linkedin_url TEXT
avatar_url TEXT
timezone TEXT
created_at TIMESTAMPTZ DEFAULT now()
updated_at TIMESTAMPTZ DEFAULT now()
```
### skills
```sql
id UUID PRIMARY KEY
canonical_name TEXT UNIQUE
parent_skill_id UUID
level ENUM('domain','area','group','specific')
description TEXT
created_at TIMESTAMPTZ DEFAULT now()
```
### profile_skills
```sql
profile_id UUID REFERENCES profiles(id)
skill_id UUID REFERENCES skills(id)
proficiency ENUM('beginner', 'intermediate', 'advanced', 'expert')
experience_years INTEGER
last_used DATE
PRIMARY KEY (profile_id, skill_id)
```
### startups
```sql
id UUID PRIMARY KEY
owner_id UUID REFERENCES users(id)
name TEXT
stage ENUM('idea','mvp','revenue','growth')
description TEXT
industry TEXT
location TEXT
created_at TIMESTAMPTZ DEFAULT now()
updated_at TIMESTAMPTZ DEFAULT now()
```
### startup_roles
```sql
id UUID PRIMARY KEY
startup_id UUID REFERENCES startups(id)
title TEXT
role_type ENUM('technical','business','product','design','other')
description TEXT
is_filled BOOLEAN
created_at TIMESTAMPTZ DEFAULT now()
```
### matches
```sql
id UUID PRIMARY KEY
candidate_profile_id UUID REFERENCES profiles(id)
startup_role_id UUID REFERENCES startup_roles(id)
score FLOAT
status ENUM('pending','accepted','rejected')
created_at TIMESTAMPTZ DEFAULT now()
updated_at TIMESTAMPTZ DEFAULT now()
```
### messages
```sql
id UUID PRIMARY KEY
match_id UUID REFERENCES matches(id)
sender_profile_id UUID REFERENCES profiles(id)
recipient_profile_id UUID REFERENCES profiles(id)
content TEXT
sent_at TIMESTAMPTZ DEFAULT now()
status ENUM('sent','delivered','read')
```

---

## 5. Caching & Optimization
- **Redis as cache-aside:** Use lazy-loading with expiry for commonly read entities (profiles, skills, matches)
- **Write-through for session/auth caches** if security demands always-fresh data
- **Partition large tables** by date (e.g., messages, audit logs)
- **Indexes**: Always index foreign keys & critical query columns

---

## 6. AI Agent Design Patterns (2025)
- **Task-Oriented (Autonomous) Agents:** Each agent (Skill Extraction, Matching, Match Explanation, Message Suggestion) is a stateless REST microservice
- **Event-Driven Collaboration:** Agents communicate via messaging/event bus for decoupled orchestration (RabbitMQ or Redis Streams, phase 2/3)
- **Multi-Agent Society:** Design for expansion to actor/crew-style multi-agent patterns as complexity grows
- **RAG Functions:** First-class integration for Retrieval-Augmented Generation pipelines for all LLM-facing agents
- **Agent Orchestrator:** Coordinator microservice for chaining agent outputs (workflow engine, e.g., Temporal.io, optional at scale)

---

## 7. API Specification
- **RESTful design, JSON/HTTP, OpenAPI 3.1+** (API documentation auto-generated with Swagger UI)
- API Gateway handles versioning, auth, rate-limits
- **Separation of concern**: Public API (frontend), Internal API (agent/service communication)
- **Endpoints for all resource types** (users, profiles, startups, matches, messages, skills)

---

## 8. DevOps & Deployment
- **Dockerized** for all microservices (Node.js API, FastAPI ML services, PostgreSQL, Redis, Vector DB)
- **Local developer workflow:** docker-compose, .env support
- **CI/CD:** GitHub Actions for build/test/deploy
- **Kubernetes (K8s):** Optional for scale-up, managed by Helm charts

---

## 9. Security, Scalability, Privacy
- **GDPR architecture:** No PII in logs, auditable data flows
- **RLS on all core tables**
- **API authentication:** JWT + OAuth2 for delegated auth
- **Agent-level authorization & service discovery**
- **Audit logging:** Separate, partitioned table
- **Monitoring:** Prometheus, Grafana, Sentry for log/error/metric aggregation
- **Multi-tenancy:** Tenant ID enforced on all access-controlled entities

---

*Detailed individual files for each section to follow:*
- system-architecture.md
- database-schema.md
- ai-agent-design.md
- api-specification.md
- tech-stack-justification.md

---

**Version: 0.9 (Draft)**
**Date: November 30, 2025**