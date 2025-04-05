# Practical CI/CD with Docker Agents and Kubernetes Deployment

## Introduction

In the modern DevOps landscape, continuous integration and continuous delivery (CI/CD) have become foundational practices for delivering reliable, scalable, and fast software. Jenkins, being one of the most powerful open-source automation tools, continues to be widely used in CI/CD pipelines across organizations.

This guide walks you through a **complete Jenkins setup** from scratch:
- Installing Jenkins on an EC2 instance
- Exposing Jenkins to the internet
- Configuring Docker as Jenkins agents
- Running pipelines using containerized agents
- Deploying applications to a Kubernetes cluster using ArgoCD

Whether you're a beginner or a professional DevOps engineer, this tutorial will help you understand **how Jenkins can be used with Docker and Kubernetes** to optimize CI/CD workflows.

---

## Installing Jenkins on AWS EC2

### Step 1: Launch EC2 Instance

- Use **Ubuntu** as the base AMI for compatibility with APT commands.
- Open port **8080** in the **inbound rules** of the EC2 security group.

### Step 2: SSH into EC2

```bash
ssh -i your-key.pem ubuntu@<public-ip>
```

### Step 3: Install Java and Jenkins

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
```

Add Jenkins repository and install:

```bash
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
sudo systemctl start jenkins
```

### Step 4: Access Jenkins

- Navigate to: `http://<public-ip>:8080`
- Unlock Jenkins using:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

- Install **Suggested Plugins**

---

## Configuring Jenkins with Docker

### Step 1: Install Docker

```bash
sudo apt install docker.io -y
```

### Step 2: Add Jenkins and Ubuntu users to Docker group

```bash
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
```

### Step 3: Restart Docker and Jenkins

```bash
sudo systemctl restart docker
sudo systemctl restart jenkins
```

### Step 4: Verify Docker Access

```bash
sudo su - jenkins
docker run hello-world
```

---

## Understanding Jenkins Architecture

### Traditional Setup

- Jenkins **Master** schedules jobs.
- **Worker nodes** (often EC2 instances) execute jobs.
- Problems:
  - Idle workers waste resources
  - Manual scaling and dependency conflicts

### Modern Setup with Docker Agents

- Use **Docker containers as ephemeral agents**
- Lightweight, scalable, reproducible environments
- Auto-terminate after job completion

**Benefits**:
- Reduced cost and maintenance
- Avoid conflicting dependencies
- Easily switch between tech stacks

---

## Writing Jenkins Pipelines with Docker Agents

### Example: Simple Docker Agent Pipeline

**Jenkinsfile**
```groovy
pipeline {
  agent {
    docker {
      image 'node:16-alpine'
    }
  }
  stages {
    stage('Test') {
      steps {
        sh 'node -v'
      }
    }
  }
}
```

- This pipeline pulls the Node.js container and runs `node -v`.
- Once the job is complete, the container is automatically destroyed.

---

## Multi-Stage & Multi-Agent Pipelines

### Real-World Use Case: 3-Tier Architecture

**Jenkinsfile**
```groovy
pipeline {
  agent none
  stages {
    stage('Frontend') {
      agent {
        docker { image 'node:16-alpine' }
      }
      steps {
        sh 'npm install && npm run build'
      }
    }

    stage('Backend') {
      agent {
        docker { image 'maven:3.9.0-eclipse-temurin-17' }
      }
      steps {
        sh 'mvn clean install'
      }
    }

    stage('Database') {
      agent {
        docker { image 'mysql:8' }
      }
      steps {
        sh 'echo "Running DB migrations"'
      }
    }
  }
}
```

- Each stage uses a **dedicated Docker image** for specific application tiers.

---

# Real-World End-to-End CI/CD Pipeline Implementation using:

- Jenkins as the CI orchestrator
- Docker to containerize the app
- SonarQube for static code analysis
- GitHub for version control
- DockerHub for image registry
- ArgoCD for continuous deployment
- Kubernetes as the deployment platform

The application used in this tutorial is a **Python-based ToDo application**, sourced from the GitHub repository [Jenkins-Zero-To-Hero by Veeramalla](https://github.com/iam-veeramalla/Jenkins-Zero-To-Hero/tree/main/python-jenkins-argocd-k8s/todoApp).

---

## Project Overview

The pipeline implements the following steps:

- Jenkins checks out the todoApp source code from GitHub
- Runs static code analysis using SonarQube
- Builds and pushes Docker image to DockerHub
- Updates the Kubernetes deployment YAML with the new image version
- Pushes the updated manifest to GitHub
- ArgoCD detects the change and deploys the app to a Kubernetes cluster

---

## Infrastructure Setup

Provision a VM (e.g., EC2 Ubuntu instance) and ensure these ports are open:

| Port | Service       |
|------|---------------|
| 22   | SSH           |
| 8080 | Jenkins       |
| 9000 | SonarQube     |
| 80/443 | ArgoCD UI  |
| 30000+ | Kubernetes NodePort |

Install the following:

- Jenkins
- Docker
- kubectl & minikube or a managed K8s cluster (EKS, AKS, GKE)
- SonarQube
- ArgoCD

---

## Installing Jenkins & Docker

# Java and Jenkins

```bash
sudo apt update
sudo apt install openjdk-17-jre -y
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
sudo systemctl start jenkins
```

# Docker

```bash
sudo apt install docker.io -y
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
sudo systemctl restart docker
```

## Setting Up SonarQube

```bash
sudo adduser sonar
sudo su - sonar
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip sonarqube-9.4.0.54424.zip
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start
```

Access SonarQube at `http://<ip>:9000` and generate a token.

---

## Jenkins Pipeline Configuration

### Required Jenkins Plugins

- Docker Pipeline
- GitHub Integration
- SonarQube Scanner

### Jenkins Credentials

- GitHub Token (for manifest push)
- DockerHub Username & Password
- SonarQube Token

### Jenkinsfile

```groovy
pipeline {
    agent {
        docker {
            image 'python:3.8-slim'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    environment {
        SONARQUBE_URL = 'http://<JENKINS-IP>:9000'
        SONARQUBE_TOKEN = credentials('sonarqube-token')
        DOCKERHUB_CRED = credentials('dockerhub-creds')
        GITHUB_TOKEN = credentials('github-token')
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/iam-veeramalla/Jenkins-Zero-To-Hero.git', branch: 'main'
            }
        }

        stage('Code Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'sonar-scanner -Dsonar.projectKey=todoApp -Dsonar.sources=.'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t your-dockerhub/todoapp:${BUILD_NUMBER} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-creds', url: '']) {
                    sh 'docker push your-dockerhub/todoapp:${BUILD_NUMBER}'
                }
            }
        }

        stage('Update K8s Manifest') {
            steps {
                sh '''
                sed -i "s|image: .*|image: your-dockerhub/todoapp:${BUILD_NUMBER}|" python-jenkins-argocd-k8s/todoApp/k8s/deployment.yml
                git config user.email "dev@example.com"
                git config user.name "DevOpsBot"
                git add python-jenkins-argocd-k8s/todoApp/k8s/deployment.yml
                git commit -m "Update image to ${BUILD_NUMBER}"
                git push https://your-github-token@github.com/your-user/Jenkins-Zero-To-Hero.git
                '''
            }
        }
    }
}
```

---

## Setting Up ArgoCD

Install ArgoCD into your Kubernetes cluster:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Access ArgoCD:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Log in using:

```bash
# Get default admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### Create ArgoCD Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: todo-app
  namespace: argocd
spec:
  destination:
    namespace: default
    server: https://kubernetes.default.svc
  project: default
  source:
    path: python-jenkins-argocd-k8s/todoApp/k8s
    repoURL: https://github.com/iam-veeramalla/Jenkins-Zero-To-Hero
    targetRevision: main
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Apply with:

```bash
kubectl apply -f app.yaml
```

---

## CI/CD Flow in Action

1. Code is pushed to GitHub
2. Jenkins job is triggered (polling/webhook)
3. SonarQube runs code analysis
4. Docker image is built and pushed
5. Kubernetes manifest is updated with new image tag
6. Manifest is pushed to GitHub
7. ArgoCD detects the change and deploys the new image to the cluster

---

In this article, we implemented a **complete CI/CD pipeline using Jenkins, Docker, Kubernetes, and ArgoCD** to deploy a Python-based todoApp. This architecture is scalable, modern, and suitable for real-world DevOps workflows. Adopting this approach helps reduce operational overhead and aligns your delivery pipeline with best practices in the industry.

**Start building smarter, faster, and scalable pipelines with Jenkins and ArgoCD.**

## Best Practices

- Use versioned Docker tags instead of `latest`
- Use multistage Docker builds for Python apps
- Secure credentials using Jenkins credential store
- Use branch protection rules for production deployments
- Use ArgoCD's automated sync for true GitOps
- Use **Docker Agents** instead of static workers
- Always write **Declarative Pipelines** in code (Jenkinsfile)
- Use **`pipeline-syntax` generator** to explore Groovy DSL
- Separate logic into **multiple stages and steps**
- Store Jenkinsfiles in **Git for version control**
- **Avoid 'latest'** tag in Docker images; use versioned tags

---

## Jenkins Interview Questions

**1. Explain your CI/CD workflow.**  
Discuss SCM → Build → Test → Push → Deploy flow with Jenkins, Docker, ArgoCD.

**2. Why use Docker agents in Jenkins?**  
Lightweight, isolated, and reproducible build environments.

**3. Why choose ArgoCD over manual deployment or Ansible?**  
GitOps-based, declarative, self-healing, and continuous syncing with Git.

**4. How to handle environment-specific deployments?**  
Use Helm or Kustomize with ArgoCD to manage overlays.

---

## Conclusion

Jenkins continues to be the backbone of many CI/CD systems, but evolving DevOps architectures demand more flexibility, scalability, and maintainability. By leveraging **Docker containers as Jenkins agents** and deploying via **ArgoCD into Kubernetes**, you're embracing a cloud-native, modern CI/CD workflow.

**Start simple, scale smart.**
