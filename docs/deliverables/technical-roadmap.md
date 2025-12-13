# OpenHR Technical Roadmap (12-Month Implementation)

**Date:** December 13, 2025  
**Status:** Implementation-Ready  
**Audience:** Engineering Leaders, Developers, Technical Teams

---

## Executive Summary

This roadmap breaks down OpenHR's development into 4 phases over 12 months, prioritizing features that deliver the most value to users fastest. We follow a "ship fast, iterate based on feedback" philosophy rather than building perfect-first versions. Each phase includes specific sprints, milestones, and success metrics.

**Key principle:** Ship MVP in 8 weeks, then iterate in 2-week cycles based on user feedback.

---

## Phase 1: MVP Launch (Weeks 1-8)

**Goal:** Minimal viable product with core matching engine, onboarding, and messaging

**Timeline:** January 1 - February 28, 2025  
**Team:** 2-3 engineers + 1 product person  
**Success Metric:** 500+ signups, 40%+ day-7 retention, first 10 successful matches

### Architecture Foundation (Week 1)

**Deliverables:**
- [ ] Backend API structure (Express.js + TypeScript)
- [ ] Database schema (Supabase)
- [ ] Frontend boilerplate (Next.js + React)
- [ ] Dev environment setup (Docker, Git workflow)
- [ ] CI/CD pipeline (GitHub Actions)
- [ ] Monitoring setup (Sentry, LogRocket)

**Effort:** 2-3 days (use templates/boilerplates)

### Authentication & User Profiles (Week 1-2)

**Backend:**
```typescript
// API endpoints
POST   /auth/signup          // User registration
POST   /auth/login           // Login
GET    /auth/github/callback // GitHub OAuth
GET    /api/users/:id        // User profile
PUT    /api/users/:id        // Update profile
GET    /api/users/:id/skills // List skills
```

**Frontend:**
- [ ] Signup form (email, password)
- [ ] GitHub OAuth button
- [ ] Profile edit page
- [ ] Skill input (search + dropdown)

**Deliverables:**
- [ ] User registration flow
- [ ] GitHub OAuth integration
- [ ] Profile data model (Supabase)
- [ ] Skill input component

**Effort:** 3-4 days

### Profile Enrichment v1 (Week 2-3)

**Start with GitHub only (simple):**

**Backend:**
```typescript
// Fetch GitHub data when OAuth completes
async function enrichFromGitHub(githubToken) {
  const repos = await github.getRepos(githubToken);
  const languages = extractLanguages(repos);
  const skills = mapLanguagesToSkills(languages);
  return { skills, confidence: 0.8 };
}
```

**Flow:**
1. User clicks "Connect GitHub"
2. OAuth redirects to GitHub
3. We fetch their public repos
4. Extract languages + topics
5. Map to skill taxonomy
6. Display "✅ GitHub connected! We found X skills"

**Deliverables:**
- [ ] GitHub API integration
- [ ] Language extraction from repos
- [ ] Skill mapping logic
- [ ] UI to show extracted skills

**Effort:** 4-5 days

### Skill Matching Engine (Week 3-4)

**Start with simple cosine similarity:**

```python
# backend/ml/matching.py
from sentence_transformers import SentenceTransformer
import numpy as np

model = SentenceTransformer('all-mpnet-base-v2')

def find_matches(user_id, user_skills):
    """Find users with complementary skills."""
    
    # Embed user skills
    user_embedding = model.encode(user_skills).mean(axis=0)
    
    # Get all other users
    all_users = get_all_users(exclude=user_id)
    
    # Score each user
    scores = []
    for other_user in all_users:
        other_embedding = model.encode(other_user.skills).mean(axis=0)
        
        # Cosine similarity
        similarity = np.dot(user_embedding, other_embedding) / \
                     (np.linalg.norm(user_embedding) * np.linalg.norm(other_embedding))
        
        scores.append({
            'user_id': other_user.id,
            'score': similarity,
            'reason': generate_match_reason(user_skills, other_user.skills)
        })
    
    # Return top 10
    return sorted(scores, key=lambda x: x['score'], reverse=True)[:10]

def generate_match_reason(skills1, skills2):
    """Generate human-readable match explanation."""
    common = set(skills1) & set(skills2)
    complementary = set(skills1) ^ set(skills2)
    
    if common:
        return f"You both know {', '.join(list(common)[:2])}"
    else:
        return "Complementary skills"
```

**API:**
```
GET /api/matches
Response: [
  {
    "id": "user123",
    "name": "John",
    "skills": ["React", "Node.js"],
    "score": 0.87,
    "reason": "Both React experts + complementary backend/frontend"
  },
  ...
]
```

**Frontend:**
- [ ] Browse matches page (card-based)
- [ ] Match details modal (name, skills, reason)
- [ ] "Connect" button per match

**Deliverables:**
- [ ] Sentence Transformers setup
- [ ] Matching algorithm (cosine similarity)
- [ ] Match scoring logic
- [ ] UI to display matches

**Effort:** 4-5 days

### Real-Time Messaging (Week 4-5)

**Using Supabase Realtime:**

```typescript
// backend/services/messaging.ts
import { supabase } from '../config/supabase';

class MessagingService {
  async sendMessage(senderId, recipientId, text) {
    return await supabase.from('messages').insert([
      {
        sender_id: senderId,
        recipient_id: recipientId,
        text,
        created_at: new Date(),
        read_at: null
      }
    ]);
  }
  
  subscribeToMessages(userId) {
    return supabase
      .from(`messages:recipient_id=eq.${userId}`)
      .on('INSERT', (payload) => {
        console.log('New message:', payload.new);
        // Update UI
      })
      .subscribe();
  }
}
```

**Frontend:**
```tsx
// frontend/components/Chat.tsx
const Chat = ({ recipientId }) => {
  const [messages, setMessages] = useState([]);
  
  useEffect(() => {
    // Subscribe to real-time updates
    const subscription = messagesService.subscribeToMessages(currentUserId);
    return () => subscription.unsubscribe();
  }, []);
  
  const sendMessage = (text) => {
    messagesService.sendMessage(currentUserId, recipientId, text);
  };
  
  return (
    <div className="chat">
      <div className="messages">
        {messages.map(m => <Message key={m.id} message={m} />)}
      </div>
      <MessageInput onSend={sendMessage} />
    </div>
  );
};
```

**Deliverables:**
- [ ] Database schema for messages
- [ ] Messaging API (POST, GET)
- [ ] Supabase Realtime subscription
- [ ] Chat UI component
- [ ] Message list + input

**Effort:** 3-4 days

### Onboarding Flow (Week 5-6)

**Goal:** Get user to first match in <5 minutes

**Steps:**
1. Sign up with email
2. Create profile (name, bio, role)
3. Add skills (3-5 skills minimum)
4. Optional: Connect GitHub
5. See matches
6. View first match details
7. Send message

**Frontend Pages:**
- [ ] /signup
- [ ] /onboarding/profile
- [ ] /onboarding/skills
- [ ] /onboarding/github
- [ ] /matches
- [ ] /matches/[id] (detail view)
- [ ] /messages

**Deliverables:**
- [ ] Step-by-step onboarding flow
- [ ] Progress indicator
- [ ] Ability to skip steps
- [ ] Post-onboarding redirect to matches

**Effort:** 4-5 days

### Testing & Bug Fixes (Week 6-8)

**Priority 1: Core flows**
- [ ] Signup → Match viewing → Messaging
- [ ] Data integrity (no lost messages, profile data)
- [ ] Auth security (no unauthorized access)

**Priority 2: Performance**
- [ ] Onboarding page <2s load
- [ ] Match loading <1s
- [ ] Message send <500ms latency

**Priority 3: Mobile experience**
- [ ] Responsive design (mobile-first)
- [ ] Touch-friendly buttons
- [ ] Works on iOS + Android

**Testing:**
- [ ] Unit tests (core functions)
- [ ] Integration tests (API flows)
- [ ] E2E tests (full user journey)
- [ ] Load testing (100 concurrent users)

**Launch Preparation:**
- [ ] Error tracking (Sentry)
- [ ] Performance monitoring (LogRocket)
- [ ] Incident response plan
- [ ] Documentation

**Deliverables:**
- [ ] All tests passing
- [ ] <1% error rate
- [ ] 99.5%+ uptime
- [ ] Launch-ready code

**Effort:** 5-7 days

---

## Phase 2: Early Growth & Iteration (Weeks 9-16)

**Goal:** Reach 2000+ signups, validate product-market fit, incorporate feedback

**Timeline:** March 1 - April 30, 2025  
**Team:** 2-3 engineers + 1 product/growth person  
**Success Metric:** 40%+ day-7 retention, 50+ successful co-founder connections

### Sprint 9-10: Profile Enrichment v2

**Add resume parsing:**

```python
# backend/ml/resume_parser.py
from llm_service import OpenAI
import PyPDF2

async def parse_resume(file_path):
    """Extract skills from resume using GPT-4."""
    
    # Extract text from PDF
    text = extract_pdf_text(file_path)
    
    # Send to GPT-4
    prompt = f"""
    Extract the following from this resume in JSON:
    - skills (list of technical skills)
    - experience (list of past roles with years)
    - education (degree, school)
    
    Resume: {text}
    
    Return ONLY valid JSON.
    """
    
    response = await openai.ChatCompletion.acreate(
        model='gpt-4',
        messages=[{'role': 'user', 'content': prompt}],
        temperature=0.1
    )
    
    return json.loads(response.choices[0].message.content)

async def handle_resume_upload(user_id, file):
    """User uploads resume -> extract skills."""
    file_path = save_file(file)
    extracted = await parse_resume(file_path)
    
    # Merge with existing skills
    existing_skills = get_user_skills(user_id)
    merged_skills = merge_skills(existing_skills, extracted['skills'])
    
    update_user_skills(user_id, merged_skills, source='resume')
```

**Deliverables:**
- [ ] Resume upload endpoint
- [ ] PDF text extraction
- [ ] LLM-based skill extraction
- [ ] Skill merging logic
- [ ] Resume upload UI

**Effort:** 5-6 days

### Sprint 11-12: Match Quality v2

**Improve matching with more signals:**

```python
def score_match_v2(user1_id, user2_id):
    """Score match with multiple factors."""
    
    user1 = get_user(user1_id)
    user2 = get_user(user2_id)
    
    score = 0.0
    
    # Factor 1: Skill complementarity (40%)
    skill_score = calculate_skill_complement(user1.skills, user2.skills)
    score += skill_score * 0.4
    
    # Factor 2: Experience level alignment (20%)
    exp_gap = abs(user1.years_experience - user2.years_experience)
    exp_score = 1.0 - min(exp_gap / 10, 1.0)  # Perfect match at same level
    score += exp_score * 0.2
    
    # Factor 3: Location/timezone (15%)
    tz_distance = calculate_timezone_distance(user1.timezone, user2.timezone)
    tz_score = 1.0 - min(tz_distance / 12, 1.0)  # 12h apart = 0 score
    score += tz_score * 0.15
    
    # Factor 4: Startup stage preference (15%)
    stage_match = 1.0 if user1.startup_stage == user2.startup_stage else 0.5
    score += stage_match * 0.15
    
    # Factor 5: Equity preference alignment (10%)
    equity_match = 1.0 if abs(user1.equity_seek - user2.equity_offer) < 10 else 0.5
    score += equity_match * 0.1
    
    return score
```

**Deliverables:**
- [ ] Multi-factor scoring algorithm
- [ ] Location/timezone matching
- [ ] Experience level alignment
- [ ] Startup stage filtering
- [ ] Preference-based filtering

**Effort:** 5-6 days

### Sprint 13-14: Messaging Features

**Add messaging quality features:**

```typescript
// Message suggestions based on context
function generateMessageSuggestion(user1, user2) {
  const commonSkills = intersection(user1.skills, user2.skills);
  const complementarySkills = difference(user1.skills, user2.skills);
  
  if (commonSkills.length > 0) {
    return `Hi! I noticed we both know ${commonSkills[0]}. Would love to chat about co-founding together.`;
  } else if (complementarySkills.length > 0) {
    return `Hey! I love that you're strong in ${complementarySkills[0]}. That's exactly what I'm looking for in a co-founder.`;
  }
  
  return `Hi! I think we could be a great founding team. Interested in chatting?`;
}
```

**Deliverables:**
- [ ] Message templates
- [ ] Smart message suggestions
- [ ] Read receipts
- [ ] Typing indicators
- [ ] Message notifications via email
- [ ] "Last seen" timestamp

**Effort:** 4-5 days

### Sprint 15-16: Analytics & Iteration

**Set up dashboards to track metrics:**

```typescript
// Track key events
analytics.track('user_signup', { source: 'producthunt' });
analytics.track('profile_completed', { time_to_complete_minutes: 5 });
analytics.track('match_viewed', { match_id, score: 0.87 });
analytics.track('message_sent', { recipient_id });
analytics.track('connection_made', { user1_id, user2_id });
```

**Dashboards:**
- [ ] Daily active users (DAU)
- [ ] Signup funnel (Signup → Profile → Match view → Message)
- [ ] Retention cohorts (day-1, day-7, day-30)
- [ ] Match quality metrics (message rate, connection rate)
- [ ] Feature usage (GitHub connections, resume uploads)

**Deliverables:**
- [ ] Event tracking infrastructure
- [ ] Analytics dashboards
- [ ] Conversion funnel analysis
- [ ] User feedback collection (in-app survey)

**Effort:** 4-5 days

---

## Phase 3: Scale & Polish (Weeks 17-28)

**Goal:** Reach 10K+ signups, prepare for partnerships, add missing features

**Timeline:** May 1 - August 31, 2025  
**Team:** 3-4 engineers + 1-2 product/growth  
**Success Metric:** 10K+ signups, 500+ successful partnerships, 50%+ day-30 retention

### Partnership Integrations (Weeks 17-20)

**Enable Y Combinator integration:**

```typescript
// Feature: "YC Badge" on profile
function applyYCBadge(user_id, yc_batch) {
  update_user({ badges: ['yc:s24'] });
  // Now matches show: "YC S24 Founder"
  // Profile shows batch year
  // Can filter by YC status
}

// Feature: YC-exclusive matches
function get_yc_matches(user_id) {
  const user = get_user(user_id);
  if (!user.yc_batch) return [];
  
  // Return only YC founders
  return find_matches(user_id).filter(m => m.user.yc_batch);
}
```

**Deliverables:**
- [ ] YC OAuth integration
- [ ] YC badge/verification
- [ ] YC-exclusive features
- [ ] Batch-based matching
- [ ] YC partner dashboard

**Effort:** 5-6 days per partnership

### LinkedIn Integration (Weeks 21-22)

**Similar to GitHub:**
- [ ] LinkedIn OAuth
- [ ] Fetch experience, education, endorsements
- [ ] Extract skills from experience
- [ ] LinkedIn badge on profile

**Effort:** 4-5 days

### Advanced Matching (Weeks 23-24)

**Add personality compatibility:**

```python
# Brief personality quiz (5 questions)
quiz = [
    "How do you handle disagreement with co-founder?",  # Conflict style
    "What's your risk tolerance?",  # Risk
    "Startup stage preference?",  # Stage
    "Equity expectations?",  # Compensation
    "Work style?",  # Remote vs office
]

# Match personality vectors
def score_personality_match(user1, user2):
    # Cosine similarity on personality vectors
    similarity = cosine_similarity(
        user1.personality_embedding,
        user2.personality_embedding
    )
    return similarity  # 0-1 score
```

**Deliverables:**
- [ ] Personality quiz
- [ ] Personality vector embeddings
- [ ] Personality-based matching
- [ ] Combined skill + personality scoring

**Effort:** 5-6 days

### White-Label MVP (Weeks 25-28)

**Enable accelerators to white-label OpenHR:**

```typescript
// White-label API
GET /api/white-label/:brand/config
// Returns: {
//   name: "Techstars Founder Matching",
//   logo_url: "https://...",
//   colors: { primary: "#...", secondary: "#..." },
//   domain: "techstars.openhr.com"
// }
```

**Deliverables:**
- [ ] White-label config system
- [ ] Branding customization
- [ ] Custom domain support
- [ ] Partner analytics dashboard
- [ ] Revenue tracking per partner

**Effort:** 6-8 days

---

## Phase 4: Monetization & Enterprise (Weeks 29-52)

**Goal:** Reach 50K+ signups, launch paid tiers, enterprise features

**Timeline:** September 1 - December 31, 2025  
**Team:** 4-5 engineers + 2-3 growth/marketing

### Premium Tier Launch (Weeks 29-32)

```typescript
// Freemium model
const tiers = {
  free: {
    price: 0,
    features: [
      'Create profile',
      'View 10 matches/month',
      'Send 5 messages/month',
      'Basic filters'
    ]
  },
  pro: {
    price: 99,  // $99/year
    features: [
      'All free features',
      'Unlimited matches',
      'Unlimited messaging',
      'Advanced filters',
      'See who viewed profile',
      'Match explanations',
      'Featured profile (30 days)'
    ]
  },
  team: {
    price: 499,  // $499/year
    features: [
      'All Pro features',
      '1:1 matchmaker call',
      'Custom team building',
      'Priority support',
      'Analytics dashboard',
      'Featured (90 days)',
      'Equity calculator'
    ]
  }
};
```

**Deliverables:**
- [ ] Stripe integration
- [ ] Subscription management
- [ ] Feature gating
- [ ] Billing dashboard
- [ ] Trial period logic

**Effort:** 5-6 days

### Enterprise Features (Weeks 33-40)

**For accelerators buying white-label:**

```typescript
// Admin dashboard
GET /api/admin/dashboard
// Returns:
// - Total users
// - Partnerships formed
// - Engagement metrics
// - Revenue

// API for third-party integrations
GET /api/v1/matches?user_id=xyz
GET /api/v1/users/:id/profile
POST /api/v1/messages
```

**Deliverables:**
- [ ] Admin dashboard
- [ ] Team analytics
- [ ] Custom branding
- [ ] API documentation
- [ ] API rate limiting
- [ ] Enterprise SLA

**Effort:** 8-10 days

### Performance & Scale (Weeks 41-48)

**Optimize for 50K+ concurrent users:**

```typescript
// Caching layer
const cache = new Redis();

// Cache matches for 1 hour
async function get_matches(user_id) {
  const cached = await cache.get(`matches:${user_id}`);
  if (cached) return cached;
  
  const matches = await compute_matches(user_id);
  await cache.setex(`matches:${user_id}`, 3600, matches);
  return matches;
}

// Database indexing
CREATE INDEX idx_user_skills ON user_skills(user_id, skill);
CREATE INDEX idx_messages ON messages(recipient_id, created_at DESC);
```

**Deliverables:**
- [ ] Caching strategy (Redis)
- [ ] Database query optimization
- [ ] API rate limiting
- [ ] Load balancing
- [ ] CDN for static assets
- [ ] Database replication

**Effort:** 10-12 days

### Expansion & Final Polish (Weeks 49-52)

**International launch, mobile app beta, etc.**

**Deliverables:**
- [ ] Multi-language support (i18n)
- [ ] International payment methods
- [ ] Mobile app (React Native or Flutter) beta
- [ ] SEO optimization
- [ ] Content marketing launch

**Effort:** Ongoing throughout phase

---

## Resource Requirements

### Team Composition

**Phase 1 (Weeks 1-8):**
- 2 Backend engineers (Node.js, database)
- 1 Frontend engineer (React)
- 1 ML engineer (optional, for tuning matching)
- 1 Product manager (part-time)
- **Total:** 3-4 FTE

**Phase 2-3 (Weeks 9-28):**
- 3 Backend engineers
- 2 Frontend engineers
- 1 DevOps engineer
- 1 QA engineer
- 1 Product manager
- 1 Growth/data analyst
- **Total:** 6-7 FTE

**Phase 4 (Weeks 29-52):**
- 4-5 engineers (distributed)
- 1 DevOps engineer
- 1-2 QA engineers
- 1 Product manager
- 2 Growth/marketing people
- **Total:** 8-10 FTE

### Infrastructure Costs

```
Phase 1: $500-1000/month
- Supabase: $100
- Vercel: $20
- SendGrid (email): $50
- Sentry (errors): $29
- Other: $300

Phase 2-3: $2000-5000/month
- Database scaling: $500
- API infrastructure: $1000
- ML services: $500
- Monitoring: $200
- Other: $500

Phase 4: $5000-10000/month (at 50K+ users)
- Database: $2000
- API/compute: $2000
- CDN: $500
- Services: $1000
- Other: $500
```

---

## Key Dependencies & Risks

### Technical Risks

1. **Matching algorithm accuracy**
   - Mitigation: Start simple, iterate based on user feedback
   - Target: 85%+ user satisfaction with matches

2. **Performance at scale**
   - Mitigation: Load test weekly
   - Target: <1s match loading at 50K users

3. **Data quality**
   - Mitigation: Validation on all inputs
   - Target: <0.1% data integrity issues

### Business Risks

1. **Low retention**
   - Mitigation: Gather user feedback weekly
   - Target: 40%+ day-7 retention

2. **Partnership delays**
   - Mitigation: Start outreach in week 1
   - Target: 3+ partnerships by month 3

3. **Scaling cost**
   - Mitigation: Efficient infrastructure choices
   - Target: <$0.50 CAC from organic

---

## Success Criteria by Phase

### Phase 1: MVP (Feb 28)
- [ ] Feature-complete MVP
- [ ] 500+ signups
- [ ] 40%+ day-7 retention
- [ ] 10+ successful matches
- [ ] <1% error rate

### Phase 2: Growth (Apr 30)
- [ ] 2000+ signups
- [ ] 35%+ day-7 retention
- [ ] 50+ successful connections
- [ ] 3+ accelerator partnerships announced
- [ ] Positive user feedback (NPS 30+)

### Phase 3: Scale (Aug 31)
- [ ] 10K+ signups
- [ ] 30%+ day-30 retention
- [ ] 500+ successful partnerships
- [ ] 10+ active partnerships
- [ ] <2s match loading

### Phase 4: Enterprise (Dec 31)
- [ ] 50K+ signups
- [ ] Profitable unit economics ($50K+ MRR)
- [ ] 10+ enterprise/accelerator customers
- [ ] 2000+ successful partnerships (cumulative)
- [ ] Ready for Series A or continued growth

---

## Conclusion

This roadmap is ambitious but achievable with a focused team. The key to success is:

1. **Ship fast** - MVP in 8 weeks, not 6 months
2. **Iterate quickly** - 2-week sprints, weekly feedback
3. **Focus on retention** - High-quality matches > user count
4. **Build partnerships early** - YC, Techstars = early scale
5. **Measure relentlessly** - Track metrics daily

**If we execute well, OpenHR will become the default co-founder matching platform by end of 2025.**

---

## Appendix: Development Environment Setup

**See:** `docs/developer-setup.md` for detailed setup instructions

**Quick start:**
```bash
git clone https://github.com/ArjunFrancis/openhr-platform.git
cd openhr-platform
npm install
npm run dev
# Frontend: localhost:3000
# Backend: localhost:4000
```
