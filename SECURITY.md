# Security Policy

## Supported Versions

OpenHR is currently in the **Research & Planning phase**. Once the MVP is released, we will provide security updates for the following versions:

| Version | Supported          |
| ------- | ------------------ |
| 1.x.x   | :white_check_mark: |
| < 1.0   | :x:                |

**Current Status**: Pre-release (no production deployments yet)

---

## Reporting a Vulnerability

We take security seriously. If you discover a security vulnerability in OpenHR, please follow responsible disclosure practices.

### 🚨 DO NOT create a public GitHub issue for security vulnerabilities

### How to Report

**Email**: [arjunfrancis21@gmail.com](mailto:arjunfrancis21@gmail.com)

**Subject Line**: `[SECURITY] Brief description of vulnerability`

**Include in your report**:
1. **Description**: What is the vulnerability?
2. **Impact**: What could an attacker do with this vulnerability?
3. **Steps to Reproduce**: Detailed steps to reproduce the issue
4. **Proof of Concept**: Code, screenshots, or video (if applicable)
5. **Suggested Fix**: Optional, but appreciated
6. **Your Contact Info**: For follow-up questions

### What to Expect

1. **Acknowledgment**: Within 48 hours of your report
2. **Initial Assessment**: Within 5 business days
3. **Updates**: Regular updates on progress (at least every 7 days)
4. **Resolution**: We aim to fix critical issues within 30 days
5. **Disclosure**: Coordinated disclosure after a fix is released

### Scope

**In Scope**:
- Authentication and authorization bypasses
- SQL injection, XSS, CSRF, and other injection attacks
- Remote code execution (RCE)
- Server-side request forgery (SSRF)
- Sensitive data exposure (PII, credentials, etc.)
- Privilege escalation
- Security misconfigurations

**Out of Scope**:
- Vulnerabilities in third-party dependencies (report to them directly, then notify us)
- Social engineering attacks
- Denial of Service (DoS) attacks
- Issues in development/staging environments (report them, but they're lower priority)
- Theoretical vulnerabilities without proof of concept

---

## Security Best Practices for Contributors

### Code Security

1. **Never commit secrets**:
   - API keys, passwords, tokens, etc. should be in `.env` files (not tracked by Git)
   - Use `.gitignore` to exclude `.env` files
   - Use environment variables for all sensitive configuration

2. **Validate all inputs**:
   - Sanitize user inputs to prevent SQL injection and XSS
   - Use parameterized queries (never string concatenation for SQL)
   - Use libraries like `validator.js` or `Joi` for input validation

3. **Use secure dependencies**:
   - Run `npm audit` (Node.js) and `pip-audit` (Python) regularly
   - Update dependencies to patch known vulnerabilities
   - Review dependency security advisories

4. **Implement proper authentication**:
   - Use Supabase Auth for user authentication (OAuth2, JWT)
   - Never store passwords in plain text
   - Implement row-level security (RLS) in Postgres

5. **Follow least privilege principle**:
   - Database users should have minimal required permissions
   - API endpoints should check authorization for every request
   - Use role-based access control (RBAC) where appropriate

### Deployment Security

1. **Use HTTPS everywhere**: All production traffic must use TLS/SSL
2. **Set security headers**:
   - `Content-Security-Policy`
   - `X-Frame-Options`
   - `X-Content-Type-Options`
   - `Strict-Transport-Security` (HSTS)
3. **Enable rate limiting**: Prevent brute-force and DoS attacks
4. **Keep secrets out of version control**: Use secret management tools
5. **Regular backups**: Automated daily backups with encryption

### Data Privacy

1. **Minimize PII collection**: Only collect data you need
2. **Encrypt sensitive data**: At rest and in transit
3. **Implement GDPR compliance**:
   - Right to access (data export)
   - Right to deletion (account deletion)
   - Right to rectification (data correction)
4. **Log security events**: Authentication attempts, permission changes, data access
5. **Anonymize logs**: Never log PII (emails, names, messages)

---

## Known Security Considerations

### Current Architecture

1. **Row-Level Security (RLS)**: We rely on Supabase's RLS policies to enforce data access control. Contributors must ensure all tables have proper RLS policies.

2. **API Authentication**: All API endpoints (except `/auth/*`) require JWT tokens. Contributors must validate tokens on every request.

3. **Third-Party Integrations**:
   - **GitHub API**: OAuth2 tokens are stored securely; never log tokens
   - **OpenAI API**: API keys are environment variables; track token usage

4. **ML Service**: The Python ML service has access to the database and vector store. Ensure input validation to prevent injection attacks.

### Future Security Roadmap

- [ ] Implement Content Security Policy (CSP) for frontend
- [ ] Add API rate limiting (per user and per IP)
- [ ] Implement automated security scanning (Snyk, Dependabot)
- [ ] Add penetration testing for MVP
- [ ] Implement security audit logging
- [ ] Add two-factor authentication (2FA) for users
- [ ] Implement anomaly detection for suspicious activity

---

## Security Contacts

**Primary Contact**: [arjunfrancis21@gmail.com](mailto:arjunfrancis21@gmail.com)  
**Project Maintainer**: [@ArjunFrancis](https://github.com/ArjunFrancis)

---

## Acknowledgments

We appreciate responsible disclosure. Security researchers who report valid vulnerabilities will be:

1. **Acknowledged** in our security advisories (unless you prefer to remain anonymous)
2. **Credited** in release notes
3. **Thanked** publicly on our GitHub and website

For significant vulnerabilities, we may offer:
- A mention in our Hall of Fame (coming soon)
- OpenHR swag (t-shirts, stickers)
- A reference/recommendation letter (upon request)

**Note**: OpenHR is an open-source project. We cannot offer monetary bug bounties at this time.

---

## Additional Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CWE Top 25 Most Dangerous Software Weaknesses](https://cwe.mitre.org/top25/)
- [Supabase Security Best Practices](https://supabase.com/docs/guides/auth/managing-user-data)
- [Node.js Security Best Practices](https://nodejs.org/en/docs/guides/security/)

---

**Thank you for helping keep OpenHR and our community safe!** 🔒