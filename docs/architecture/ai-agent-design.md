# AI Agent Design for OpenHR

**Last updated:** November 30, 2025  
**Confidence:** 95%+ (based on 2025 agentic AI patterns research)

---

## 1. Overview

OpenHR implements a **multi-agent AI architecture** using 2025 agentic patterns for autonomous, collaborative task execution. Each agent is a stateless FastAPI microservice with defined roles, tools, and RAG pipelines. Agents communicate via REST or event bus (Phase 2) for orchestration.

**Key Patterns Applied:**
- **Task-Oriented Agents:** Single-responsibility for specific workflows (resume parsing, match scoring).
- **Event-Driven Collaboration:** Agents trigger each other via message bus for complex workflows.
- **RAG Integration:** All LLM-facing agents use retrieval-augmented generation with vector DB.
- **Orchestrator Pattern:** Central coordinator for multi-agent workflows (Temporal.io optional).

---

## 2. Core AI Agents

### 2.1 Profile Enrichment Agent
**Role:** Parse resumes, enrich profiles from GitHub/LinkedIn, generate bios/summaries.

**Architecture:**
- **Input:** Resume PDF/text, GitHub username, LinkedIn profile URL.
- **Tools:** LLM (GPT-4o/Llama 3), skill extraction (SBERT), GitHub API, resume parser (PyMuPDF).
- **Workflow:**
  1. Parse raw resume → extract structure (experience, skills, education).
  2. Normalize skills via taxonomy service.
  3. Generate bio summary using RAG (user history + skill taxonomy).
  4. Enrich with GitHub stats (stars, forks, contributions).
- **Output:** Structured profile JSON + LLM-generated bio.
- **Latency Target:** <5s per profile.
- **Cost:** $0.01-0.05 per enrichment (optimized).

**Implementation:** FastAPI endpoint `/enrich-profile` with async processing queue.

### 2.2 Match Scoring Agent
**Role:** Compute multi-factor match scores between profiles and startup roles.

**Architecture:**
- **Input:** Profile ID, startup role ID, user preferences.
- **Tools:** FAISS vector search, skill taxonomy, collaborative filtering (ALS), LLM for vision alignment.
- **Workflow:**
  1. **Skill Fit (40%):** Semantic similarity via SBERT embeddings (skills vector distance).
  2. **Complementary Skills (20%):** Detect gaps in startup role skills, score profile's unique value.
  3. **Vision Alignment (15%):** LLM compares profile bio + startup mission (cosine similarity).
  4. **Work Style (10%):** Questionnaire-based matching (remote preference, timezone, commitment).
  5. **Location/Timezone (5%):** Geographic + timezone compatibility scoring.
  6. **Equity/Stage (10%):** User preferences vs role requirements.
- **Output:** Match object with score breakdown (explainable).
- **Latency Target:** <100ms per match computation.
- **Scalability:** Batch processing for 100K+ profiles.

**Implementation:** FastAPI `/compute-match` with caching (Redis for recent scores).

### 2.3 Recommendation Agent
**Role:** Generate personalized match recommendations using hybrid filtering.

**Architecture:**
- **Input:** User ID, filters (location, role type, stage), cold-start flag.
- **Tools:** Two-tower neural network, FAISS for content-based, ALS for collaborative.
- **Workflow:**
  1. **Cold Start:** Content-based only (profile embeddings → FAISS search).
  2. **Warm Users:** Hybrid (FAISS top-1000 → ALS re-rank → contextual filter).
  3. **Diversity:** Ensure role type, industry, location diversity in top-10.
  4. **Explainability:** Generate factor contributions for each recommendation.
- **Output:** Ranked list of matches with scores and explanations.
- **Latency Target:** <200ms end-to-end.
- **Accuracy Target:** 85%+ user satisfaction (A/B tested).

**Implementation:** FastAPI `/recommend-matches` with pagination support.

### 2.4 Message Suggestion Agent
**Role:** Generate personalized icebreaker messages for new matches.

**Architecture:**
- **Input:** Match ID (profile + role details).
- **Tools:** LLM (fine-tuned for professional tone), RAG with match explanation.
- **Workflow:**
  1. Retrieve match details and explanation.
  2. Generate 3-5 message variants (professional, enthusiastic, question-based).
  3. Ensure GDPR compliance (no PII in prompts).
- **Output:** Array of suggested messages.
- **Latency Target:** <2s per suggestion set.
- **Conversion Impact:** +60% faster first message composition.

**Implementation:** FastAPI `/suggest-messages` triggered on match acceptance.

### 2.5 Verification Agent
**Role:** Validate profiles, detect fraud, verify GitHub/LinkedIn authenticity.

**Architecture:**
- **Input:** Profile ID for verification request.
- **Tools:** GitHub API, LinkedIn API, email verification, LLM for anomaly detection.
- **Workflow:**
  1. Verify email ownership.
  2. Check GitHub activity (commits > threshold, recent activity).
  3. Cross-validate LinkedIn connections/skills.
  4. LLM flags suspicious patterns (duplicate profiles, inconsistent data).
- **Output:** Verification status (verified, pending, flagged) + confidence score.
- **Latency Target:** <10s per verification.

**Implementation:** Background job triggered on profile creation/update.

---

## 3. Agent Orchestration

### 3.1 Single-Agent Calls
Most user interactions trigger one agent (e.g., `/enrich-profile` → Profile Enrichment Agent).

### 3.2 Multi-Agent Workflows
Complex flows chain agents:
- **Onboarding:** Profile Enrichment → Skill Normalization → Recommendation Generation.
- **Match Discovery:** Recommendation Agent → Match Scoring → Message Suggestion.
- **Profile Update:** Verification Agent → Re-compute matches → Update recommendations.

**Orchestrator:** Central FastAPI service routes requests and coordinates via async tasks (Celery/RQ).

### 3.3 Event-Driven (Phase 2)
- Use Redis Streams or RabbitMQ for agent-to-agent communication.
- Events: `profile_updated`, `new_match`, `message_sent` trigger relevant agents.
- Decouples services for better scalability and fault tolerance.

---

## 4. Technical Implementation

**Agent Base Class:** All agents extend a common Python class with:
- Input validation (Pydantic).
- Logging with correlation IDs.
- Error handling and retries.
- Metrics collection (Prometheus).

**Example Structure:**
```python
class BaseAgent:
    def __init__(self, config: AgentConfig):
        self.llm = LLMClient(config.llm_model)
        self.vector_db = VectorDBClient(config.vector_endpoint)
        self.taxonomy = TaxonomyService()

    async def execute(self, input_data: dict) -> dict:
        # Validate, process, return
        pass

class ProfileEnrichmentAgent(BaseAgent):
    async def execute(self, resume_data: dict) -> ProfileData:
        # Specific implementation
        pass
```

**Deployment:** Each agent as separate FastAPI service in Docker/K8s.

---

## 5. RAG Pipeline Integration

All LLM agents use standardized RAG:
1. **Retrieval:** Query vector DB with user query/profile embedding.
2. **Augmentation:** Inject top-K relevant documents (skills, past matches, taxonomy).
3. **Generation:** LLM generates response grounded in retrieved context.
4. **Validation:** Post-process for hallucinations, PII, toxicity.

**Vector Sources:**
- Skill taxonomy embeddings.
- User interaction history.
- Startup mission statements.
- Industry best practices.

---

## 6. Monitoring & Observability

- **Metrics:** Latency, success rate, token usage per agent (Prometheus).
- **Tracing:** OpenTelemetry for end-to-end request tracing across agents.
- **Logging:** Structured logs with agent names, input/output hashes.
- **Alerts:** PagerDuty for agent failures >5% error rate.

**Cost Tracking:**
- Track LLM token usage per agent and user.
- Alert on cost spikes (>20% MoM).
- Cache common responses (explanations, suggestions).

---

## 7. Evolution & Scaling

**MVP (Phase 1):** 3 core agents (Profile, Match, Recommendation) as REST endpoints.
**Phase 2:** Add event-driven orchestration, Verification + Message agents.
**Phase 3:** Multi-agent crews for complex workflows (e.g., interview prep, negotiation support).

**Scaling Strategy:**
- Horizontal scaling per agent type.
- Async queues for bursty workloads.
- Model distillation (fine-tune smaller models per agent).

**References:**
- Azure Architecture Center: AI Agent Orchestration Patterns [web:222]
- Agent Design Pattern Catalogue (arXiv) [web:216]
- 5 Agentic AI Design Patterns (Shakudo, 2025) [web:226]
- Multi-Agent Architectures Survey (arXiv, 2024) [web:215]