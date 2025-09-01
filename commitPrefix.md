# Conventional commits & semantic versioning

- https://github.com/conventional-changelog/conventional-changelog

---

A list of conventional commit prefixes (also known as **commit types**) along with their **meanings**:

---

### ✅ **Main Conventional Commit Types**

| Prefix     | Meaning                                                                 |
| ---------- | ----------------------------------------------------------------------- |
| `feat`     | A new feature                                                           |
| `fix`      | A bug fix                                                               |
| `docs`     | Documentation changes only                                              |
| `style`    | Code style changes (formatting, missing semi-colons, etc.)              |
| `refactor` | Code changes that neither fix a bug nor add a feature                   |
| `perf`     | A code change that improves performance                                 |
| `test`     | Adding or updating tests                                                |
| `build`    | Changes that affect the build system or external dependencies           |
| `ci`       | Changes to CI/CD configuration files and scripts                        |
| `chore`    | Other changes that don’t modify src or test files (e.g., build scripts) |
| `revert`   | Reverts a previous commit                                               |

---

### 🧩 **Less Common / Optional Types**

| Prefix      | Meaning                                                             |
| ----------- | ------------------------------------------------------------------- |
| `wip`       | Work in progress – not yet ready                                    |
| `deps`      | Updating dependencies                                               |
| `merge`     | Merge branches (mostly auto-generated, sometimes excluded by tools) |
| `security`  | Security-related improvements                                       |
| `infra`     | Infrastructure-related changes                                      |
| `config`    | Configuration changes (e.g., `.eslintrc`, `.prettierrc`)            |
| `release`   | Version bump or changelog update                                    |
| `hotfix`    | A quick fix to a critical issue in production                       |
| `init`      | Initial project setup                                               |
| `upgrade`   | Upgrade of libraries, services, SDKs, etc.                          |
| `downgrade` | Downgrade of libraries or services                                  |

---

### 🧠 Notes

* These prefixes are widely used in **Conventional Commits** spec: [https://www.conventionalcommits.org](https://www.conventionalcommits.org)

* Format:

  ```
  <type>(optional scope): <description>
  ```

  Example:

  ```
  feat(auth): add OAuth2 support for Google
  fix(login): correct token expiration logic
  ```

* Tools like **semantic-release**, **commitlint**, and **standard-version** rely on these for automating changelogs and versioning.

Would you like a `.commitlintrc` config or templates for commit messages too?
