# GitHub Actions with Self-Hosted Runners :

## Introduction

In today’s DevOps world, Continuous Integration and Continuous Deployment (CI/CD) are crucial for delivering high-quality software quickly and reliably. GitHub Actions has become a popular CI/CD tool, especially for teams already using GitHub for source control. While GitHub-hosted runners are sufficient for many projects, enterprise-grade applications often require more control, scalability, and customization.

This article walks you through the use of **self-hosted runners** with **GitHub Actions**, using **AWS EC2** as the runner host. We’ll explore the difference between GitHub-hosted and self-hosted runners, real-world use cases, security best practices, and even compare GitHub Actions with Jenkins for better decision-making.

---

## What Are GitHub Actions Runners?

In CI/CD systems, **runners** (or **agents**) are the compute environments where jobs execute.

### GitHub-Hosted Runners

- **Provided by GitHub**
- **Free for public repositories**
- **Temporary**—spun up for each job and destroyed after execution
- **Pre-configured** with popular runtimes like Node.js, Python, Java, etc.

### Self-Hosted Runners

- **Hosted on your infrastructure** (e.g., AWS EC2, on-prem servers)
- **Persistent**—you control the environment
- Ideal for **private repositories**, **custom hardware**, or **security-sensitive workloads**

---

## When to Use Self-Hosted Runners?

Consider self-hosted runners when:

- You're working with **private repositories**
- You need **custom environments** or **large resources** (e.g., 32GB RAM)
- You're developing **security-sensitive applications** (e.g., banking)
- You need **special dependencies** not available on GitHub-hosted runners

---

## Setting Up a Self-Hosted Runner on AWS EC2

### Step 1: Launch an EC2 Instance

1. Navigate to the **EC2 Dashboard**
2. Click **Launch Instance**
3. Choose **Ubuntu** as the OS
4. Use default instance type (t2.micro or as needed)
5. Assign an existing **key pair**
6. Configure **Security Group**:
   - Inbound Rules: Open **port 80 (HTTP)** and **443 (HTTPS)**
   - Outbound Rules: Open **port 80 (HTTP)** and **443 (HTTPS)**

> **Tip**: Do *not* use `All traffic` for security reasons. Always follow the principle of least privilege.

---

### Step 2: Register the Runner in GitHub

1. Navigate to your repository → **Settings** → **Actions** → **Runners**
2. Click **New self-hosted runner**
3. Choose OS type and architecture (e.g., `Linux`, `x64`)
4. Follow the on-screen instructions:
   ```bash
   # Example commands (do not share the token!)
   mkdir actions-runner && cd actions-runner
   curl -o actions-runner-linux-x64-2.x.x.tar.gz -L https://github.com/actions/runner/releases/download/v2.x.x/actions-runner-linux-x64-2.x.x.tar.gz
   tar xzf ./actions-runner-linux-x64-2.x.x.tar.gz
   ./config.sh --url https://github.com/<owner>/<repo> --token <generated_token>
   ./run.sh
   ```

5. You should see: `Runner is listening for jobs...`

---

### Step 3: Modify Your GitHub Workflow

Update your `.github/workflows/<workflow>.yml` file:

```yaml
name: Python CI

on:
  push:

jobs:
  build:
    runs-on: self-hosted
    strategy:
      matrix:
        python-version: [3.8, 3.9]

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: ${{ matrix.python-version }}

    - name: Run addition script
      run: python addition.py
```

---

## Why Inbound and Outbound Rules Matter

- **Inbound rules**: Allow GitHub to send jobs to the runner
- **Outbound rules**: Allow the runner to send results back to GitHub

Without these, your runner may fail to communicate, causing CI jobs to hang or fail.

---

## GitHub Actions vs Jenkins: Which One to Choose?

| Feature               | GitHub Actions                  | Jenkins                             |
|-----------------------|----------------------------------|--------------------------------------|
| Hosting               | Cloud (GitHub-hosted/self-hosted)| Self-hosted (fully)                  |
| Plugin Ecosystem      | Growing                         | Mature and rich                      |
| Integration           | Best with GitHub                | Works with any source control system |
| Cost (Public Projects)| Free                            | Requires your own infrastructure     |
| UI/UX                 | Modern                          | Traditional                          |
| Learning Curve        | Low                             | Moderate                             |

### Summary:
- **Use GitHub Actions** for **open-source** or fully GitHub-based workflows.
- **Use Jenkins** for **private projects** requiring complex orchestration and extensive plugin support.

---

## Interview-Ready Questions & Answers

### 1. Why did you choose GitHub Actions over Jenkins or AWS CodeBuild?

If your project is open-source or already hosted on GitHub, GitHub Actions provides a seamless, native CI/CD experience with free runners and strong integration with GitHub features like Projects, Wikis, and Dependabot.

### 2. How do you secure secrets in GitHub Actions?

By using the **Settings > Secrets and Variables** section in GitHub, you can securely store sensitive data like API keys, tokens, and credentials.

### 3. How do you define GitHub workflows?

Inside the repo, create a file at:

```
.github/workflows/<workflow-name>.yml
```

Define the `on:` trigger (e.g., `push`, `pull_request`), and the job steps.

---

## Practical Use Case: Deploying a Python Script

You can use GitHub Actions to test and deploy a simple Python script (`addition.py`) using self-hosted runners. Just replace the runner label in your YAML and watch the jobs execute from your own infrastructure.

---

## Best Practices for Using Self-Hosted Runners

- Keep the runner updated with the latest version from GitHub
- Run runners in isolated environments (e.g., Docker containers)
- Use auto-scaling for large-scale jobs
- Monitor logs and metrics
- Never expose your runner token publicly

---

## Conclusion

GitHub Actions with self-hosted runners offers powerful flexibility for teams that need control, customization, and security. Whether you're deploying from AWS, using a private VPC, or running enterprise applications, this setup ensures performance and reliability.

Understanding when to use GitHub-hosted vs self-hosted runners is critical for both DevOps engineers and architects. With this guide, you're ready to implement and even defend your choices in interviews and real-world production pipelines.
