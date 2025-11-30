# Community Guidelines & Contribution Workflows

**Task**: Day 6 - Open-Source Community Guidelines  
**Created**: November 30, 2025  
**Status**: Final

---

## Executive Summary

OpenHR is an **open-source, community-driven project** welcoming contributions from developers, designers, researchers, and product managers worldwide. This document defines contribution workflows, code of conduct, issue/PR templates, and community norms to ensure a healthy, inclusive, and productive open-source ecosystem.

**Mission**: Build the world's best co-founder and developer matching platform through collaborative open-source development.

---

## Code of Conduct

### Our Pledge

In the interest of fostering an open and welcoming environment, we as contributors and maintainers pledge to make participation in OpenHR a **harassment-free experience** for everyone, regardless of:

- Age, body size, disability, ethnicity, gender identity and expression
- Level of experience, education, socio-economic status
- Nationality, personal appearance, race, religion
- Sexual identity and orientation

### Our Standards

**Positive Behavior (Encouraged)**:
- Using welcoming and inclusive language
- Respecting differing viewpoints and experiences
- Gracefully accepting constructive criticism
- Focusing on what is best for the community
- Showing empathy towards other community members

**Unacceptable Behavior (Prohibited)**:
- Trolling, insulting/derogatory comments, and personal or political attacks
- Public or private harassment
- Publishing others' private information (doxing)
- Other conduct which could reasonably be considered inappropriate in a professional setting

### Enforcement

**Reporting**: Email conduct@openhr.com or DM maintainers on Discord

**Response Process**:
1. **Warning**: First offense, private warning issued
2. **Temporary Ban**: Repeated offenses, 7-day ban from community spaces
3. **Permanent Ban**: Severe or repeated violations, permanent ban

**Appeals**: Email appeals@openhr.com within 30 days

**Based on**: [Contributor Covenant](https://www.contributor-covenant.org/) v2.1

---

## Getting Started

### Prerequisites

**For Code Contributions**:
- Node.js 18+ (for frontend)
- Python 3.10+ (for ML services)
- Git & GitHub account
- Supabase account (for local development)

**For Documentation Contributions**:
- Markdown editor
- Basic understanding of OpenHR architecture

**For Design Contributions**:
- Figma account
- Understanding of React/TailwindCSS components

### Setup Development Environment

**1. Fork and Clone**:
```bash
git clone https://github.com/YOUR_USERNAME/openhr-platform.git
cd openhr-platform
```

**2. Install Dependencies**:
```bash
# Frontend
cd frontend
npm install

# Backend
cd ../backend
npm install

# ML Service
cd ../ml-service
pip install -r requirements.txt
```

**3. Set Up Environment Variables**:
```bash
cp .env.example .env
# Edit .env with your Supabase, OpenAI, GitHub API keys
```

**4. Run Development Servers**:
```bash
# Frontend (port 3000)
npm run dev

# Backend (port 4000)
npm run server

# ML Service (port 8000)
python main.py
```

**5. Run Tests**:
```bash
npm test # Frontend + backend
pytest # ML service
```

---

## Contribution Types

### 1. Code Contributions

**Areas to Contribute**:
- **Frontend** (React, Next.js, TailwindCSS): UI components, pages, forms
- **Backend** (Node.js, Express, Supabase): API endpoints, database queries
- **ML Services** (Python, FastAPI): Skill matching, profile enrichment, LLM integrations
- **Infrastructure** (Vercel, Supabase, Railway): CI/CD, deployment, monitoring

**Good First Issues**:
- Look for issues labeled `good first issue` or `help wanted`
- Examples: "Add dark mode toggle", "Fix typo in error message", "Improve skill autocomplete"

**How to Contribute Code**:
1. Find an issue or create one
2. Comment "I'd like to work on this"
3. Maintainer assigns issue to you
4. Create a feature branch: `git checkout -b feature/your-feature-name`
5. Make changes, write tests, update docs
6. Commit with conventional commits: `feat: add dark mode toggle`
7. Push and create pull request
8. Address review feedback
9. Once approved, maintainer merges PR

---

### 2. Documentation Contributions

**Areas to Contribute**:
- **Research docs** (market analysis, user personas, algorithms)
- **Architecture docs** (system design, database schema, API specs)
- **Feature specs** (UI/UX, onboarding, messaging)
- **Developer docs** (setup guides, API reference, examples)

**How to Contribute Documentation**:
1. Identify gaps (missing sections, outdated info, unclear explanations)
2. Create issue: "Improve documentation for X"
3. Make changes in Markdown files
4. Submit PR with clear description of improvements

**Documentation Standards**:
- Use clear, concise language (avoid jargon)
- Include code examples and screenshots where helpful
- Follow existing structure (Executive Summary, sections, references)
- Proofread for grammar and spelling

---

### 3. Design Contributions

**Areas to Contribute**:
- **UI/UX Design**: Wireframes, mockups, prototypes
- **Visual Design**: Color schemes, typography, iconography
- **Design System**: Component library, style guide
- **User Research**: User interviews, surveys, usability testing

**How to Contribute Design**:
1. Check #design channel on Discord for current projects
2. Create Figma designs (link in GitHub issue)
3. Get feedback from community
4. Once approved, designs handed off to developers

**Design Files**:
- Store in `ui-design/` directory
- Link Figma files in issue descriptions
- Include design rationale and user flow diagrams

---

### 4. Bug Reports & Feature Requests

**How to Report Bugs**:
1. Search existing issues to avoid duplicates
2. Create new issue with template (see below)
3. Include: steps to reproduce, expected behavior, actual behavior, screenshots
4. Tag with `bug` label

**How to Request Features**:
1. Check roadmap to see if already planned
2. Create new issue with template (see below)
3. Include: problem statement, proposed solution, alternatives considered
4. Tag with `feature request` label
5. Community upvotes (👍) help prioritize

---

## Issue Templates

### Bug Report Template

```markdown
**Describe the bug**
A clear and concise description of what the bug is.

**To Reproduce**
Steps to reproduce the behavior:
1. Go to '...'
2. Click on '...'
3. Scroll down to '...'
4. See error

**Expected behavior**
A clear and concise description of what you expected to happen.

**Screenshots**
If applicable, add screenshots to help explain your problem.

**Environment:**
 - OS: [e.g. macOS, Windows, Linux]
 - Browser: [e.g. Chrome 120, Safari 17]
 - Version: [e.g. commit hash or release tag]

**Additional context**
Add any other context about the problem here.
```

### Feature Request Template

```markdown
**Is your feature request related to a problem? Please describe.**
A clear and concise description of what the problem is. Ex. I'm always frustrated when [...]

**Describe the solution you'd like**
A clear and concise description of what you want to happen.

**Describe alternatives you've considered**
A clear and concise description of any alternative solutions or features you've considered.

**Additional context**
Add any other context or screenshots about the feature request here.

**Willingness to contribute**
- [ ] I am willing to implement this feature myself and submit a PR.
```

---

## Pull Request Guidelines

### PR Title Format

**Use Conventional Commits**:
- `feat: add dark mode toggle`
- `fix: resolve message sending bug`
- `docs: update API documentation`
- `refactor: extract matching logic into service`
- `test: add unit tests for skill matching`
- `chore: update dependencies`

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, no logic change)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Build process, dependencies, tooling

---

### PR Description Template

```markdown
## What does this PR do?
Brief description of the changes.

## Related Issue
Fixes #123 (or Closes #123)

## Changes Made
- Change 1
- Change 2
- Change 3

## Screenshots (if applicable)
[Add screenshots or GIFs showing the changes]

## Checklist
- [ ] Code follows project style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex logic
- [ ] Documentation updated
- [ ] Tests added/updated
- [ ] All tests passing
- [ ] No new warnings or errors

## Additional Notes
[Any additional context or considerations]
```

---

### PR Review Process

**1. Automated Checks** (must pass):
- CI/CD pipeline (GitHub Actions)
- Linting (ESLint, Prettier)
- Type checking (TypeScript)
- Unit tests (Jest, pytest)
- Build succeeds

**2. Code Review** (by maintainers):
- Code quality and readability
- Test coverage
- Performance implications
- Security considerations
- Documentation completeness

**3. Approval & Merge**:
- At least 1 maintainer approval required
- All comments addressed
- No merge conflicts
- Squash and merge to main branch

---

## Coding Standards

### TypeScript/JavaScript (Frontend + Backend)

**Style Guide**: [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)

**Key Rules**:
- Use TypeScript for type safety
- Prefer `const` over `let`, never use `var`
- Use arrow functions for callbacks
- Use destructuring for objects and arrays
- Use async/await over promises

**Linting**:
```bash
npm run lint # ESLint + Prettier
npm run lint:fix # Auto-fix issues
```

**Example**:
```typescript
// Good
const fetchUser = async (userId: string): Promise<User> => {
  const { data, error } = await supabase
    .from('users')
    .select('*')
    .eq('id', userId)
    .single();

  if (error) throw error;
  return data;
};

// Bad
function fetchUser(userId) {
  return supabase.from('users').select('*').eq('id', userId).single().then(result => {
    if (result.error) throw result.error;
    return result.data;
  });
}
```

---

### Python (ML Services)

**Style Guide**: [PEP 8](https://peps.python.org/pep-0008/)

**Key Rules**:
- Use type hints for function arguments and return types
- Use snake_case for variables and functions
- Use docstrings for all functions
- Maximum line length: 88 characters (Black formatter)

**Linting**:
```bash
black . # Code formatter
flake8 . # Linter
mypy . # Type checker
```

**Example**:
```python
# Good
def compute_skill_similarity(user1_skills: List[str], user2_skills: List[str]) -> float:
    """Compute cosine similarity between two skill lists.
    
    Args:
        user1_skills: List of skill names for user 1
        user2_skills: List of skill names for user 2
    
    Returns:
        Similarity score between 0 and 1
    """
    embeddings1 = model.encode(user1_skills)
    embeddings2 = model.encode(user2_skills)
    return cosine_similarity(embeddings1, embeddings2)

# Bad
def computeSkillSim(s1,s2):
    e1=model.encode(s1)
    e2=model.encode(s2)
    return cosine_similarity(e1,e2)
```

---

### Git Commit Messages

**Format**: `<type>(<scope>): <subject>`

**Examples**:
- `feat(matching): add complementary skill gap analysis`
- `fix(messages): resolve real-time sync issue`
- `docs(api): update endpoint documentation`
- `test(profile): add integration tests for GitHub import`

**Best Practices**:
- Use present tense ("add" not "added")
- Keep subject line under 72 characters
- Separate subject from body with blank line
- Explain "what" and "why", not "how"

---

## Testing Standards

### Frontend Tests (Jest + React Testing Library)

**Test Coverage**: 80%+ for core features

**Example**:
```typescript
import { render, screen } from '@testing-library/react';
import MatchCard from './MatchCard';

describe('MatchCard', () => {
  it('renders match information correctly', () => {
    const match = {
      name: 'John Doe',
      role: 'Full-Stack Developer',
      matchScore: 85,
    };

    render(<MatchCard match={match} />);

    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('Full-Stack Developer')).toBeInTheDocument();
    expect(screen.getByText('85% match')).toBeInTheDocument();
  });
});
```

### Backend Tests (pytest)

**Example**:
```python
def test_compute_matches():
    """Test match computation for a user."""
    user_id = "user-123"
    matches = compute_matches(user_id)
    
    assert len(matches) == 20
    assert matches[0]['score'] >= matches[-1]['score'] # Sorted descending
    assert all(0 <= m['score'] <= 100 for m in matches)
```

---

## Communication Channels

### GitHub Discussions

**URL**: https://github.com/ArjunFrancis/openhr-platform/discussions

**Categories**:
- **Announcements**: Project updates, releases
- **General**: Open-ended discussions
- **Ideas**: Feature brainstorming
- **Q&A**: Ask questions, get answers
- **Show and Tell**: Share projects built with OpenHR

### Discord Server

**URL**: [Invite link in README]

**Channels**:
- `#general`: General discussion
- `#development`: Code-related questions
- `#design`: UI/UX design discussions
- `#ml-ai`: Machine learning and AI topics
- `#help`: Get help with setup, bugs
- `#showcase`: Show your contributions

### Twitter/X

**Handle**: [@OpenHRPlatform](https://twitter.com/OpenHRPlatform) (example)

**Use for**: Project announcements, community highlights, success stories

---

## Recognition & Rewards

### Contributor Badges

**Earned through contributions**:
- 🌟 **First-Time Contributor**: First merged PR
- 🚀 **Regular Contributor**: 5+ merged PRs
- 💎 **Core Contributor**: 20+ merged PRs or significant feature contributions
- 📚 **Documentation Hero**: 10+ documentation PRs
- 🎨 **Design Champion**: Major design contributions
- 🐛 **Bug Hunter**: 10+ bug fixes merged

### All Contributors

**Recognition**: Contributors listed in README.md and website

**Implementation**: [All Contributors](https://allcontributors.org/) bot

---

## Governance

### Decision Making

**Model**: Benevolent Dictator For Life (BDFL) transitioning to community governance

**Current Maintainers**:
- [@ArjunFrancis](https://github.com/ArjunFrancis) - Project Owner

**Future**: Add maintainers based on contributions and trust

### Roadmap & Prioritization

**Quarterly Planning**:
1. Community proposes features (GitHub Discussions)
2. Maintainers prioritize based on:
   - User impact
   - Community interest (upvotes)
   - Alignment with project vision
   - Contributor availability
3. Roadmap published in [ROADMAP.md](../../ROADMAP.md)

---

## License & Legal

**License**: MIT (permissive, business-friendly)

**Contributor License Agreement (CLA)**: Not required (MIT license covers it)

**Copyright**: Contributors retain copyright, grant license to OpenHR

**Third-Party Code**: Ensure compatible licenses (MIT, Apache 2.0, BSD)

---

## Resources

### For New Contributors

- [First Contributions](https://firstcontributions.github.io/) - Learn Git/GitHub
- [How to Contribute to Open Source](https://opensource.guide/how-to-contribute/)
- [OpenHR Architecture Docs](../architecture/system-architecture.md)

### For Maintainers

- [Maintainer Guide](https://opensource.guide/best-practices/)
- [Code Review Guide](https://google.github.io/eng-practices/review/)

---

## Contact

- **General Inquiries**: hello@openhr.com
- **Code of Conduct Violations**: conduct@openhr.com
- **Security Issues**: security@openhr.com (see [SECURITY.md](../../SECURITY.md))

---

**Document Owner**: OpenHR Community  
**Last Updated**: November 30, 2025  
**Version**: 1.0  
**Status**: Final
