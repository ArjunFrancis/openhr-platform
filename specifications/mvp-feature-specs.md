# MVP Feature Specifications

**Task**: Day 5 - MVP Feature Specifications  
**Created**: November 30, 2025  
**Status**: Final

---

## Executive Summary

This document defines the **Minimum Viable Product (MVP)** for OpenHR Platform—the core features required to deliver value to co-founders and developers seeking startup opportunities. The MVP focuses on **profile creation, semantic skill matching, match discovery, and real-time messaging** to validate the platform's core value proposition: connecting compatible startup team members through AI-powered matching.

The MVP excludes advanced features like portfolio verification, endorsements, video calls, and complex personality assessments, which will be added in Phase 2 and Phase 3.

---

## MVP Scope & Timeline

### Phase 1: Core MVP (8-12 weeks)

**Goal**: Enable users to create profiles, discover quality matches, and initiate conversations.

**Success Criteria**:
- 100+ users signed up and profile complete
- 500+ matches generated
- 30%+ conversation rate (matches → messages sent)
- 40%+ 30-day retention rate

**Out of Scope for MVP**:
- Portfolio verification and trust system
- Endorsements and reputation scoring
- Video call integration
- Collaborative document sharing
- Advanced personality assessments (Big Five, MBTI)
- Equity split recommendation engine
- Mobile apps (iOS, Android)
- White-label licensing

---

## Core MVP Features

### 1. User Authentication & Onboarding

**Priority**: P0 (Blocker)  
**User Story**: As a new user, I want to sign up quickly using my GitHub or email so I can start finding co-founders or developers.

#### Feature Description

Users can sign up using:
- **GitHub OAuth** (preferred for developers)
- **Google OAuth** (for business co-founders)
- **Email + Password** (fallback)

**Auto-import from GitHub**: Username, avatar, bio, location, public repos

#### Acceptance Criteria

- [ ] User can sign up with GitHub OAuth in ≤3 clicks
- [ ] User can sign up with Google OAuth in ≤3 clicks
- [ ] User can sign up with email + password (validation: valid email, password ≥8 chars)
- [ ] After signup, user is redirected to onboarding flow
- [ ] GitHub OAuth auto-imports: username, avatar, bio, location
- [ ] User receives welcome email with profile completion checklist
- [ ] Supabase Auth session persists across browser sessions

#### Technical Implementation

**Tech Stack**: Supabase Auth, Next.js App Router, React Hook Form

**API Endpoints**:
- `POST /auth/signup` - Create new user account
- `POST /auth/login` - Login existing user
- `POST /auth/oauth/github` - GitHub OAuth callback
- `POST /auth/oauth/google` - Google OAuth callback

**Database Tables**:
- `users` (id, email, created_at, last_login)
- `profiles` (user_id, username, avatar_url, bio, location)

---

### 2. Progressive Profile Creation

**Priority**: P0 (Blocker)  
**User Story**: As a user, I want to complete my profile step-by-step so I can quickly get to my first match without feeling overwhelmed.

#### Feature Description

**Multi-step wizard** with progress indicator:
1. **Basic Info** (name, role, location, timezone)
2. **Skills & Experience** (primary skills, years of experience, tech stack)
3. **Startup Preferences** (stage, role type, equity vs salary, remote vs in-person)
4. **About You** (bio, vision, what you're looking for)

**GitHub Import**: Auto-extract skills from repos and contributions

**Skill Auto-Suggestions**: As user types skills, show suggestions from skill taxonomy

#### Acceptance Criteria

- [ ] User sees 4-step progress indicator (e.g., "Step 1 of 4")
- [ ] User can navigate back/forward between steps
- [ ] Each step has validation (e.g., at least 3 skills required)
- [ ] User can skip optional fields and complete later
- [ ] GitHub import button fetches top 10 repos and extracts skills
- [ ] Skill input shows auto-suggestions from taxonomy (React, Node.js, Python, etc.)
- [ ] User can save draft and return later
- [ ] Profile completeness score shown (e.g., "Profile 70% complete")
- [ ] After completing profile, user is redirected to match discovery

#### Technical Implementation

**Tech Stack**: Next.js, React Hook Form, Zod validation, Supabase PostgreSQL

**API Endpoints**:
- `POST /profiles` - Create new profile
- `PATCH /profiles/:id` - Update profile (incremental updates)
- `POST /profiles/:id/github-import` - Trigger GitHub import

**Database Tables**:
- `profiles` (user_id, role, location, timezone, bio, years_of_experience, profile_completeness)
- `user_skills` (user_id, skill_id, proficiency_level, source: manual|github|resume)
- `user_preferences` (user_id, startup_stage, role_type, equity_preference, remote_preference)

---

### 3. GitHub Profile Enrichment

**Priority**: P0 (Blocker)  
**User Story**: As a developer, I want to auto-import my skills from GitHub so I don't have to manually list every technology I've used.

#### Feature Description

When user connects their GitHub account:
1. Fetch top 10-20 public repos
2. Analyze repo languages and topics
3. Extract skills from README files and code
4. Normalize skills to canonical taxonomy (React.js → React)
5. Store skills with source = "github"

**User can review and approve/reject** suggested skills before adding to profile.

#### Acceptance Criteria

- [ ] User can connect GitHub account via OAuth
- [ ] System fetches top 10-20 repos ordered by stars/forks/activity
- [ ] System extracts skills from repo languages (e.g., JavaScript, Python, Go)
- [ ] System extracts skills from repo topics (e.g., react, nodejs, machine-learning)
- [ ] Skills are normalized to canonical taxonomy (TypeScript → TypeScript, not TS)
- [ ] User sees "Review Skills" modal with suggested skills
- [ ] User can approve all, reject individual skills, or add custom skills
- [ ] Approved skills are added to user profile with source = "github"
- [ ] GitHub import can be re-run to refresh skills

#### Technical Implementation

**Tech Stack**: GitHub API, FastAPI (Python), spaCy, skill taxonomy JSON

**API Endpoints**:
- `POST /enrichment/github-import` - Trigger GitHub import
- `GET /enrichment/github-import/:job_id` - Check import status

**Background Job**: Async job queue (Supabase Edge Functions or Railway worker)

**Database Tables**:
- `enrichment_jobs` (job_id, user_id, status, started_at, completed_at)
- `user_skills` (user_id, skill_id, source, confidence_score)

---

### 4. Semantic Skill Matching Algorithm

**Priority**: P0 (Blocker)  
**User Story**: As a user, I want to see matches who have complementary skills and shared interests, not just keyword overlaps.

#### Feature Description

**Semantic matching** using sentence transformers (all-mpnet-base-v2):
1. Generate embeddings for user's skill profile
2. Compute cosine similarity with all other users
3. Apply filters (location, remote preference, startup stage)
4. Score matches using multi-factor algorithm:
   - Skill fit (40%): Semantic similarity + complementary skill gaps
   - Timezone overlap (20%): Overlapping working hours
   - Equity preference alignment (20%): Both equity-heavy or both salary-focused
   - Startup stage match (20%): User seeks "seed" stage, match is building seed-stage startup

**Top 20 matches** returned per user, re-computed daily.

#### Acceptance Criteria

- [ ] User skill profile is embedded using sentence-transformers
- [ ] Cosine similarity computed between user and all other users
- [ ] Filters applied: location (if not remote), startup stage preference
- [ ] Match score combines: skill fit (40%), timezone (20%), equity (20%), stage (20%)
- [ ] Top 20 matches returned, ordered by score descending
- [ ] Matches exclude: users already matched, users blocked
- [ ] Match scores are cached and recomputed daily (not real-time)
- [ ] If user updates profile, matches recomputed within 24 hours

#### Technical Implementation

**Tech Stack**: Python FastAPI, Sentence Transformers, FAISS (vector search), PostgreSQL

**API Endpoints**:
- `POST /matching/compute-matches/:user_id` - Trigger match computation
- `GET /matching/matches/:user_id` - Fetch top 20 matches

**Background Job**: Daily cron job recomputes matches for all active users

**Database Tables**:
- `matches` (user_id, matched_user_id, score, skill_fit, timezone_overlap, created_at)
- `user_embeddings` (user_id, embedding_vector, updated_at)

---

### 5. Match Discovery Interface

**Priority**: P0 (Blocker)  
**User Story**: As a user, I want to browse my matches and understand why we're compatible so I can decide who to reach out to.

#### Feature Description

**Grid-based match discovery UI** (similar to LinkedIn or AngelList):
- Display top 20 matches in 2-column grid (desktop) or 1-column (mobile)
- Each match card shows:
  - Avatar, name, role (e.g., "Full-Stack Developer | Seeking Technical Co-Founder")
  - Location, timezone, years of experience
  - Top 3-5 skills with badges
  - Match score (e.g., "85% match")
  - "Why you matched" summary (e.g., "Both experienced in React, seeking seed-stage opportunities")
  - "View Profile" and "Send Message" buttons

**Filters**: Location, startup stage, role type, skills

#### Acceptance Criteria

- [ ] User sees grid of top 20 matches on dashboard
- [ ] Each match card displays: avatar, name, role, location, timezone, experience
- [ ] Match card shows top 3-5 skills as badges
- [ ] Match score displayed (e.g., "85% match")
- [ ] "Why you matched" summary generated using LLM (GPT-4 or Claude)
- [ ] User can click "View Profile" to see full profile modal
- [ ] User can click "Send Message" to open chat interface
- [ ] User can filter matches by: location, startup stage, role type
- [ ] User can search matches by skill keywords
- [ ] Match cards are responsive (2-column on desktop, 1-column on mobile)

#### Technical Implementation

**Tech Stack**: Next.js, React, TailwindCSS, Radix UI, Supabase Realtime

**API Endpoints**:
- `GET /matches/:user_id` - Fetch top 20 matches
- `GET /matches/:user_id/:matched_user_id/profile` - Fetch matched user's full profile

**LLM Integration**: Generate "Why you matched" using GPT-4 or Claude with prompt template

---

### 6. Real-Time Messaging

**Priority**: P0 (Blocker)  
**User Story**: As a user, I want to message my matches in real-time so I can start conversations and explore collaboration opportunities.

#### Feature Description

**In-app chat** with Supabase Realtime:
- **Conversation list**: Show all active conversations, sorted by most recent message
- **Chat interface**: Real-time messaging with typing indicators and read receipts
- **Message templates**: Pre-written intro messages ("Hi, I saw we're both building in fintech...")
- **Context panel**: Show matched user's profile sidebar while chatting

**Anti-spam**: Users can only message matches (no cold outreach to random users)

#### Acceptance Criteria

- [ ] User can see list of active conversations on left sidebar
- [ ] Clicking a conversation opens chat interface on right panel
- [ ] User can send text messages (max 1000 characters)
- [ ] Messages appear in real-time (no page refresh needed)
- [ ] Typing indicator shows when other user is typing
- [ ] Read receipts show when message has been read
- [ ] User can use message templates for first message
- [ ] User can see matched user's profile in right sidebar while chatting
- [ ] User can block or report a conversation
- [ ] User receives email notification for new messages (if opted in)
- [ ] Messages are stored in PostgreSQL and synced via Supabase Realtime

#### Technical Implementation

**Tech Stack**: Supabase Realtime, Next.js, React, PostgreSQL

**API Endpoints**:
- `GET /conversations/:user_id` - Fetch all conversations
- `GET /conversations/:conversation_id/messages` - Fetch messages (paginated)
- `POST /conversations/:conversation_id/messages` - Send new message
- `PATCH /messages/:message_id/read` - Mark message as read

**Supabase Realtime**: Subscribe to `messages` table with filter `conversation_id = ?`

**Database Tables**:
- `conversations` (id, user1_id, user2_id, created_at, last_message_at)
- `messages` (id, conversation_id, sender_id, content, read_at, created_at)

---

### 7. User Profile Pages

**Priority**: P1 (High)  
**User Story**: As a user, I want to view another user's full profile so I can learn more about their experience, skills, and startup interests before messaging them.

#### Feature Description

**Public profile page** with:
- Basic info: Avatar, name, role, location, timezone, years of experience
- About section: Bio and vision statement
- Skills section: All skills with proficiency badges (Beginner, Intermediate, Expert)
- Startup preferences: Stage, role type, equity vs salary, remote vs in-person
- GitHub profile link (if connected)
- "Send Message" button (if matched)

**Privacy**: Only matched users can see full profiles (non-matches see limited info)

#### Acceptance Criteria

- [ ] User can navigate to profile page via `/profile/:username`
- [ ] Profile page displays: avatar, name, role, location, timezone, experience
- [ ] Profile page shows full bio and vision statement
- [ ] Profile page shows all skills with proficiency badges
- [ ] Profile page shows startup preferences (stage, role, equity, remote)
- [ ] If user is matched, "Send Message" button is shown
- [ ] If user is not matched, limited info is shown (name, role, location only)
- [ ] GitHub profile link opens in new tab (if connected)
- [ ] Profile page is responsive (mobile-friendly)
- [ ] User can edit their own profile via "Edit Profile" button

#### Technical Implementation

**Tech Stack**: Next.js, React, TailwindCSS, Supabase PostgreSQL

**API Endpoints**:
- `GET /profiles/:username` - Fetch user profile
- `GET /profiles/:username/is-matched` - Check if current user is matched with this user

---

## Non-Functional Requirements

### Performance

- **Page Load Time**: ≤ 2 seconds on 3G connection
- **Match Computation**: ≤ 10 seconds for 10,000 users
- **Real-Time Messaging**: ≤ 500ms latency for message delivery
- **API Response Time**: 95th percentile ≤ 500ms

### Scalability

- **Concurrent Users**: Support 1,000 concurrent users
- **Database Capacity**: 100,000 users, 1M+ matches, 500K+ messages
- **Match Computation**: Batch process for 10K+ users nightly

### Security

- **Authentication**: OAuth 2.0 (GitHub, Google) + JWT tokens
- **Authorization**: Supabase Row-Level Security (RLS) policies
- **Data Encryption**: TLS 1.3 for data in transit, AES-256 for data at rest
- **Rate Limiting**: 100 requests/min per user, 1000 requests/min per IP
- **Anti-Spam**: Users can only message matches, not arbitrary users

### Privacy

- **GDPR Compliance**: User consent for data collection, data export/deletion workflows
- **Profile Privacy**: Non-matches see limited profile info
- **GitHub Data**: Only public repos and profile data fetched (no private data)
- **Message Privacy**: Messages encrypted in transit, stored encrypted at rest

### Accessibility

- **WCAG 2.1 Level AA** compliance
- **Keyboard Navigation**: All interactive elements accessible via keyboard
- **Screen Reader Support**: Proper ARIA labels and semantic HTML
- **Color Contrast**: Minimum 4.5:1 contrast ratio for text

---

## MVP Success Metrics (KPIs)

### Acquisition Metrics

- **Sign-ups**: 100+ users in first 4 weeks
- **Profile Completion Rate**: 70%+ users complete profile (80%+ completeness)
- **GitHub Connection Rate**: 60%+ developers connect GitHub

### Engagement Metrics

- **Time to First Match**: ≤ 24 hours after profile completion
- **Match Quality Score**: 50%+ users rate matches as "Good" or "Excellent"
- **Conversation Rate**: 30%+ of matches result in messages sent
- **Messages per Conversation**: Average 5+ messages per conversation

### Retention Metrics

- **7-Day Retention**: 50%+ users return within 7 days
- **30-Day Retention**: 40%+ users active after 30 days
- **Weekly Active Users (WAU)**: 60%+ of users active weekly

### Outcome Metrics

- **Co-Founder Partnerships Formed**: 5+ partnerships within first 8 weeks
- **Developer Placements**: 10+ developers find startup opportunities
- **User Satisfaction (NPS)**: Net Promoter Score ≥ 40

---

## Implementation Roadmap

### Week 1-2: Authentication & Onboarding

- [ ] Set up Supabase project and authentication
- [ ] Implement GitHub OAuth and Google OAuth
- [ ] Build email + password signup/login
- [ ] Create onboarding flow with 4-step wizard
- [ ] Implement profile creation form with validation

### Week 3-4: Profile Enrichment

- [ ] Integrate GitHub API for repo fetching
- [ ] Build skill extraction pipeline (Python FastAPI)
- [ ] Implement skill normalization with taxonomy
- [ ] Create "Review Skills" UI for user approval
- [ ] Store skills in `user_skills` table

### Week 5-6: Skill Matching Algorithm

- [ ] Set up sentence-transformers model (all-mpnet-base-v2)
- [ ] Build embedding generation pipeline
- [ ] Implement cosine similarity matching
- [ ] Add filters (location, stage, preferences)
- [ ] Create daily cron job for match computation

### Week 7-8: Match Discovery & Messaging

- [ ] Build match discovery grid UI
- [ ] Implement match card with "Why you matched" LLM
- [ ] Create profile modal for viewing matched users
- [ ] Build real-time chat interface with Supabase Realtime
- [ ] Implement message templates and typing indicators

### Week 9-10: Polish & Testing

- [ ] Add filters and search to match discovery
- [ ] Implement read receipts and notifications
- [ ] Mobile responsive design testing
- [ ] User acceptance testing (UAT) with beta users
- [ ] Bug fixes and performance optimizations

### Week 11-12: Launch Preparation

- [ ] SEO optimization (meta tags, sitemap)
- [ ] Analytics integration (PostHog, Mixpanel)
- [ ] Set up monitoring (Sentry, Supabase logs)
- [ ] Write launch blog post and press release
- [ ] Beta launch to 100 users (invite-only)

---

## Testing Strategy

### Unit Tests

- **Frontend**: Jest + React Testing Library
- **Backend**: Jest (Node.js), pytest (Python)
- **Coverage Target**: 80%+ code coverage

### Integration Tests

- **API Endpoints**: Supertest (Node.js)
- **Database**: Supabase test database
- **GitHub API**: Mock responses for CI/CD

### End-to-End Tests

- **Framework**: Playwright or Cypress
- **Critical Flows**: Signup → Onboarding → Match Discovery → Messaging
- **Frequency**: Run on every PR + nightly

### User Acceptance Testing (UAT)

- **Beta Users**: 20-30 co-founders and developers
- **Feedback Collection**: Google Forms + in-app surveys
- **Success Criteria**: 70%+ users rate experience as "Good" or "Excellent"

---

## Risks & Mitigations

### Risk 1: Low GitHub Connection Rate

**Impact**: Users don't connect GitHub, profile enrichment fails  
**Probability**: Medium  
**Mitigation**: Make GitHub connection optional but highly encouraged; allow manual skill entry; add LinkedIn import as alternative

### Risk 2: Poor Match Quality

**Impact**: Users get irrelevant matches, low conversation rate  
**Probability**: Medium  
**Mitigation**: Tune matching algorithm with user feedback; add filters for users to refine matches; implement "Not Interested" feedback loop

### Risk 3: Cold Start Problem

**Impact**: New users see no matches because platform has few users  
**Probability**: High  
**Mitigation**: Seed platform with 50-100 beta users before public launch; use invite-only model to ensure minimum user density

### Risk 4: Spam and Fake Profiles

**Impact**: Fake profiles reduce trust, spam messages annoy users  
**Probability**: Low (MVP only allows messaging between matches)  
**Mitigation**: Email verification required; GitHub OAuth encourages real profiles; users can report/block; manual review for suspicious accounts

---

## Post-MVP Features (Phase 2 & 3)

### Phase 2: AI Enhancement (Weeks 13-20)

- Resume parsing and auto-import
- LinkedIn profile enrichment
- Personality compatibility assessment (Big Five)
- LLM-powered intro message suggestions
- Match explanation improvements (detailed breakdown)

### Phase 3: Trust & Growth (Weeks 21-28)

- Portfolio verification (GitHub commits, website ownership)
- Skill endorsements from collaborators
- Reputation scoring and badges
- Referral program (invite friends, earn credits)
- Video call scheduling integration (Calendly, Google Meet)

### Phase 4: Scale & Monetization (Weeks 29+)

- Mobile apps (iOS, Android)
- White-label licensing for recruiters
- API access for third-party integrations
- Advanced analytics dashboard
- Enterprise plan for companies

---

## References

1. **Onboarding Flow Design**: [docs/specifications/onboarding-flow.md](onboarding-flow.md)
2. **Match Discovery UI**: [docs/specifications/match-discovery-ui.md](match-discovery-ui.md)
3. **Messaging UX**: [docs/specifications/messaging-ux.md](messaging-ux.md)
4. **System Architecture**: [docs/architecture/system-architecture.md](../docs/architecture/system-architecture.md)
5. **Database Schema**: [docs/architecture/database-schema.md](../docs/architecture/database-schema.md)
6. **Skill Matching Algorithms**: [docs/research/skill-matching-algorithms.md](../docs/research/skill-matching-algorithms.md)
7. **Real-Time Messaging**: [docs/platform/realtime-messaging.md](../docs/platform/realtime-messaging.md)

---

## Next Steps

1. **For Product Team**: Review and approve MVP scope, confirm success metrics
2. **For Design Team**: Create high-fidelity mockups for key screens (onboarding, match discovery, messaging)
3. **For Engineering Team**: Begin Week 1-2 implementation (authentication & onboarding)
4. **For Marketing Team**: Plan beta launch strategy (invite-only, 100 users)
5. **For Community Team**: Prepare contributor onboarding docs and issue templates

---

**Document Owner**: OpenHR Research & Planning Agent  
**Last Updated**: November 30, 2025  
**Version**: 1.0  
**Status**: Final
