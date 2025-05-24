# 💡 How We Saved $5,000 💰 by Redesigning Our Dev CI/CD with GitHub Actions and k3s

# A real-world DevOps implementation where I redesigned our CI/CD architecture using GitHub Actions and k3s instead of AWS EKS and Flux CD, resulting in significant cost savings and improved Dev workflow efficiency.

## 🚀 Introduction

In our journey to optimize cloud costs and simplify infrastructure, I identified and implemented a game-changing DevOps architecture in our organization. By replacing AWS EKS and Flux CD in our development environment with a lightweight k3s-based Kubernetes cluster running inside GitHub Actions, we achieved over **$5,000 in annual savings**—without compromising CI/CD automation.

This post details the real-world architecture change I personally implemented, the reasons behind it, and how you can replicate it in your organization.

---

## 🔍 Our Previous Setup: Overkill for Dev CI/CD

### 🧱 The Old Architecture

- **Dev Environment** had a **3-node AWS EKS cluster** (t2.xlarge instances).
- **GitHub Enterprise** triggered CI workflows via GitHub Actions on every pull request.
- CI pipeline stages included:
  - ✅ Unit Tests
  - 🔍 Static Code Analysis
  - 🏗️ Binary Build & Push to Artifactory
  - 🐳 Docker Image Creation & Scanning
  - 📦 Push Image to Amazon ECR
  - 📝 Update Helm Chart values.yaml
- **Flux CD** monitored Helm charts and deployed changes to the EKS cluster.
- 🧪 Integration tests executed on the EKS cluster after deployment.

### ⚠️ Key Observations

- The entire process was automated—no manual QA or developer access to the EKS cluster.
- Despite using best practices, the setup was **over-engineered for the use case**.
- The **EKS cluster was running 24/7** only to serve automated test needs.
- Result: **Unnecessary cloud spend**, especially in lower environments (Dev/UAT).

---

## 🔁 What I Changed in Our Architecture

### 🧠 Strategy: Simplify & Optimize

Instead of keeping an always-on EKS + Flux CD setup, I re-architected the Dev CI/CD pipeline by:

- **Eliminating** AWS EKS and Flux CD from Dev environments.
- **Running k3s inside GitHub Runners** during each workflow.
- **Reusing GitHub Enterprise runner minutes**, which we were already paying for.

---

## 🛠️ Our New Dev CI/CD Pipeline with GitHub Actions + k3s

### 📌 Updated Workflow

1. **CI via GitHub Actions**:
   - ✅ Unit Testing
   - 🔍 Static Code Analysis
   - 🏗️ Build Binary → Push to Artifactory
   - 🐳 Build Docker Image → Scan → Push to Amazon ECR
   - 📝 Update Helm values.yaml

2. **CD on GitHub Runner**:
   - ⚡ Install `k3s` inside the runner VM
   - 🛠️ Use `helm upgrade/install` to deploy the application
   - 🧪 Run integration tests directly on k3s

```yaml
# Example: Install k3s on GitHub Runner
- name: Install k3s
  run: |
    curl -sfL https://get.k3s.io | sh -
    kubectl get nodes

# Example: Helm Deployment to k3s
- name: Helm Deploy
  run: |
    helm upgrade --install my-app ./helm-chart \
    --set image.tag=${{ github.sha }}

# Example: Run Integration Tests
- name: Run Tests
  run: |
    pytest tests/integration/
````

---

## 📉 Impact of This Change

| Metric                      | Before (EKS + FluxCD) | After (GitHub Actions + k3s) |
| --------------------------- | --------------------- | ---------------------------- |
| EKS Cost (per year)         | \$5,000+              | \$0 (EKS removed)            |
| Infrastructure Availability | Always-On             | Ephemeral GitHub Runners     |
| Complexity                  | High (EKS + Flux CD)  | Low (k3s + Shell scripts)    |
| CI/CD Duration              | Similar               | Similar                      |
| Developer Interaction       | None                  | None                         |

---

## 🧪 When to Use This Pattern

✅ Ideal For:

* Fully automated CI/CD in Dev or UAT.
* No need for developer or QA access to the Kubernetes cluster.
* Teams using GitHub Enterprise with included runner minutes.

🚫 Not Recommended For:

* Environments requiring persistent clusters (e.g., staging, pre-prod).
* Manual QA, performance testing, or sandbox checks.
* Scenarios with shared EKS access across teams.

---

## 🧭 Best Practices We Followed

* Keep Dev architecture **lean and purpose-fit**.
* **Avoid persistent infrastructure** where possible.
* Leverage GitHub Enterprise runner **minutes and resources effectively**.
* Use **k3s** for lightweight Kubernetes testing—fast and reliable.
* Run integration tests **only when needed**, not 24/7.

---

## 🎯 Conclusion

Cloud cost optimization isn’t just about using fewer resources—it’s about **re-thinking architectures** to fit the actual use case. By implementing this change in our company, we not only saved thousands of dollars but also simplified our Dev CI/CD pipeline, making it faster and easier to maintain.

This pattern won’t work for every environment, but for automated Dev workflows, it's a **powerful and production-ready alternative**.

---

## 🙌 Want to Implement This?

If you're interested in seeing a working demo of this pipeline, sample repo, or want to replicate this in your organization, feel free to star the repo, fork it, or connect on LinkedIn.

---

> ⭐ If this article helped you, please consider giving the repo a star and sharing it with your network.
