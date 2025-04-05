# Software environments

In software development, different **environments** serve different purposes in the development and deployment lifecycle. Each environment helps isolate stages of work and testing to ensure quality before reaching production.

---

## 🌍 Common Environments

### 1. **Development (dev)**
- **Who uses it?** Developers.
- **Purpose:** Active coding, integration of new features.
- **Access:** Local or internal.
- **Typical setup:** May use mock data, debug mode enabled.
- **CI/CD:** May auto-deploy on every commit or merge to `develop` branch.

---

### 2. **Testing (test)**
- **Who uses it?** Developers, QA engineers.
- **Purpose:** Automated or manual tests (unit, integration).
- **Access:** Internal only.
- **Typical setup:** Mirrors dev but can include CI test suites, mock services.
- **CI/CD:** Run test suites automatically on pull requests or dev merges.

---

### 3. **Staging**
- **Who uses it?** QA, Product Owners, Stakeholders.
- **Purpose:** Final testing with production-like setup before going live.
- **Access:** Internal, sometimes semi-public.
- **Typical setup:** Same as production — same DB structure, third-party integrations.
- **CI/CD:** Triggered on `staging` branch merges.

✅ **Mimics production** as closely as possible.

---

### 4. **User Acceptance Testing (UAT)**
- **Who uses it?** Business users, clients, beta testers.
- **Purpose:** Verify if the solution meets business requirements.
- **Access:** Restricted, external testers or clients.
- **Typical setup:** Close to staging or could be its own branch/env.
- **CI/CD:** Optional, may be manual deploy for precise control.

🗣️ UAT is like staging but focused on **business validation**, not just technical correctness.

---

### 5. **Production (prod)**
- **Who uses it?** End users.
- **Purpose:** Live system where real users interact.
- **Access:** Public (typically).
- **Typical setup:** Fully optimized, with monitoring, error tracking, scaling, backups, etc.
- **CI/CD:** Strict control — usually triggered only after approvals.

🚨 Mistakes here have real-world consequences.

---

## 🛣️ Common Flow Example:

```plaintext
feature/* --> develop --> test --> staging --> UAT --> main (production)
```

Or a simplified version with GitHub Flow:

```plaintext
feature/* --> staging --> main
```

---

## 🧩 Tips
- Use **feature flags** to test in production safely.
- Always keep **staging and prod environments as similar as possible** (infra, data, auth).
- Use **infrastructure-as-code** (e.g., Terraform) to avoid config drift between environments.
- Consider **data masking** or **mocking** in non-prod environments for security.

---

Want help mapping this to your own project or setting up CI/CD to handle these environments?
