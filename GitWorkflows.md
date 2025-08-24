# Git workflows (or "branching strategy")

# Semantic Versioning

Tags follow the format vMAJOR.MINOR.PATCH

# Tags

???

# 1. **Gitflow**

- https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow

**Gitflow** is a popular branching model introduced by Vincent Driessen. It emphasizes a strict branching strategy for managing feature development, releases, and hotfixes, typically using:

- `main` (or `master`) — for production-ready code.
- `develop` — for integration and ongoing development.
- `feature/*` — for individual features.
- `release/*` — **for preparing a new production release.**
- `hotfix/*` — for quick patches to production.

---

- **Has environment branches**
- **Production deploy happens via RELEASE BRANCH.**
- **Deployment happens when you decide to ship a release.**
- **Deployment = manual, release-driven.**

### Pros of Gitflow:
- Structured and great for teams releasing software in cycles.
- Separates stable production code from active development.

### Cons:
- Can be overly complex for CI/CD and frequent deployments.
- Too many branches might slow teams down, especially in fast-paced environments.

---

# 2. **GitHub Flow**

- https://githubflow.github.io/

---

- **Branches**: `main`, `feature branches`
- **Process**: Every change is a feature branch off `main`, with pull requests and code reviews.
- **Deployment**: Continuous deployment; merge to `main` triggers deployment.

**Best for**: Continuous deployment, web apps, teams using GitHub Actions.

---

### 🔹 1. GitHub Flow basics

* **Main (master) branch** is always **deployable** (production-ready).
* **Feature branches** are created from `main` (or `master` in older setups).

  * Example: `feature/add-login`, `bugfix/cart-issue`.
* Developers work on those feature branches, push commits, and open a **Pull Request (PR)** back to `main`.
* Once approved & tested, the branch is merged into `main` → triggers deployment to **production**.

So yes: feature branches are usually created from `main/master`.

---

### 🔹 2. CI/CD on feature branches

* Typically, **CI pipelines** run on every push to **any branch** (including feature branches).
* Common tasks:

  * Run **unit tests**
  * Run **linting / static analysis**
  * Build artifacts (Docker image, JAR, etc.)
* This ensures problems are caught before merging to `main`.

So: **yes, tests and builds run automatically for feature branches**.

---

## 🔹 3. GitHub Flow and environments

Strictly speaking, **pure GitHub Flow** (the way GitHub first described it) has just one environment:

* **`main` branch → production**
  That’s it. No staging/dev, because GitHub Flow was designed for **continuous deployment**: every merge to `main` goes live.

---

## 🔹 How teams extend GitHub Flow with environments

In practice, many teams **layer environments on top of GitHub Flow** without changing the branching strategy:

### Option 1: **Preview environments (per feature branch)**

* Every feature branch gets deployed automatically to an **ephemeral environment** (temporary, isolated).
* Example:

  * Branch: `feature/add-payments` → deploys to `add-payments.dev.example.com`
* Useful for QA/PM to test before merge.
* Destroyed after the PR is merged or closed.

### Option 2: **Protected environments with rules**

GitHub has the concept of **Environments** (Settings → Environments), e.g.:

* `development`
* `staging`
* `production`

You can configure GitHub Actions like:

```yaml
on:
  push:
    branches:
      - 'feature/*'
      - 'main'

jobs:
  deploy-preview:
    if: startsWith(github.ref, 'refs/heads/feature/')
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying preview for ${{ github.ref }}"

  deploy-production:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - run: echo "Deploying to production"
```

* **Feature branches** → deploy to **preview env**
* **Main** → deploy to **production**

You can add `staging` by either:

* Using a **separate branch** like `staging` (merge PRs into it before `main`), or
* Tagging commits and mapping tags to environments.

---

## 🔹 So does GitHub Flow “use envs”?

* **Original GitHub Flow**: no, only `main → production`.
* **Real-world GitHub Flow**: yes, you can map feature branches to preview environments and protect `main` for production.

This keeps the simplicity of GitHub Flow (no `develop/release` branches like Git Flow), but still supports multiple environments.


✅ **Summary**:

* Feature branches are created from `main`.
* CI runs tests/builds on every feature branch.
* Deployment depends on workflow: either just `main` → prod, or branches mapped to staging/dev/prod environments.

## How about testers? how they test a product with original github flow with only one env?

Ah, good catch 👍
This is exactly why **pure GitHub Flow** (only `main → production`) works mostly for **small teams** or products where you can safely push code live multiple times a day.

Let’s break it down:

---

## 🔹 Original GitHub Flow (only `main → prod`)

* Developers open PRs → all automated tests & code reviews happen before merge.
* Once merged → code goes straight to **production**.
* **Testing is done before merge**:

  * Unit tests
  * Integration tests
  * Automated end-to-end tests
  * Possibly manual QA on a **local/dev environment** (run the branch locally or in Docker)

⚠️ But testers (QA) don’t have a dedicated environment unless:

* They check out the branch locally, or
* You spin up a temporary environment (like a preview deployment for the branch).

That’s why original GitHub Flow assumes:

* Strong **automated test coverage**
* Small, incremental changes that are low-risk
* Quick rollback if something breaks

---

## 🔹 How real teams adapt it for testers

Most companies **extend GitHub Flow** to support QA without going full Git Flow:

1. **Ephemeral (preview) environments**

   * Every feature branch deploys automatically to its own isolated environment.
   * QA can test new features **before merge** without affecting production.
   * Example:

     * PR #123 → deployed to `pr-123.dev.example.com`

2. **Shared staging environment**

   * Some teams add a `staging` branch or special deployment trigger.
   * Merging selected PRs to `staging` gives QA one central place to test features together before they hit `main`.

3. **Feature flags / toggles**

   * Merge directly to `main` (production) but keep new functionality hidden behind a flag.
   * QA can enable the flag in production and test, without exposing it to users.

---

## 🔹 Summary

* **Pure GitHub Flow** = only `main → prod`, so testers rely heavily on **automated tests** and sometimes manual testing in local/dev setups.
* **In practice** = teams add **preview envs, staging, or feature flags** so QA can validate safely before production.

---

# 3. **GitLab Flow**
- A hybrid of Gitflow and GitHub Flow.
- Can combine feature branches with environment-specific branches (e.g., `main`, `staging`, `production`).
- Supports issue tracking and merge request integration.

**Best for**: Teams using GitLab and needing a workflow that integrates CI/CD and issue tracking.

---

- **Like gitflow but without RELEASE BRANCH. Deploy to envs: production, staging, development**

---

# 4. **Trunk-Based Development**

- https://www.atlassian.com/continuous-delivery/continuous-integration/trunk-based-development

---

- **Branches**: Mostly just `main` (a.k.a. `trunk`)
- **Process**: Developers work on short-lived branches or commit directly to `main` behind feature flags.
- Encourages frequent integration (multiple times daily).

**Best for**: High-performing teams, microservices, CI/CD-first environments.

---

Trunk-Based is the same as GitHub Flow but with feature flags, and features enabled only for test users (Targeted Rollout/User-targeted feature flag).

---

# 5. **Release Flow (Microsoft)**
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
