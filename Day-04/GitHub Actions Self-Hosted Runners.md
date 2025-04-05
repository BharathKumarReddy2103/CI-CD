# GitHub Actions with Self-Hosted Runners in Docker:

## Introduction

CI/CD pipelines are the backbone of modern DevOps workflows. GitHub Actions offers a streamlined CI/CD platform deeply integrated with GitHub repositories. While GitHub-hosted runners are suitable for many scenarios, using **self-hosted runners** in **Docker containers** gives you full control over the environment, dependencies, and resources.

In this article, you’ll learn how to set up a **self-hosted GitHub Actions runner using Docker**, understand the difference between GitHub-hosted and self-hosted runners, explore real-world use cases, and compare GitHub Actions with Jenkins.

---

## What Are GitHub Actions Runners?

### GitHub-Hosted Runners

- Managed and provisioned by GitHub
- Free for public repositories
- Short-lived and auto-terminated after each job
- Pre-installed with common runtimes and tools

### Self-Hosted Runners

- Hosted on your infrastructure (Docker, VM, bare metal)
- Persistent and customizable
- Full control over dependencies and resources
- Ideal for private repositories and enterprise use

---

## When to Use Self-Hosted Runners in Docker?

You should use Docker-based self-hosted runners when:

- You want **reproducible environments** that can be version-controlled
- Your application needs **specific software** or **custom libraries**
- You require **more memory/CPU** than GitHub-hosted runners offer
- You are working with **private repositories** and need enhanced **security**
- You want to **run jobs locally** without provisioning virtual machines

---

## Setting Up a GitHub Self-Hosted Runner in Docker

### Step 1: Clone the Official Runner Image

Use the official [GitHub Actions Runner image](https://github.com/actions/runner) to set up your container.

### Step 2: Create a Dockerfile

```Dockerfile
FROM ubuntu:22.04

# Install dependencies
RUN apt-get update && \
    apt-get install -y curl jq git sudo && \
    apt-get clean

# Create a user
RUN useradd -m -s /bin/bash runner
WORKDIR /home/runner

# Set environment variables
ENV RUNNER_VERSION=2.315.0
ENV RUNNER_HOME=/home/runner/actions-runner

# Download GitHub Actions runner
RUN curl -o actions-runner.tar.gz -L https://github.com/actions/runner/releases/download/v${RUNNER_VERSION}/actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz && \
    mkdir -p ${RUNNER_HOME} && \
    tar -zxf actions-runner.tar.gz -C ${RUNNER_HOME} && \
    rm actions-runner.tar.gz

# Copy entrypoint
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
```

### Step 3: Create the Entry Script (`entrypoint.sh`)

```bash
#!/bin/bash

# Configure the runner
cd /home/runner/actions-runner
./config.sh --url https://github.com/<OWNER>/<REPO> \
            --token $RUNNER_TOKEN \
            --name $(hostname) \
            --labels self-hosted,docker \
            --unattended \
            --replace

# Start the runner
./run.sh
```

> Replace `<OWNER>/<REPO>` with your GitHub repository path.

---

### Step 4: Build and Run the Docker Container

```bash
docker build -t gh-self-hosted-runner .

docker run -d \
  --name gh-runner \
  -e RUNNER_TOKEN=<YOUR_GENERATED_TOKEN> \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gh-self-hosted-runner
```

> You can get the `RUNNER_TOKEN` from:  
> `GitHub Repo → Settings → Actions → Runners → New self-hosted runner`

---

## Modify Your GitHub Workflow for Self-Hosted Runner

Update your `.github/workflows/ci.yml` file to use the self-hosted label:

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
    - name: Checkout
      uses: actions/checkout@v2

    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: ${{ matrix.python-version }}

    - name: Run Script
      run: python addition.py
```

---

## Security and Networking Best Practices

- Avoid exposing Docker daemons to the public
- Use **GitHub secrets** to manage credentials
- Isolate runners in **Docker networks**
- Limit container capabilities and add **resource quotas**

---

## GitHub Actions vs Jenkins

| Feature               | GitHub Actions (Docker Runner)  | Jenkins (Docker Agent)              |
|----------------------|----------------------------------|--------------------------------------|
| Setup Complexity     | Simple with containerization     | Medium to complex                    |
| Plugin Support       | Limited (growing)                | Extensive                            |
| Docker Integration   | Native support                   | Native + plugins                     |
| Cost (Public Repos)  | Free                             | Self-hosted, incurs infra cost       |
| Security             | Secrets via GitHub settings      | Plugin-based security                |
| Ideal For            | GitHub-centric projects          | Complex, large-scale CI pipelines    |

---

## Common Interview Questions & Answers

### 1. Why use Docker for self-hosted runners?

- Portability
- Easy resets via container restarts
- Consistency across environments
- Lightweight compared to VMs

### 2. How do you manage secrets securely?

Store sensitive values in:

```
GitHub → Settings → Secrets and variables → Actions
```

Access them in workflows via `${{ secrets.SECRET_NAME }}`.

### 3. How does GitHub Actions detect your Docker runner?

Once registered, your container listens for jobs via `run.sh`. On new commits or PRs, GitHub dispatches the job to available runners matching the `self-hosted` label.

---

## Best Practices

- Use labels (e.g., `--labels self-hosted,docker`) to organize runners
- Monitor runner health with Docker logs or external observability tools
- Auto-restart runners using Docker Compose or systemd
- Keep the runner image up to date with security patches

---

## Conclusion

Docker-based GitHub self-hosted runners give you full control over your CI/CD environment while leveraging GitHub's powerful workflow engine. Whether you want to run tests locally, enforce custom environments, or meet enterprise security policies, Docker offers the flexibility and repeatability you need.

For scalable and secure DevOps pipelines, this approach is both professional and production-ready.
