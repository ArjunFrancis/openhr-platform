# Contributing to OpenHR Platform

Thank you for your interest in contributing to OpenHR! 🎉

OpenHR is an open-source, AI-powered talent matching platform for co-founders and developers. We welcome contributions from developers, designers, researchers, and community members of all skill levels.

---

## 📋 Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How Can I Contribute?](#how-can-i-contribute)
- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Pull Request Process](#pull-request-process)
- [Coding Standards](#coding-standards)
- [Commit Message Guidelines](#commit-message-guidelines)
- [Community & Support](#community--support)

---

## Code of Conduct

This project adheres to the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code. Please report unacceptable behavior to [arjunfrancis21@gmail.com](mailto:arjunfrancis21@gmail.com).

---

## How Can I Contribute?

### 1. Reporting Bugs 🐛

If you find a bug, please create an issue with:
- A clear, descriptive title
- Steps to reproduce the issue
- Expected vs. actual behavior
- Screenshots or error messages (if applicable)
- Your environment (OS, browser, Node.js version, etc.)

**Before reporting**, please search existing issues to avoid duplicates.

### 2. Suggesting Features 💡

We welcome feature requests! Please create an issue with:
- A clear description of the feature
- Why it would be useful (user stories or use cases)
- Any relevant examples from other platforms
- Potential implementation ideas (optional)

### 3. Improving Documentation 📚

Documentation is critical for an open-source project. You can help by:
- Fixing typos or unclear explanations
- Adding examples or tutorials
- Improving API documentation
- Translating documentation (future)

### 4. Contributing Code 💻

We accept code contributions via pull requests. See the [Development Workflow](#development-workflow) section below.

**Areas where we need help:**
- Frontend: React/Next.js components, UI/UX improvements
- Backend: API endpoints, database optimizations
- ML/AI: Skill matching algorithms, profile enrichment pipelines
- Testing: Unit tests, integration tests, E2E tests
- DevOps: CI/CD improvements, Docker configurations

### 5. Community Support 🤝

- Answer questions in GitHub Discussions
- Help triage issues
- Review pull requests from other contributors
- Share OpenHR on social media and with your network

---

## Getting Started

### Prerequisites

- **Node.js**: 18+ (for frontend and backend)
- **Python**: 3.10+ (for ML service)
- **Docker**: Latest version (for local development)
- **Git**: Latest version
- **GitHub Account**: For forking and creating PRs

### Local Setup

1. **Fork the repository**
   ```bash
   # Click "Fork" on GitHub, then clone your fork:
   git clone https://github.com/YOUR_USERNAME/openhr-platform.git
   cd openhr-platform
   ```

2. **Set up environment variables**
   ```bash
   cp .env.example .env
   # Edit .env with your Supabase, OpenAI, GitHub API keys
   ```

3. **Install dependencies**
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

4. **Start services with Docker Compose**
   ```bash
   docker-compose up
   ```

5. **Access the application**
   - Frontend: http://localhost:3000
   - Backend API: http://localhost:4000
   - ML Service: http://localhost:8000
   - Supabase Studio: http://localhost:54323

### Running Tests

```bash
# Frontend tests
cd frontend && npm test

# Backend tests
cd backend && npm test

# ML service tests
cd ml-service && pytest

# E2E tests
npm run test:e2e
```

---

## Development Workflow

### 1. Create a Branch

Always work on a feature branch, not `main`.

```bash
git checkout -b feature/your-feature-name
# or
git checkout -b fix/bug-description
```

**Branch Naming Conventions:**
- `feature/` for new features (e.g., `feature/skill-endorsements`)
- `fix/` for bug fixes (e.g., `fix/messaging-realtime-bug`)
- `docs/` for documentation updates (e.g., `docs/update-readme`)
- `refactor/` for code refactoring (e.g., `refactor/api-error-handling`)
- `test/` for adding tests (e.g., `test/profile-enrichment`)

### 2. Make Your Changes

- Write clean, readable code following our [Coding Standards](#coding-standards)
- Add tests for new functionality
- Update documentation if needed
- Ensure all tests pass locally

### 3. Commit Your Changes

Follow our [Commit Message Guidelines](#commit-message-guidelines).

```bash
git add .
git commit -m "feat: add skill endorsement feature"
```

### 4. Push to Your Fork

```bash
git push origin feature/your-feature-name
```

### 5. Create a Pull Request

Go to the [OpenHR repository](https://github.com/ArjunFrancis/openhr-platform) and click **"New Pull Request"**.

---

## Pull Request Process

### Before Submitting

- [ ] All tests pass (`npm test`, `pytest`)
- [ ] Code follows our coding standards
- [ ] Commit messages follow our guidelines
- [ ] Documentation is updated (if applicable)
- [ ] You've tested your changes locally
- [ ] Branch is up to date with `main`

### PR Description Template

Use this template when creating your PR:

```markdown
## Description
Brief description of what this PR does.

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Documentation update
- [ ] Refactoring
- [ ] Performance improvement
- [ ] Test coverage improvement

## Related Issue(s)
Fixes #123
Related to #456

## How Has This Been Tested?
- [ ] Unit tests
- [ ] Integration tests
- [ ] Manual testing
- [ ] E2E tests

## Screenshots (if applicable)
[Add screenshots here]

## Checklist
- [ ] My code follows the project's coding standards
- [ ] I have added tests that prove my fix/feature works
- [ ] All new and existing tests pass
- [ ] I have updated the documentation
- [ ] My commit messages follow the commit guidelines
```

### Review Process

1. **Automated Checks**: CI/CD pipeline runs tests and linting
2. **Code Review**: At least one maintainer reviews your code
3. **Revisions**: Address feedback and push updates
4. **Approval**: Once approved, a maintainer will merge your PR

**Response Time:** We aim to review PRs within 3-5 business days.

---

## Coding Standards

### General Principles

- **Write readable code**: Code is read more often than written
- **Keep it simple**: Avoid over-engineering
- **Don't repeat yourself (DRY)**: Extract common logic into reusable functions
- **Test your code**: Aim for 80%+ test coverage on new code
- **Document complex logic**: Add comments for non-obvious code

### Language-Specific Standards

#### JavaScript/TypeScript (Frontend & Backend)

- **Style Guide**: Follow [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
- **Linting**: Run `npm run lint` before committing
- **Formatting**: Use Prettier (configured in `.prettierrc`)
- **Naming Conventions**:
  - Variables/functions: `camelCase`
  - Components: `PascalCase`
  - Constants: `UPPER_SNAKE_CASE`
  - Files: `kebab-case.tsx` or `PascalCase.tsx` for components

**Example:**
```typescript
// Good
const fetchUserProfile = async (userId: string): Promise<Profile> => {
  const response = await api.get(`/profiles/${userId}`);
  return response.data;
};

// Bad
const FetchUserProfile = async (user_id) => {
  const resp = await api.get('/profiles/' + user_id);
  return resp.data;
};
```

#### Python (ML Service)

- **Style Guide**: Follow [PEP 8](https://pep8.org/)
- **Linting**: Run `flake8` or `pylint` before committing
- **Formatting**: Use `black` for auto-formatting
- **Type Hints**: Use type hints for function signatures
- **Docstrings**: Use Google-style docstrings

**Example:**
```python
# Good
def compute_similarity(
    embedding_1: np.ndarray,
    embedding_2: np.ndarray
) -> float:
    """
    Compute cosine similarity between two embeddings.

    Args:
        embedding_1: First embedding vector
        embedding_2: Second embedding vector

    Returns:
        Cosine similarity score between 0 and 1
    """
    return np.dot(embedding_1, embedding_2) / (
        np.linalg.norm(embedding_1) * np.linalg.norm(embedding_2)
    )

# Bad
def compute_sim(e1, e2):
    return np.dot(e1, e2) / (np.linalg.norm(e1) * np.linalg.norm(e2))
```

### Database

- Use snake_case for table and column names
- Always add indexes for foreign keys and frequently queried columns
- Write migrations for schema changes (never edit tables directly)
- Use transactions for multi-step operations

### API Design

- Follow RESTful conventions (`GET`, `POST`, `PUT`, `DELETE`)
- Use plural nouns for resources (`/profiles`, `/matches`)
- Version your APIs (`/v1/profiles`)
- Return consistent error responses:
  ```json
  {
    "error": {
      "code": "VALIDATION_ERROR",
      "message": "Invalid user_id",
      "details": { "field": "user_id", "issue": "must be a valid UUID" }
    }
  }
  ```

---

## Commit Message Guidelines

We follow [Conventional Commits](https://www.conventionalcommits.org/) for clear commit history.

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, no logic change)
- `refactor`: Code refactoring (no feature change)
- `perf`: Performance improvements
- `test`: Adding or updating tests
- `chore`: Maintenance tasks (dependencies, CI, etc.)

### Examples

```bash
# Good
feat(matching): add timezone-based filtering
fix(auth): resolve JWT token refresh bug
docs(readme): update installation instructions
test(profile): add unit tests for profile enrichment

# Bad
Added timezone filtering
Fixed bug
Updated docs
```

### Detailed Example

```
feat(matching): add multi-factor scoring algorithm

Implement a new scoring system that combines:
- Semantic skill similarity (40%)
- Timezone compatibility (20%)
- Equity preferences (20%)
- Startup stage alignment (20%)

This improves match quality by 15% based on user feedback.

Closes #234
```

---

## Community & Support

### Communication Channels

- **GitHub Issues**: Bug reports, feature requests
- **GitHub Discussions**: Questions, ideas, show-and-tell
- **Email**: [arjunfrancis21@gmail.com](mailto:arjunfrancis21@gmail.com) for sensitive topics

### Getting Help

- Check the [documentation](docs/) first
- Search existing issues and discussions
- Ask in GitHub Discussions if you're stuck
- Tag maintainers with `@ArjunFrancis` if urgent

### Recognition

All contributors will be recognized in:
- The project's README (Contributors section)
- Release notes for significant contributions
- Special shoutouts in community updates

---

## License

By contributing to OpenHR, you agree that your contributions will be licensed under the [MIT License](LICENSE).

---

## Thank You! 🙏

Your contributions make OpenHR better for everyone. We appreciate your time and effort!

**Questions?** Open a discussion or reach out to [@ArjunFrancis](https://github.com/ArjunFrancis).

---

**Built with ❤️ by the OpenHR Community**