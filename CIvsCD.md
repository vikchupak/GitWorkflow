The terms **CI (Continuous Integration)** and **CD (Continuous Delivery / Continuous Deployment)** are closely related but refer to different stages in the software development lifecycle.

---

### **CI – Continuous Integration**

**Goal:**
- Automatically build and test code **every time it's pushed** to your version control (like GitHub).
- So CI ensures the code is **ready to deploy**, but **it doesn’t actually deploy**.

**CI Answers:**
> “Does the code build? Does it pass the tests?”

**Key Points:**

* Ensures code changes integrate cleanly.
* Runs automated tests, linters, build steps, etc.
* Detects bugs or conflicts early.
* Common tools: GitHub Actions, Jenkins, GitLab CI, CircleCI

**Example CI flow:**

* Developer pushes code →
* GitHub Actions runs `npm test`, `eslint`, `build` →
* Notifies team if it passes or fails.

---

### **CD – Continuous Delivery / Continuous Deployment**

CD has **two interpretations**:

#### 1. **Continuous Delivery** (safer, manual release)

* Automatically prepares builds for deployment (e.g., staging).
* **Deploy to production is manual**, but all artifacts are ready.
* Still useful for frequent, safe releases.

**Think:**

> “Code is always shippable, but we press the button.”

#### 2. **Continuous Deployment** (fully automated)

* Takes it one step further — **automatically deploys** code to production once CI passes.
* Zero human intervention.

**Think:**

> “CI passed? Ship it to production now.”

---

### 🧠 Summary

| Term   | Description                                      | Key Focus           |
| ------ | ------------------------------------------------ | ------------------- |
| **CI** | Automate testing/building code after each commit | Code correctness    |
| **CD** | Automate delivering code to staging/production   | Fast, safe releases |
