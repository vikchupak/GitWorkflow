# Git workflows

#### 1. **Gitflow**
**Gitflow** is a popular branching model introduced by Vincent Driessen. It emphasizes a strict branching strategy for managing feature development, releases, and hotfixes, typically using:

- `main` (or `master`) — for production-ready code.
- `develop` — for integration and ongoing development.
- `feature/*` — for individual features.
- `release/*` — for preparing a new production release.
- `hotfix/*` — for quick patches to production.

### Pros of Gitflow:
- Structured and great for teams releasing software in cycles.
- Separates stable production code from active development.

### Cons:
- Can be overly complex for CI/CD and frequent deployments.
- Too many branches might slow teams down, especially in fast-paced environments.

---

#### 2. **GitHub Flow**
- **Branches**: `main`, `feature branches`
- **Process**: Every change is a feature branch off `main`, with pull requests and code reviews.
- **Deployment**: Continuous deployment; merge to `main` triggers deployment.

**Best for**: Continuous deployment, web apps, teams using GitHub Actions.

---

#### 3. **GitLab Flow**
- A hybrid of Gitflow and GitHub Flow.
- Can combine feature branches with environment-specific branches (e.g., `main`, `staging`, `production`).
- Supports issue tracking and merge request integration.

**Best for**: Teams using GitLab and needing a workflow that integrates CI/CD and issue tracking.

---

#### 4. **Trunk-Based Development**
- **Branches**: Mostly just `main` (a.k.a. `trunk`)
- **Process**: Developers work on short-lived branches or commit directly to `main` behind feature flags.
- Encourages frequent integration (multiple times daily).

**Best for**: High-performing teams, microservices, CI/CD-first environments.

---

#### 5. **Release Flow (Microsoft)**
- Based on long-lived `main`, with branch-per-release strategy.
- New features merge into `main`, and releases are branched off as needed.

**Best for**: Teams managing multiple parallel releases and needing flexibility.

---

### How to Choose?
| Criteria | Suggested Workflow |
|---------|---------------------|
| Frequent Deployments | GitHub Flow / Trunk-Based |
| Structured Release Cycle | Gitflow |
| Simplicity | GitHub Flow |
| Multiple Environments | GitLab Flow / Release Flow |
| Large Teams | Gitflow / Trunk-Based with feature flags |
