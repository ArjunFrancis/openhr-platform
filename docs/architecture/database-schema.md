# Database Schema for OpenHR

Last updated: November 30, 2025  
Confidence: 95%+

---

## 1. Design Goals

- Support co-founder/developer–startup matching, messaging, and profile enrichment.
- First-class support for semantic skills, preferences, matches, and LLM artifacts.
- Optimized for PostgreSQL/Supabase with Row Level Security (RLS) and multi-tenancy.

Key principles:
- Third Normal Form (3NF) for core entities, selective denormalization for read paths.
- Explicit foreign keys and indexes on all relationships.
- Soft-deletion where necessary; minimal PII and privacy by design.

---

## 2. Core Entities

### 2.1 users
Represents login identity and account-level settings.

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  hashed_password TEXT NOT NULL,
  role TEXT NOT NULL CHECK (role IN ('user','admin','moderator')),
  status TEXT NOT NULL CHECK (status IN ('active','invited','disabled')) DEFAULT 'active',
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_users_email ON users(email);
```

### 2.2 profiles
1–1 with users; stores visible user information and preferences.

```sql
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  display_name TEXT NOT NULL,
  bio TEXT,
  location TEXT,
  timezone TEXT,
  github_url TEXT,
  linkedin_url TEXT,
  website_url TEXT,
  avatar_url TEXT,
  looking_for_role TEXT, -- 'technical_cofounder','business_cofounder','founding_engineer', etc.
  open_to_relocation BOOLEAN DEFAULT FALSE,
  remote_preference TEXT CHECK (remote_preference IN ('remote_first','hybrid','onsite','any')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_profiles_location ON profiles(location);
CREATE INDEX idx_profiles_remote_pref ON profiles(remote_preference);
```

---

## 3. Skill & Taxonomy Tables

### 3.1 skills
Canonical skills from ESCO + Stack Overflow augmentation.

```sql
CREATE TABLE skills (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  canonical_name TEXT UNIQUE NOT NULL,
  description TEXT,
  domain TEXT,        -- Level 1
  area TEXT,          -- Level 2
  skill_group TEXT,   -- Level 3
  level TEXT CHECK (level IN ('domain','area','group','specific')) DEFAULT 'specific',
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_skills_domain ON skills(domain);
CREATE INDEX idx_skills_area ON skills(area);
CREATE INDEX idx_skills_group ON skills(skill_group);
```

### 3.2 skill_variants
Maps variants/aliases ("react.js", "ReactJS") to canonical skills.

```sql
CREATE TABLE skill_variants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  skill_id UUID NOT NULL REFERENCES skills(id) ON DELETE CASCADE,
  variant_name TEXT NOT NULL UNIQUE
);
CREATE INDEX idx_skill_variants_skill_id ON skill_variants(skill_id);
CREATE INDEX idx_skill_variants_name ON skill_variants(variant_name);
```

### 3.3 skill_relationships
Stores parent/child, sibling, related, prerequisite links.

```sql
CREATE TABLE skill_relationships (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  skill_id UUID NOT NULL REFERENCES skills(id) ON DELETE CASCADE,
  related_skill_id UUID NOT NULL REFERENCES skills(id) ON DELETE CASCADE,
  relationship_type TEXT NOT NULL CHECK (
    relationship_type IN ('parent','child','sibling','related','prerequisite')
  ),
  weight REAL DEFAULT 1.0,
  UNIQUE (skill_id, related_skill_id, relationship_type)
);
CREATE INDEX idx_skill_rel_skill ON skill_relationships(skill_id);
CREATE INDEX idx_skill_rel_related ON skill_relationships(related_skill_id);
CREATE INDEX idx_skill_rel_type ON skill_relationships(relationship_type);
```

### 3.4 profile_skills
Many–many between profiles and skills, with proficiency.

```sql
CREATE TABLE profile_skills (
  profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  skill_id UUID NOT NULL REFERENCES skills(id) ON DELETE CASCADE,
  proficiency TEXT CHECK (proficiency IN ('beginner','intermediate','advanced','expert')),
  experience_years INTEGER CHECK (experience_years >= 0),
  last_used DATE,
  PRIMARY KEY (profile_id, skill_id)
);
CREATE INDEX idx_profile_skills_profile_id ON profile_skills(profile_id);
CREATE INDEX idx_profile_skills_skill_id ON profile_skills(skill_id);
```

---

## 4. Startup & Role Modeling

### 4.1 startups
Represents a startup or project seeking co-founders/developers.

```sql
CREATE TABLE startups (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  owner_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  slug TEXT UNIQUE,
  description TEXT,
  mission TEXT,
  industry TEXT,
  stage TEXT CHECK (stage IN ('idea','mvp','revenue','growth')),
  website_url TEXT,
  location TEXT,
  remote_friendly BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_startups_owner_id ON startups(owner_id);
CREATE INDEX idx_startups_stage ON startups(stage);
CREATE INDEX idx_startups_location ON startups(location);
```

### 4.2 startup_roles
Represents specific roles a startup wants to fill.

```sql
CREATE TABLE startup_roles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  startup_id UUID NOT NULL REFERENCES startups(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  role_type TEXT CHECK (role_type IN ('technical','business','product','design','other')),
  description TEXT,
  responsibilities TEXT,
  equity_min REAL,
  equity_max REAL,
  time_commitment TEXT,  -- 'full_time','part_time','nights_weekends'
  is_filled BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_startup_roles_startup_id ON startup_roles(startup_id);
CREATE INDEX idx_startup_roles_role_type ON startup_roles(role_type);
```

### 4.3 startup_role_skills
Required skills for a given role.

```sql
CREATE TABLE startup_role_skills (
  startup_role_id UUID NOT NULL REFERENCES startup_roles(id) ON DELETE CASCADE,
  skill_id UUID NOT NULL REFERENCES skills(id) ON DELETE CASCADE,
  importance INTEGER CHECK (importance BETWEEN 1 AND 5) DEFAULT 3,
  PRIMARY KEY (startup_role_id, skill_id)
);
CREATE INDEX idx_startup_role_skills_role_id ON startup_role_skills(startup_role_id);
CREATE INDEX idx_startup_role_skills_skill_id ON startup_role_skills(skill_id);
```

---

## 5. Matching & Recommendation Tables

### 5.1 matches
Stores match instances between a profile and a startup_role.

```sql
CREATE TABLE matches (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  candidate_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  startup_role_id UUID NOT NULL REFERENCES startup_roles(id) ON DELETE CASCADE,
  score REAL NOT NULL,  -- 0-1 or 0-100 normalized score
  status TEXT NOT NULL CHECK (status IN ('suggested','liked','mutual','rejected','hidden')) DEFAULT 'suggested',
  explanation_id UUID, -- optional FK to llm_explanations
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (candidate_profile_id, startup_role_id)
);
CREATE INDEX idx_matches_candidate ON matches(candidate_profile_id);
CREATE INDEX idx_matches_role ON matches(startup_role_id);
CREATE INDEX idx_matches_status ON matches(status);
```

### 5.2 match_scores_breakdown
Stores factor-level contributions to a match score (for explainability).

```sql
CREATE TABLE match_scores_breakdown (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  match_id UUID NOT NULL REFERENCES matches(id) ON DELETE CASCADE,
  factor TEXT NOT NULL,        -- 'skill_fit','complementary','vision','work_style','timezone','equity','stage'
  weight REAL NOT NULL,
  raw_score REAL NOT NULL,
  contribution REAL NOT NULL   -- weight * raw_score
);
CREATE INDEX idx_match_scores_match_id ON match_scores_breakdown(match_id);
```

### 5.3 recommendations_log
Audit log for recommendations shown to users (for debugging/metrics).

```sql
CREATE TABLE recommendations_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  match_id UUID NOT NULL REFERENCES matches(id) ON DELETE CASCADE,
  position INTEGER NOT NULL,      -- position in ranked list
  algorithm_version TEXT NOT NULL,
  viewed_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_reco_log_user ON recommendations_log(user_id);
```

---

## 6. Messaging & Collaboration

### 6.1 conversations
Thread entity between two (or more in future) profiles.

```sql
CREATE TABLE conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  started_by_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  startup_role_id UUID REFERENCES startup_roles(id) ON DELETE SET NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_conversations_starter ON conversations(st_started_by_profile_id);
```

### 6.2 conversation_participants
Supports multi-party in future; currently usually 2 participants.

```sql
CREATE TABLE conversation_participants (
  conversation_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
  profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  PRIMARY KEY (conversation_id, profile_id)
);
CREATE INDEX idx_conv_participant_profile ON conversation_participants(profile_id);
```

### 6.3 messages
Individual chat messages.

```sql
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
  sender_profile_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  content TEXT NOT NULL,
  sent_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  status TEXT NOT NULL CHECK (status IN ('sent','delivered','read')) DEFAULT 'sent'
);
CREATE INDEX idx_messages_conversation ON messages(conversation_id);
CREATE INDEX idx_messages_sender ON messages(sender_profile_id);
CREATE INDEX idx_messages_status ON messages(status);
```

---

## 7. LLM & AI Artifacts

### 7.1 llm_explanations
Stores generated explanations for matches (optional cache).

```sql
CREATE TABLE llm_explanations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  match_id UUID NOT NULL REFERENCES matches(id) ON DELETE CASCADE,
  model_name TEXT NOT NULL,
  prompt_hash TEXT NOT NULL,
  explanation TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (match_id, model_name, prompt_hash)
);
CREATE INDEX idx_llm_expl_match_id ON llm_explanations(match_id);
```

### 7.2 resume_parsing_jobs
Tracks async resume parsing tasks.

```sql
CREATE TABLE resume_parsing_jobs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  status TEXT NOT NULL CHECK (status IN ('queued','processing','completed','failed')) DEFAULT 'queued',
  source TEXT,                -- 'upload','github','linkedin'
  error_message TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_resume_jobs_user ON resume_parsing_jobs(user_id);
CREATE INDEX idx_resume_jobs_status ON resume_parsing_jobs(status);
```

### 7.3 resumes
Stores parsed resume structure (not raw files) to respect privacy.

```sql
CREATE TABLE resumes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  headline TEXT,
  summary TEXT,
  raw_text TEXT, -- optional, can be truncated or removed after parsing
  parsed_json JSONB, -- structured data (education, experience, skills)
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_resumes_user ON resumes(user_id);
```

---

## 8. Analytics & Audit

### 8.1 audit_logs
Generic audit trail for critical actions.

```sql
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE SET NULL,
  action TEXT NOT NULL,      -- 'login','profile_update','match_accept','message_send', etc.
  entity_type TEXT,
  entity_id UUID,
  metadata JSONB,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_audit_logs_user ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_action ON audit_logs(action);
CREATE INDEX idx_audit_logs_entity ON audit_logs(entity_type, entity_id);
```

---

## 9. Indexing & Performance

General guidelines:
- Index all foreign keys: user_id, profile_id, startup_id, startup_role_id, skill_id, match_id.
- Use partial indexes where appropriate (e.g., open roles, active users).
- Avoid over-indexing write-heavy tables (messages, audit_logs) – consider partitioning by month.

Example partial index:
```sql
CREATE INDEX idx_matches_mutual
  ON matches (startup_role_id)
  WHERE status = 'mutual';
```

---

## 10. Supabase & RLS Considerations

- Enable RLS on users, profiles, matches, conversations, messages.
- Policies enforce: users can only see their own profile, messages, and matches where they are participant.

Example RLS policy (conceptual):
```sql
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

CREATE POLICY profiles_select_own
ON profiles FOR SELECT
USING (id = auth.uid());
```

Multi-tenancy (future):
- Add tenant_id/org_id to startups, startup_roles, profiles if needed.
- RLS policies filter by tenant_id for enterprise use cases.

---

This schema provides a production-ready foundation for the OpenHR platform and aligns with the Day 2 AI/ML strategy documents (skill matching, taxonomy, recommendation engine, LLM use-cases).