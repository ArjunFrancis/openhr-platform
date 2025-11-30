# Privacy & GDPR Compliance

**Task**: Day 6 - Privacy & GDPR Compliance  
**Created**: November 30, 2025  
**Status**: Final

---

## Executive Summary

This document defines OpenHR Platform's **privacy and GDPR compliance strategy**, ensuring user data is collected, stored, processed, and deleted in accordance with GDPR (General Data Protection Regulation), CCPA (California Consumer Privacy Act), and other global privacy laws.

As a platform handling sensitive career and personal data (skills, work history, compensation preferences), OpenHR must implement **privacy-by-design** principles and give users full control over their data.

---

## Legal Requirements

### GDPR (EU General Data Protection Regulation)

**Applies to**: EU citizens and residents, regardless of where platform is hosted

**Key Requirements**:
1. **Lawful Basis for Processing**: Consent, contract, legitimate interest
2. **Data Minimization**: Collect only necessary data
3. **Purpose Limitation**: Use data only for stated purposes
4. **Right to Access**: Users can download their data
5. **Right to Erasure**: Users can request account deletion
6. **Right to Portability**: Users can export data in machine-readable format
7. **Breach Notification**: Report breaches within 72 hours

### CCPA (California Consumer Privacy Act)

**Applies to**: California residents

**Key Requirements**:
1. **Right to Know**: Disclose what data is collected
2. **Right to Delete**: Users can request data deletion
3. **Right to Opt-Out**: Users can opt-out of data selling (N/A for OpenHR)
4. **Non-Discrimination**: Cannot deny service for exercising rights

### Other Privacy Laws

- **UK GDPR**: Similar to EU GDPR
- **PIPEDA (Canada)**: Personal Information Protection and Electronic Documents Act
- **LGPD (Brazil)**: Lei Geral de Proteção de Dados

**OpenHR Approach**: Build for GDPR compliance → meets most other privacy laws

---

## Privacy Principles

### 1. Privacy by Design

**Principle**: Privacy considerations integrated from the start

**Implementation**:
- Default to minimal data collection
- Encrypt data at rest and in transit
- Anonymize analytics data
- Implement Row-Level Security (RLS) in Supabase
- Regular privacy audits

### 2. Transparency

**Principle**: Users understand what data is collected and why

**Implementation**:
- Clear, readable Privacy Policy (not legalese)
- Data collection notices at point of collection
- "Why do we need this?" tooltips on forms
- Annual transparency reports

### 3. User Control

**Principle**: Users have full control over their data

**Implementation**:
- Easy data export (JSON, CSV formats)
- One-click account deletion
- Granular privacy settings (profile visibility, data sharing)
- Opt-out of non-essential data collection

---

## Data Inventory

### Personal Data Collected

| Data Type | Purpose | Legal Basis | Retention |
|-----------|---------|-------------|----------|
| **Identity Data** | | | |
| Email address | Authentication, communication | Contract | Until account deletion |
| Name | Profile display | Contract | Until account deletion |
| Avatar/Photo | Profile display | Consent | Until account deletion |
| **Profile Data** | | | |
| Bio, vision statement | Match discovery | Contract | Until account deletion |
| Location | Match filtering | Contract | Until account deletion |
| Timezone | Match filtering | Contract | Until account deletion |
| Skills | Skill matching | Contract | Until account deletion |
| Work experience | Profile enrichment | Consent | Until account deletion |
| **Preferences** | | | |
| Startup stage pref | Match filtering | Contract | Until account deletion |
| Equity vs salary pref | Match filtering | Contract | Until account deletion |
| Remote vs in-person pref | Match filtering | Contract | Until account deletion |
| **Engagement Data** | | | |
| Messages sent/received | Messaging feature | Contract | Until account deletion |
| Matches viewed | Match recommendations | Legitimate interest | 90 days (aggregated) |
| Login activity | Security, analytics | Legitimate interest | 90 days (aggregated) |
| **Third-Party Data** | | | |
| GitHub profile, repos | Profile enrichment | Consent | Until revoked |
| LinkedIn profile | Profile enrichment | Consent | Until revoked |
| Resume content | Profile enrichment | Consent | Until deleted |

### Sensitive Data (Extra Protection)

**Not Collected by OpenHR**:
- Race, ethnicity, religion
- Political opinions
- Health data
- Biometric data (beyond avatar photo)
- Criminal records

**If Users Voluntarily Provide**: Flagged, extra consent required, encrypted

---

## Consent Management

### Consent Collection

**On Signup**:
```
☐ I agree to the Terms of Service and Privacy Policy (required)
☐ I consent to profile enrichment from GitHub and LinkedIn (optional)
☐ I consent to receive product updates and match notifications via email (optional)
```

**Granular Consent** (Settings Page):
- ☐ Allow OpenHR to analyze my GitHub repos for skills
- ☐ Allow OpenHR to access my LinkedIn profile data
- ☐ Allow OpenHR to analyze my resume for skills
- ☐ Show my profile in search results
- ☐ Allow matches to see my full profile
- ☐ Send me weekly match digests via email

### Consent Tracking

**Database Schema**:
```sql
CREATE TABLE user_consents (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  consent_type VARCHAR(100), -- 'terms', 'privacy', 'github_enrichment', 'email_notifications'
  granted BOOLEAN,
  granted_at TIMESTAMP,
  revoked_at TIMESTAMP,
  ip_address INET,
  user_agent TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);
```

**Audit Trail**: Record when and how consent was given/revoked

---

## User Rights Implementation

### 1. Right to Access (Data Export)

**Goal**: User can download all their data in machine-readable format

#### Implementation

**Data Export Feature**:
1. User clicks "Export My Data" in settings
2. System generates ZIP file containing:
   - `profile.json` (name, bio, location, skills)
   - `preferences.json` (startup stage, equity preference)
   - `matches.json` (list of matches with scores)
   - `messages.json` (all messages sent/received)
   - `activity.json` (login history, profile views)
   - `consents.json` (consent history)
3. Download link emailed within 24 hours
4. Link expires after 7 days

**API Endpoint**:
- `POST /privacy/export-data` - Request data export
- `GET /privacy/export-data/:job_id` - Check export status

**Example JSON Output**:
```json
{
  "user_id": "uuid-123",
  "email": "john@example.com",
  "profile": {
    "name": "John Doe",
    "bio": "Full-stack developer...",
    "location": "San Francisco, CA",
    "skills": ["React", "Node.js", "Python"]
  },
  "matches": [
    {
      "matched_user_id": "uuid-456",
      "match_score": 85,
      "matched_at": "2025-11-20T10:00:00Z"
    }
  ],
  "messages": [
    {
      "conversation_id": "uuid-789",
      "content": "Hi, I saw we're both...",
      "sent_at": "2025-11-21T15:30:00Z"
    }
  ]
}
```

---

### 2. Right to Erasure (Account Deletion)

**Goal**: User can permanently delete their account and all associated data

#### Implementation

**Account Deletion Flow**:
1. User clicks "Delete Account" in settings
2. Confirmation modal with warning:
   - "This action is permanent and cannot be undone"
   - "All your data will be deleted within 30 days"
   - Checkbox: "I understand and want to delete my account"
3. User re-authenticates (enter password)
4. Account marked as "pending_deletion"
5. Grace period: 30 days to change mind
6. After 30 days, permanent deletion job runs

**Data Deletion** (cascading):
```sql
-- Soft delete for grace period
UPDATE users SET status = 'pending_deletion', deletion_scheduled_at = NOW() + INTERVAL '30 days'
WHERE id = <user_id>;

-- Hard delete after grace period
DELETE FROM user_skills WHERE user_id = <user_id>;
DELETE FROM user_preferences WHERE user_id = <user_id>;
DELETE FROM matches WHERE user_id = <user_id> OR matched_user_id = <user_id>;
DELETE FROM messages WHERE sender_id = <user_id>; -- Anonymize instead?
DELETE FROM user_consents WHERE user_id = <user_id>;
DELETE FROM profiles WHERE user_id = <user_id>;
DELETE FROM users WHERE id = <user_id>;
```

**Exception**: Messages sent to other users are **anonymized** (not deleted) to preserve conversation context:
```sql
UPDATE messages 
SET sender_id = NULL, 
    sender_name = '[Deleted User]', 
    anonymized = TRUE
WHERE sender_id = <user_id>;
```

**API Endpoint**:
- `POST /privacy/delete-account` - Request account deletion
- `POST /privacy/cancel-deletion` - Cancel deletion (within grace period)

---

### 3. Right to Portability (Data Transfer)

**Goal**: User can export data in a format usable by other services

#### Implementation

**Export Formats**:
- **JSON**: Machine-readable, full data
- **CSV**: Spreadsheet-compatible (profile, skills, matches)
- **vCard**: Contact information (name, email, LinkedIn)

**Example CSV Export** (skills.csv):
```csv
skill_name,proficiency_level,years_experience,source
React,Expert,5,github
Node.js,Advanced,4,manual
Python,Intermediate,2,resume
```

---

### 4. Right to Rectification (Data Correction)

**Goal**: User can correct inaccurate or incomplete data

#### Implementation

**Self-Service Editing**:
- User can edit all profile fields at any time
- "Edit Profile" button on profile page
- Changes saved instantly (no approval needed)

**Request Manual Correction**:
- If data is system-generated (e.g., GitHub skill extraction), user can flag as incorrect
- Support ticket created for manual review

---

### 5. Right to Object (Opt-Out)

**Goal**: User can object to specific data processing activities

#### Implementation

**Opt-Out Options** (Settings Page):
- ☐ Stop using my data for match recommendations (hides profile from search)
- ☐ Stop sending me email notifications (opt-out of all emails except account security)
- ☐ Stop analyzing my GitHub repos (revoke GitHub access)
- ☐ Stop analyzing my LinkedIn profile (revoke LinkedIn access)
- ☐ Stop tracking my activity for analytics (reduces trust score, but respects privacy)

**Impact Warnings**: "Opting out of match recommendations will hide your profile from search. You can still browse matches manually."

---

## Data Security

### Encryption

**Data at Rest**:
- PostgreSQL database encrypted with AES-256 (Supabase default)
- File storage (avatars, resumes) encrypted with AES-256 (Supabase Storage)
- Backups encrypted

**Data in Transit**:
- TLS 1.3 for all API requests
- HTTPS enforced (no HTTP allowed)
- WebSocket connections over WSS (secure)

### Access Controls

**Supabase Row-Level Security (RLS)**:
```sql
-- Users can only read their own data
CREATE POLICY users_select_own ON users
  FOR SELECT USING (auth.uid() = id);

-- Users can only update their own profile
CREATE POLICY profiles_update_own ON profiles
  FOR UPDATE USING (auth.uid() = user_id);

-- Users can only see messages in their conversations
CREATE POLICY messages_select_own ON messages
  FOR SELECT USING (
    auth.uid() IN (
      SELECT user1_id FROM conversations WHERE id = conversation_id
      UNION
      SELECT user2_id FROM conversations WHERE id = conversation_id
    )
  );
```

**API Authentication**:
- JWT tokens with 1-hour expiration
- Refresh tokens with 30-day expiration
- API keys for ML services (server-to-server only)

### Monitoring & Auditing

**Security Logs**:
- All data access logged (who, what, when)
- Failed login attempts tracked
- Suspicious activity alerts (e.g., 100 profile views in 1 hour)
- Quarterly security audits

---

## Third-Party Data Sharing

### When OpenHR Shares Data

**Never Shared**:
- User data is **never sold** to third parties
- User data is **never shared** with advertisers

**Limited Sharing** (with user consent):

| Third Party | Data Shared | Purpose | Legal Basis |
|-------------|-------------|---------|-------------|
| GitHub | User ID | OAuth authentication | Consent |
| LinkedIn | User ID | OAuth authentication | Consent |
| Supabase | All data | Database hosting | Contract (DPA) |
| Vercel | None (except logs) | Frontend hosting | Contract |
| Sentry | Error logs (anonymized) | Error tracking | Legitimate interest |
| PostHog/Mixpanel | Usage analytics (anonymized) | Product analytics | Legitimate interest |

**Data Processing Agreements (DPAs)**:
- Signed with all third-party processors
- Ensures GDPR compliance for data processing
- Specifies data handling, security, and deletion procedures

---

## Privacy Policy

### Required Sections

**1. Introduction**
- Who we are (OpenHR, open-source project)
- What this policy covers

**2. Data We Collect**
- Personal data (name, email, location)
- Usage data (login activity, messages)
- Third-party data (GitHub, LinkedIn)

**3. How We Use Data**
- Match recommendations
- Profile enrichment
- Communication
- Analytics

**4. Legal Basis**
- Consent
- Contract
- Legitimate interest

**5. Data Sharing**
- Third-party services (Supabase, Vercel)
- No selling, no advertising

**6. Your Rights**
- Access, portability, erasure
- How to exercise rights

**7. Data Security**
- Encryption, access controls
- Breach notification

**8. Cookies**
- Essential cookies only (authentication)
- No tracking cookies

**9. Contact**
- privacy@openhr.com
- Data Protection Officer (if applicable)

**10. Updates**
- Last updated date
- Notification of changes

### Privacy Policy Template

See: [Example Privacy Policy](https://www.privacypolicies.com/blog/privacy-policy-template/)

**Key Principle**: Write in plain language, not legalese

---

## Cookie & Tracking Policy

### Cookies Used

**Essential Cookies** (no consent required):
| Cookie | Purpose | Duration |
|--------|---------|----------|
| `auth-token` | Authentication | 1 hour |
| `refresh-token` | Session refresh | 30 days |
| `csrf-token` | CSRF protection | Session |

**Analytics Cookies** (consent required):
| Cookie | Purpose | Duration |
|--------|---------|----------|
| `_analytics` | Anonymous usage tracking | 1 year |

**No Third-Party Tracking**:
- No Google Analytics (use privacy-friendly PostHog or Plausible)
- No Facebook Pixel
- No ad retargeting cookies

### Cookie Consent Banner

**On First Visit**:
```
+---------------------------------------------------------------+
| This website uses cookies to improve your experience.        |
|                                                               |
| Essential cookies are required for the site to function.     |
| Analytics cookies help us improve OpenHR (optional).         |
|                                                               |
| [Accept All] [Essential Only] [Cookie Settings]              |
+---------------------------------------------------------------+
```

**Compliance**: GDPR requires explicit consent for non-essential cookies

---

## Data Breach Response Plan

### Breach Definition

A **data breach** occurs when:
- Unauthorized access to user data
- Data is stolen, leaked, or lost
- Security vulnerability is exploited

### Response Procedure

**Within 1 Hour**:
1. Identify scope of breach (what data, how many users)
2. Contain breach (disable compromised systems, revoke API keys)
3. Assemble incident response team

**Within 24 Hours**:
4. Investigate root cause
5. Assess risk to users (low, medium, high)
6. Prepare user notification (if required)

**Within 72 Hours** (GDPR requirement):
7. Notify supervisory authority (e.g., ICO in UK, CNIL in France)
8. Notify affected users (if high risk to rights and freedoms)
9. Document breach in breach register

**Within 1 Week**:
10. Implement fixes and security improvements
11. Publish transparency report (optional, but recommended)

### User Notification Template

**Email Subject**: Important Security Notice - OpenHR Data Breach

**Email Body**:
```
Dear [Name],

We are writing to inform you of a security incident that may have affected your OpenHR account.

What happened:
[Brief description of breach]

What data was affected:
[List of data types: email, profile, messages, etc.]

What we're doing:
- [Action 1: e.g., reset all passwords]
- [Action 2: e.g., enhanced security monitoring]
- [Action 3: e.g., security audit]

What you should do:
- Change your password immediately
- Enable two-factor authentication
- Monitor your account for suspicious activity

We sincerely apologize for this incident. Your privacy and security are our top priorities.

Contact us:
privacy@openhr.com

Sincerely,
OpenHR Team
```

---

## GDPR Compliance Checklist

### Pre-Launch

- [ ] Privacy Policy published and accessible
- [ ] Terms of Service published
- [ ] Cookie consent banner implemented
- [ ] Data export feature built
- [ ] Account deletion feature built
- [ ] Consent tracking system implemented
- [ ] Row-Level Security (RLS) policies set up
- [ ] Data encryption enabled (at rest and in transit)
- [ ] DPAs signed with third-party processors
- [ ] Data breach response plan documented

### Ongoing

- [ ] Quarterly privacy audits
- [ ] Annual privacy policy review
- [ ] User consent re-confirmation (if policy changes)
- [ ] Security vulnerability scanning (monthly)
- [ ] Penetration testing (annually)
- [ ] Data retention policy enforcement (delete old logs)
- [ ] Breach register maintained

---

## Implementation Roadmap

### Phase 1: Core Compliance (Weeks 1-4)

- [ ] Write Privacy Policy and Terms of Service
- [ ] Implement consent tracking (signup, settings)
- [ ] Set up RLS policies in Supabase
- [ ] Enable TLS/HTTPS for all endpoints
- [ ] Cookie consent banner

### Phase 2: User Rights (Weeks 5-8)

- [ ] Data export feature (JSON, CSV)
- [ ] Account deletion with grace period
- [ ] Privacy settings page (opt-outs, consents)
- [ ] Data rectification (edit profile)

### Phase 3: Security & Monitoring (Weeks 9-12)

- [ ] Security logging and monitoring
- [ ] Breach detection alerts
- [ ] Quarterly security audits
- [ ] Penetration testing

---

## Resources & Tools

### GDPR Resources

- **ICO (UK)**: https://ico.org.uk/for-organisations/gdpr-guidance/
- **CNIL (France)**: https://www.cnil.fr/en/home
- **GDPR Checklist**: https://gdpr.eu/checklist/

### Privacy Tools

- **Supabase RLS**: https://supabase.com/docs/guides/auth/row-level-security
- **Sentry (Error Tracking)**: https://sentry.io/
- **PostHog (Privacy-Friendly Analytics)**: https://posthog.com/
- **Cookie Consent**: https://www.osano.com/cookieconsent

### Legal Templates

- **Privacy Policy Generator**: https://www.freeprivacypolicy.com/
- **Terms of Service Template**: https://www.termsfeed.com/

---

## References

1. **GDPR Official Text**: https://gdpr-info.eu/
2. **CCPA Official Text**: https://oag.ca.gov/privacy/ccpa
3. **Supabase Security**: https://supabase.com/security
4. **OWASP Privacy Guidelines**: https://owasp.org/www-project-top-ten/

---

**Document Owner**: OpenHR Research & Planning Agent  
**Last Updated**: November 30, 2025  
**Version**: 1.0  
**Status**: Final
