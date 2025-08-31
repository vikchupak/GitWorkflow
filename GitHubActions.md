### Github - Jenkins map
- Github workflow = Jenkins pipeline
- Github job = Jenkins stage
- Github step = Jenkins stage
- [Jenkins info](https://github.com/vikchupak/Jenkins/blob/main/jobs.md)

### Settings
- Secrets are in repo settings

### Tutorial
- [Tutorial](https://www.youtube.com/watch?v=R8_veQiYBjI)

<img width="967" height="547" alt="image" src="https://github.com/user-attachments/assets/f76cfcd1-aef4-495c-9f1d-639e8780b390" />

<img width="969" height="551" alt="image" src="https://github.com/user-attachments/assets/22308164-c8f9-4af5-baae-cc64007e6436" />

```
GitHub Actions (Platform)
  └── Workflows (YAML file)
         └── Jobs (Individual task groups)
                └── Steps(tasks) (Commands or actions)
```

Let's clarify and compare **GitHub Actions**, **GitHub Workflows**, and **GitHub Jobs** — these are not competing technologies but **hierarchical building blocks** within GitHub's CI/CD system:

---

### **GitHub Actions**

GitHub Actions is the **automation platform** within GitHub. It allows you to run custom scripts on **GitHub-hosted** or self-hosted **runners**, triggered by GitHub events (e.g., push, pull request).

---

### **GitHub Workflows**

A **workflow** is a **YAML configuration file** (usually stored in `.github/workflows/`) that defines **when and how** your automation should run.

* Triggered by events like `push`, `pull_request`, `schedule`, or manual `workflow_dispatch`.
* A workflow contains **one or more jobs**.

---

### **GitHub Jobs**

A **job** is a **set of steps(tasks)** that run in the same **environment (RUNNER - VM)**. Jobs run either in **parallel** or **sequentially** (if you specify dependencies with `needs:`).

* Jobs are part of a workflow.
* Jobs run in clean environments (Docker container, virtual machine, or your own runner).

---

### Summary Table:

| Term           | Type          | Role / Scope                     | Where it’s Defined          |
| -------------- | ------------- | -------------------------------- | --------------------------- |
| GitHub Actions | Platform      | Automation engine                | Built into GitHub           |
| Workflow       | YAML file     | Defines trigger + jobs           | `.github/workflows/*.yml`   |
| Job            | Workflow unit | Contains steps, runs on a runner | Inside a workflow YAML file |
| Step           | Sub-unit      | Individual command or action     | Inside a job                |
