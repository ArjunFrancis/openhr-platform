# Metrics, Success Criteria & Analytics Dashboard

**Date Created:** December 13, 2025  
**Status:** Final  
**Audience:** Product, Analytics, and Executive Teams

---

## Executive Summary

Success for OpenHR isn't just about user growth—it's about matching quality, platform engagement, and startup success. This document defines key performance indicators (KPIs) across three dimensions: **Adoption** (user growth), **Engagement** (platform activity), and **Outcomes** (real startup success). We establish baselines, targets, and alert thresholds that tell us whether the platform is healthy and achieving its mission. Real-time dashboards will surface these metrics to the team, enabling data-driven decisions on features, marketing, and community investment.

---

## 1. Adoption Metrics

### 1.1 User Growth

| Metric | Month 1 | Month 3 | Month 6 | Month 12 | Calculation |
|--------|---------|---------|---------|----------|-------------|
| **Total Signups** | 500 | 5,000 | 50,000 | 250,000 | Sum of all registered users |
| **DAU (Daily Active)** | 50 | 800 | 12,000 | 60,000 | Unique users with login in last 24h |
| **WAU (Weekly Active)** | 150 | 2,400 | 30,000 | 150,000 | Unique users with login in last 7d |
| **MAU (Monthly Active)** | 250 | 4,000 | 40,000 | 180,000 | Unique users with login in last 30d |
| **DAU/MAU Ratio** | 20% | 20% | 30% | 33% | Daily active / Monthly active |

**Target DAU/MAU = 30%+** (Healthy platforms like Facebook have 60%+; for niche platforms, 30% is excellent)

### 1.2 Cohort-Based Retention

**Track retention by signup cohort:**

| Cohort | Day-1 | Day-7 | Day-30 | Day-90 | Day-180 | Target |
|--------|-------|-------|--------|--------|---------|--------|
| Month 1 | 60% | 40% | 30% | 20% | 15% | 60% → 40% → 30% |
| Month 3 | 65% | 45% | 35% | 25% | 18% | Target: 35% day-30 |
| Month 6 | 70% | 50% | 40% | 30% | 20% | Improving over time |

**Alert:** If day-7 retention drops below 35%, pause all marketing and investigate user experience issues

### 1.3 Source of Signups

**Track where users come from:**

| Source | Target % | Month 1 | Month 3 | Month 6 | Why It Matters |
|--------|----------|---------|---------|---------|----------------|
| Organic (search, word-of-mouth) | 30% | 10% | 25% | 40% | Most sustainable |
| Paid ads (Google, Facebook) | 20% | 50% | 35% | 25% | Check ROI carefully |
| Partnerships (Y Combinator, accelerators) | 20% | 20% | 20% | 20% | High quality, verified users |
| Content/SEO | 15% | 5% | 15% | 10% | Growing long-term |
| Community events | 15% | 15% | 5% | 5% | Early phase |

**Goal:** Shift from paid ads to organic over time (sustainable growth)

---

## 2. Engagement Metrics

### 2.1 Profile Completion

| Metric | Target | Calculation | Why |
|--------|--------|-------------|-----|
| **Profiles 50%+ Complete** | 70% | Fields filled / total required fields | Users invested in profile |
| **Profiles 100% Complete** | 40% | All required + optional fields | High-quality matches |
| **GitHub Connected** | 60% | Users with linked GitHub | Data for enrichment |
| **LinkedIn Verified** | 50% | Users with verified LinkedIn | Trust signal |

**Action:** If profile completion <50%, create onboarding campaign

### 2.2 Search & Discovery Activity

| Metric | Target | Calculation | Alert Threshold |
|--------|--------|-------------|------------------|
| **Daily Searches** | 1000+ | Search queries submitted per day | <500: Investigate |
| **Profiles Viewed** | 5000+ | Profile page views per day | <2000: Low interest |
| **Avg Views/User/Day** | 3+ | Total views / DAU | <2: Users not exploring |
| **Avg Search Depth** | 5+ profiles | Profiles viewed per search | <3: Search not relevant |

### 2.3 Match Engagement

| Metric | Target | Calculation | Interpretation |
|--------|--------|-------------|----------------|
| **Matches Generated/User/Month** | 5+ | Total matches / users / 30 days | Algorithm producing options |
| **Matches Viewed (click-through)** | 30% | Matches clicked / matches shown | Users finding matches relevant |
| **Matches Saved** | 15% | Profiles bookmarked / viewed | Users interested in follow-up |
| **Profile Actions** | 20% | (Connect + Message + Save) / viewed | Engagement with matches |

**Alert:** If match CTR <20%, adjust matching algorithm

### 2.4 Messaging Activity

| Metric | Target | Calculation | Benchmark |
|--------|--------|-------------|----------|
| **Users with 1+ Message** | 40% | Users who sent any message | 40% = healthy engagement |
| **Avg Messages/User/Month** | 3+ | Total messages / active users | 3 = 1 conversation ongoing |
| **Message Response Rate** | 70% | Responses / sent messages | 70% = good community |
| **Avg Response Time** | <24 hours | Time to first response | Shows active user base |
| **Conversation Threads (2+ msgs)** | 25% of sent msgs | Back-and-forth conversations | Indicates real connections |

**Success indicator:** 40% message engagement = active matching community

---

## 3. Outcome Metrics

### 3.1 Match Success

**Most important: Did the platform actually help people form teams?**

| Metric | Target | Calculation | Timeline |
|--------|--------|-------------|----------|
| **Successful Matches/Month** | 50+ | Partnerships formed (self-reported) | By month 3 |
| **Equity Split Agreements** | 30+ | Co-founder agreements reached | By month 6 |
| **Technical Roles Filled** | 20+ | Developers hired by startups | By month 6 |
| **Co-Founder Pairs Formed** | 10+ | Two co-founders matched | By month 6 |

**Survey after 30 days:** "Is your match still active? Would you recommend?"

### 3.2 Match Quality (NPS-based)

**Net Promoter Score (NPS):**

```
NPS = (Promoters - Detractors) / Total Respondents × 100

Promoters (score 9-10): Users very satisfied; will recommend
Passives (score 7-8): Satisfied but not enthusiastic
Detractors (score 0-6): Unhappy; may damage reputation

Target NPS by month 6: 40+
(20+ is good; 40+ is excellent for B2C platforms)
```

### 3.3 Startup Success Tracking

**6-month follow-up with matched teams:**

| Metric | Target | How Measured | Importance |
|--------|--------|--------------|------------|
| **Partnership Still Active** | 80% | Still working together after 6mo | Core mission |
| **Raised Funding** | 30% | Series A/Seed funding raised | Validates team |
| **Shipped Product/MVP** | 50% | Live product or user-tested MVP | Shows execution |
| **Founders Still Together** | 90% | No co-founder breakups | Healthy teams |
| **Would Match Again** | 85% | "Would you find co-founder via OpenHR again" | Platform satisfaction |

**Qualitative feedback:** "How did OpenHR help your co-founder search?" (open-ended)

---

## 4. Financial Metrics (Future)

### 4.1 Unit Economics

**When monetization begins:**

| Metric | Target | Calculation |
|--------|--------|-------------|
| **CAC (Customer Acquisition Cost)** | <$20 | Marketing spend / new paying users |
| **LTV (Lifetime Value)** | >$200 | Revenue per user over lifetime |
| **LTV:CAC Ratio** | >10:1 | LTV / CAC (healthy = 3:1+) |
| **Payback Period** | <3 months | Months to recoup acquisition cost |

---

## 5. Fairness & Inclusion Metrics

**Ensure we're serving all user groups equitably:**

| Metric | Target | Calculation | Frequency |
|--------|--------|-------------|----------|
| **Gender Match Parity** | 0.95-1.05 | Female match rate / Male match rate | Daily |
| **Geographic Representation** | Proportional | % matches by region / % signups by region | Weekly |
| **Non-Traditional Background %** | 30%+ | Bootcamp/self-taught in matches | Monthly |
| **Age Diversity** | 15%+ | 45+ year olds in matches | Weekly |
| **User Satisfaction by Demographic** | No >10% gap | NPS by gender, geography, age | Monthly |

**Alert:** If any demographic drops >20% below parity, trigger fairness review

---

## 6. Technical Health Metrics

### 6.1 Platform Reliability

| Metric | Target | Calculation | SLA |
|--------|--------|-------------|-----|
| **Uptime** | 99.9% | Uptime minutes / total minutes | 4.5 hrs downtime/month max |
| **API Error Rate** | <0.1% | 5xx errors / total requests | Alert at 0.5% |
| **p95 Latency** | <500ms | 95th percentile response time | Alert at >1s |
| **p95 Search Time** | <1s | Search query response time | Alert at >2s |
| **Database Connections** | <80% | Used connections / pool size | Alert at >85% |

### 6.2 Code Quality

| Metric | Target | Frequency |
|--------|--------|----------|
| **Test Coverage** | >80% | Weekly |
| **Critical Bugs** | 0 open | Daily |
| **Security Vulnerabilities** | 0 | Upon each deploy |
| **Debt Ratio** | <5% | Monthly |

---

## 7. Analytics Dashboard

### 7.1 Real-Time Dashboard (for team)

```
┌─────────────────────────────────────────────────────┐
│  OPENHR COMMAND CENTER - LIVE METRICS              │
├─────────────────────────────────────────────────────┤
│
│  [TODAY]                [THIS WEEK]           [THIS MONTH]
│  DAU: 245 ⬆️ 12%        WAU: 1,832 ⬆️ 8%    MAU: 5,243 ⬆️ 15%
│  
│  ┌──────────────────┐  ┌──────────────────┐
│  │ ENGAGEMENT       │  │ PROFILE STATUS   │
│  ├──────────────────┤  ├──────────────────┤
│  │ Searches: 542    │  │ 50%+ Complete: 68% ✅ │
│  │ Matches: 128     │  │ 100% Complete: 38%   │
│  │ Messages: 387    │  │ GitHub Link: 58% ✅  │
│  │ Resp Rate: 71% ✅│  │ LinkedIn: 52% ✅     │
│  └──────────────────┘  └──────────────────┘
│
│  ┌──────────────────┐  ┌──────────────────┐
│  │ MATCH QUALITY    │  │ ALERTS           │
│  ├──────────────────┤  ├──────────────────┤
│  │ Success: 8/mo ✅ │  │ ⚠️  DAU down 5%   │
│  │ NPS: 38 ⬆️       │  │ ⚠️  Response<65%  │
│  │ Retention D30: 32%│  │ ✅ All systems go │
│  └──────────────────┘  └──────────────────┘
│
│  SOURCE BREAKDOWN:  Organic 28% | Paid 35% | Partners 20% | Other 17%
│
└─────────────────────────────────────────────────────┘
```

### 7.2 Weekly Metrics Report

**Auto-generated every Monday 9am UTC:**

```
📊 OPENHR WEEKLY METRICS (Dec 9-15, 2025)

✅ Growth
  • New signups: 847 (+12% vs last week)
  • DAU: 234 avg (+8% trend)
  • Retention D7: 41% (target: 40%+)

✅ Engagement  
  • Avg profile completion: 62% (+3 points)
  • Searches/user/day: 2.3 (baseline: 2.1)
  • Messages sent: 1,247 (+15%)

⚠️  Concerns
  • Match quality score: 3.8/5 (down from 4.1)
  • Response rate: 65% (target: 70%)
  
💡 Recommendations
  1. Review match algorithm for relevance issues
  2. Increase messaging prompts to boost engagement
  3. Feature top matches to improve visibility

Next update: Monday Dec 16, 9am UTC
```

---

## 8. Experimentation Framework

### 8.1 A/B Testing Protocol

**All features changes tested before rollout:**

| Phase | Duration | Sample Size | Decision |
|-------|----------|-------------|----------|
| **Alpha** | 1 week | 100 users | Bugs, UX issues |
| **Beta** | 2 weeks | 5% of users | Statistical significance |
| **Rollout** | Ongoing | 100% | Monitor key metrics |

**Key metric to optimize per feature:**

- Match discovery UI: Match CTR (target: 30%+)
- Messaging prompts: Message send rate (target: 40%+)
- Profile enrichment: Profile completion (target: 70%)
- Onboarding: Day-7 retention (target: 40%+)

### 8.2 Hypothesis Testing

**Example A/B test:**

```
Hypothesis: "Match explanation cards increase CTR"

Control: Match card without explanation
Variant: Match card with "Why you matched: React expert + 5yo exp + TZ overlap"

Metric: Click-through rate (CTR)
Target: 30% vs 25% (20% improvement)
Duration: 2 weeks
Sample: 5% of users (1000+ users)

Result: 
  Control CTR: 24%
  Variant CTR: 31%
  Improvement: +29% ✅ STAT SIG
  Decision: ROLLOUT to 100%
```

---

## 9. Monitoring & Alerts

### 9.1 Alert Thresholds

| Metric | Green | Yellow | Red | Action |
|--------|-------|--------|-----|--------|
| **DAU Change** | >-5% | -5% to -15% | <-15% | Investigate UX |
| **Retention D7** | >40% | 35-40% | <35% | Pause growth; fix product |
| **Match CTR** | >25% | 20-25% | <20% | Retrain algorithm |
| **Response Rate** | >70% | 60-70% | <60% | Community outreach |
| **Message Response Time** | <24h | 24-48h | >48h | User engagement issue |
| **Uptime** | >99.9% | 99.5-99.9% | <99.5% | Critical incident |

### 9.2 Alert Notification

**Real-time alerts via:**
- Slack #metrics channel (automated)
- Email to product + engineering leads
- SMS for critical incidents (uptime <99%)

---

## 10. Monthly Review Process

**Every first Monday of month, 10am UTC:**

### Attendees
- Product Lead
- Analytics Lead
- Engineering Lead
- Growth Lead
- CEO/Founder

### Agenda (1 hour)

1. **Metrics Review** (20 min)
   - Last month's performance vs targets
   - Highlight wins and misses
   - Retention cohorts

2. **Deep Dive** (20 min)
   - Focus on 1-2 key metrics needing improvement
   - Root cause analysis
   - Proposed actions

3. **Roadmap Adjustment** (15 min)
   - Reprioritize features based on metrics
   - Resource allocation
   - Next month targets

4. **Documentation** (5 min)
   - Document decisions
   - Update roadmap
   - Schedule next review

---

## 11. Success Criteria by Phase

### Phase 1: MVP Launch (Month 1-2)

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| Signups | 500 | - | ⏳ Launch pending |
| DAU | 100+ | - | ⏳ |
| Profile Complete | 60%+ | - | ⏳ |
| Retention D7 | 35%+ | - | ⏳ |

**MVP is successful if:** 500+ signups, 35%+ D7 retention

### Phase 2: Growth (Month 3-6)

| Metric | Target | Status |
|--------|--------|--------|
| Signups | 5,000 | ⏳ |
| DAU | 1,000+ | ⏳ |
| Matches Generated | 500+ | ⏳ |
| Successful Partnerships | 50+ | ⏳ |

### Phase 3: Scale (Month 6-12)

| Metric | Target |
|--------|--------|
| Signups | 50,000 |
| DAU | 10,000+ |
| Successful Partnerships | 500+ |
| NPS | 40+ |

---

## Conclusion

These metrics define what "success" looks like for OpenHR. By tracking them consistently, we can:

✅ Know if the product is resonating (engagement metrics)  
✅ Understand if we're reaching the right users (adoption metrics)  
✅ Validate that we're actually solving the problem (outcome metrics)  
✅ Ensure we're serving all user groups fairly (fairness metrics)  
✅ Maintain platform reliability (technical metrics)  

**Real-time dashboards, weekly reports, and monthly reviews** keep the team aligned and data-driven.

---

## References

- AARRR Metrics Framework (Pirate Metrics) by Dave McClure
- SiriusDecisions benchmark data for B2B SaaS
- Greylock Analytics Framework for network effects
- Y Combinator Startup Success Metrics
