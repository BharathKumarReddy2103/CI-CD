# Building the Ultimate CI/CD Pipeline Using Jenkins and Argo CD (GitOps Approach) Explanation

## Introduction

In the world of modern DevOps practices, implementing a robust and scalable CI/CD pipeline is critical. This article walks you through a real-time, end-to-end CI/CD pipeline that leverages Jenkins, Maven, SonarQube, Docker, Argo CD, and GitOps principles. Designed for DevOps and Cloud professionals, this setup ensures automated code integration, rigorous testing, and seamless deployment to Kubernetes clusters.

If you're preparing for DevOps interviews or aiming to build a professional portfolio, this real-world implementation will serve as a strong foundation.

---

## Table of Contents

1. [Understanding CI/CD](#understanding-cicd)
2. [Tools Used](#tools-used)
3. [Project Architecture Overview](#project-architecture-overview)
4. [Continuous Integration (CI) Workflow](#continuous-integration-ci-workflow)
5. [Continuous Delivery (CD) Workflow](#continuous-delivery-cd-workflow)
6. [GitOps Workflow with Argo CD](#gitops-workflow-with-argo-cd)
7. [Interview Preparation Tips](#interview-preparation-tips)
8. [Best Practices](#best-practices)
9. [Conclusion](#conclusion)

---

## Understanding CI/CD

- **Continuous Integration (CI)**: Automates the integration of code changes into a shared repository. It ensures that the code builds, tests pass, and Docker images are created.
- **Continuous Delivery (CD)**: Automates the release of code to production or staging environments. This article follows a GitOps-based CD model using Argo CD.

---

## Tools Used

- **GitHub**: Source code and application manifest repository
- **Jenkins**: CI pipeline orchestration
- **Maven**: Java build tool
- **SonarQube**: Static code analysis
- **Docker**: Containerization and image creation
- **Docker Hub / ECR / Quay.io**: Container registries
- **Argo CD**: GitOps-based continuous delivery tool
- **Argo Image Updater**: Updates Git repository with new Docker image tags
- **Kubernetes**: Deployment platform for containerized applications

---

## Project Architecture Overview

```plaintext
Developer → GitHub (App Repo)
          ↓
     Webhook → Jenkins CI Pipeline
                ↓ Build + Test + Scan + Docker Build + Push
                → Container Registry (ECR/DockerHub)
                          ↓
                Argo Image Updater → GitHub (Manifest Repo)
                          ↓
                        Argo CD → Kubernetes Cluster
```

---

## Continuous Integration (CI) Workflow

### 1. Developer Workflow

- Developer raises a Pull Request (PR) in GitHub.
- Webhook triggers Jenkins pipeline upon PR creation.

### 2. Jenkins Pipeline Stages

```groovy
pipeline {
  agent {
    docker {
      image 'maven:3.9.1-jdk-17'
    }
  }
  stages {
    stage('Build') {
      steps {
        sh 'mvn clean install'
      }
    }
    stage('Unit Tests') {
      steps {
        sh 'mvn test'
      }
    }
    stage('Code Analysis') {
      steps {
        sh 'mvn sonar:sonar'
      }
    }
    stage('Docker Build & Push') {
      steps {
        sh '''
          docker build -t example-app:v1.0.1 .
          docker push example-app:v1.0.1
        '''
      }
    }
  }
}
```

### 3. Notifications

- Jenkins can be configured to send:
  - Email alerts (via Email plugin)
  - Slack alerts (via Webhooks/API)
  - AWS SNS notifications

---

## Continuous Delivery (CD) Workflow

### Traditional Approach

- CD tasks (like deploying to Kubernetes) were often included in Jenkins pipeline via shell scripts or Ansible.
- Issues:
  - Not scalable
  - Poor visibility and auditing

### GitOps Approach (Recommended)

- Separation of concerns:
  - **CI** handles build, test, and image creation.
  - **CD** handles deployment via declarative GitOps model.

---

## GitOps Workflow with Argo CD

### Why Two Git Repositories?

1. **Source Code Repository** (App Code + Jenkinsfile + Dockerfile)
2. **Manifest Repository** (Helm Charts / YAMLs for Kubernetes)

### How it Works:

1. **CI Completes**: Jenkins pushes Docker image to registry.
2. **Argo Image Updater**:
   - Detects new image tag.
   - Updates `deployment.yaml` or `values.yaml` in manifest repo.
3. **Git Commit Triggers Argo CD**:
   - Argo CD detects new commit.
   - Syncs updated manifest to Kubernetes.

### Argo CD Features:

- Reverts unauthorized cluster changes.
- Maintains desired state declared in Git.
- Supports Helm, Kustomize, plain YAML, etc.

---

## Interview Preparation Tips

Be prepared to answer the following:

- **What CI/CD tools have you used?**
  - Mention Jenkins for CI and Argo CD for CD.
- **How is Jenkins connected to GitHub?**
  - Via Webhook for PR and push events.
- **What type of Jenkins pipeline did you use?**
  - Use Declarative pipelines for maintainability.
- **What agent is used in Jenkins?**
  - Docker agents to avoid tool installations.
- **How is the CD process triggered?**
  - Through Argo Image Updater and GitOps via Argo CD.
- **Why use separate repos for code and manifests?**
  - Enables clean separation of source and deployment logic, better auditing, and scalability.

---

## Best Practices

- Use **Docker agents** in Jenkins for lightweight, tool-preloaded environments.
- Choose **Declarative Jenkins pipelines** for clarity and collaboration.
- Ensure **code quality gates** with SonarQube or similar tools.
- Automate image promotion with **Argo Image Updater**.
- Maintain desired state using **Argo CD** GitOps pattern.
- Keep **separate Git repos** for source code and deployment manifests.

---

This CI/CD pipeline exemplifies modern DevOps best practices by combining automation, scalability, and security. By leveraging Jenkins for CI and Argo CD for GitOps-based CD, you create a clear separation between integration and delivery while maintaining control over the software lifecycle.

This setup not only prepares you for real-world scenarios but also helps you stand out in DevOps interviews and contribute meaningfully to open-source projects.

# Ultimate End-to-End CI/CD Pipeline Implementation with Jenkins, Docker, SonarQube, Shell Script, and Argo CD on Kubernetes

In this comprehensive guide, you'll learn how to implement a complete end-to-end CI/CD pipeline using industry-standard DevOps tools such as Jenkins, Maven, Docker, SonarQube, Shell Scripting, and Argo CD on a Kubernetes cluster. This tutorial walks through everything from source code to deployment using GitOps principles, making it an excellent real-world example for DevOps engineers, open-source contributors, and those preparing for interviews.

Whether you're deploying a Java Spring Boot application or working with CI/CD best practices, this project will help you understand tool integration, configuration, automation, and troubleshooting from scratch.

---

## GitHub Repository

All the source code, manifests, and configurations used in this CI/CD pipeline are available in the following repository:

**GitHub Repository**: [Jenkins-Zero-To-Hero - Java Maven Sonar Argo CD](https://github.com/iam-veeramalla/Jenkins-Zero-To-Hero/tree/main/java-maven-sonar-argocd-helm-k8s/spring-boot-app)

> Clone the repository and follow along with the setup:
```bash
git clone https://github.com/iam-veeramalla/Jenkins-Zero-To-Hero.git
cd Jenkins-Zero-To-Hero/java-maven-sonar-argocd-helm-k8s/spring-boot-app
```

## Architecture Overview

The CI/CD pipeline covers the following key stages:

1. **Java Application Build with Maven**
2. **Static Code Analysis using SonarQube**
3. **Docker Image Build & Push**
4. **Manifest Update with Shell Script**
5. **Deployment with Argo CD on Kubernetes**

This approach follows the GitOps model for Continuous Delivery, where Argo CD syncs Kubernetes manifests stored in Git repositories.

---

## Tools Used

- **Jenkins**: CI server to orchestrate the pipeline
- **Maven**: To build and test the Java application
- **SonarQube**: For static code analysis
- **Docker**: For containerizing the application
- **Shell Script**: To update the image tag in manifests
- **Argo CD**: For GitOps-based Kubernetes deployment
- **Kubernetes**: Application hosting environment (Minikube used for demo)
- **AWS EC2**: Hosting Jenkins and SonarQube

---

## CI/CD Pipeline Flow

```plaintext
Source Code (GitHub)
      |
      v
Jenkins CI Pipeline:
    - Maven Build
    - SonarQube Analysis
    - Docker Build & Push
    - Update K8s Manifest
      |
      v
Argo CD watches the Git Repo
      |
      v
Kubernetes Cluster Deployment
```
---

## Application Overview

We use a simple **Java Spring Boot MVC application** located at:
**`java-maven-sonar-argocd-helm-k8s/spring-boot-app/`**

- Displays text on a web page (`index.html`)
- Is packaged using Maven
- Is containerized using Docker
- Is deployed to Kubernetes via Argo CD

> The `README.md` in the above folder explains how to run the application locally and sets the stage for the CI/CD pipeline.

---

## Jenkins Pipeline Implementation

### Jenkinsfile Structure

```groovy
pipeline {
  agent {
    docker {
      image 'nbkumar2103/maven-docker-agent:latest'
    }
  }
  environment {
    DOCKER_CREDS = credentials('docker-creds')
    GITHUB_TOKEN = credentials('github-token')
    SONAR_TOKEN = credentials('sonar-token')
  }
  stages {
    stage('Build') {
      steps {
        sh '''
        cd java-springboot-app
        mvn clean package
        '''
      }
    }

    stage('Static Code Analysis') {
      steps {
        sh '''
        cd java-springboot-app
        mvn sonar:sonar \
          -Dsonar.projectKey=springboot-app \
          -Dsonar.host.url=http://<SONAR_HOST>:9000 \
          -Dsonar.login=$SONAR_TOKEN
        '''
      }
    }

    stage('Docker Build & Push') {
      steps {
        sh '''
        docker build -t nbkumar2103/ultimate-cicd:${BUILD_NUMBER} java-springboot-app
        echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin
        docker push nbkumar2103/ultimate-cicd:${BUILD_NUMBER}
        '''
      }
    }

    stage('Update Manifest') {
      steps {
        sh '''
        git clone https://github.com/nbkumar2103/jenkins-zero-to-hero.git
        cd jenkins-zero-to-hero/java-springboot-app/manifests
        sed -i "s|replace-image-tag|${BUILD_NUMBER}|" deployment.yaml
        git config --global user.email "ci@demo.com"
        git config --global user.name "CI Bot"
        git add deployment.yaml
        git commit -m "CI: Update image tag to ${BUILD_NUMBER}"
        git push https://$GITHUB_TOKEN@github.com/nbkumar2103/jenkins-zero-to-hero.git
        '''
      }
    }
  }
}
```

---

## SonarQube Setup

- Create a SonarQube EC2 instance with at least 8GB RAM
- Install Java, unzip Sonar binaries, and start the server
- Access via `http://<EC2_PUBLIC_IP>:9000` (default login: `admin/admin`)
- Generate a Sonar token under **My Account > Security**

---

## Docker Agent for Jenkins

**Why use Docker as a Jenkins agent?**

- Saves EC2 cost and resources
- Automatically cleans up containers after builds
- Requires no permanent slave nodes
- Portable and repeatable build environments

Image used: `nbkumar2103/maven-docker-agent:latest` (includes Maven + Docker)

---

## Argo CD Setup

### Step 1: Install Minikube

```bash
brew install minikube
minikube start --driver=hyperkit
```

### Step 2: Install OLM (Operator Lifecycle Manager)

```bash
kubectl apply -f https://operatorhub.io/install.yaml
```

### Step 3: Install Argo CD Operator

```bash
kubectl create -f https://operatorhub.io/install/argocd.yaml
```

### Step 4: Deploy Argo CD Controller

```yaml
# argocd.yaml
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: argocd
  namespace: default
spec: {}
```

```bash
kubectl apply -f argocd.yaml
```

### Step 5: Access Argo CD UI

```bash
kubectl edit svc argocd-server -n default
# Change type: ClusterIP to NodePort

minikube service argocd-server -n default
# Open the URL in your browser
```

### Step 6: Login to Argo CD

- Username: `admin`
- Password: Run:
```bash
kubectl get secret argocd-cluster -o jsonpath="{.data.admin\.password}" | base64 --decode
```

---

## Kubernetes Manifest Structure

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springboot-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: springboot-app
  template:
    metadata:
      labels:
        app: springboot-app
    spec:
      containers:
        - name: springboot
          image: nbkumar2103/ultimate-cicd:replace-image-tag
          ports:
            - containerPort: 8080
```

After the CI job completes, the `replace-image-tag` will be updated to the correct build number via shell script.

---

## GitOps Deployment with Argo CD

Argo CD continuously watches the Git repository. Once it detects the updated manifest, it:

- Pulls the new YAML
- Applies it to the Kubernetes cluster
- Auto-heals if any configuration drift is detected
- Provides a visual UI for rollout and status

---

## Best Practices

- Use **Docker agents** for Jenkins to save costs and improve performance
- Store **Jenkinsfiles** within your application repo for collaboration
- Secure all secrets using **Jenkins Credentials Store**
- Use **GitOps** and **Argo CD** for safer, audit-friendly CD
- Configure proper **RBAC** and **namespace isolation** in Kubernetes
- Keep manifest files version-controlled in Git

---

## Real-World Use Cases

- Automating Spring Boot application deployment
- Multi-stage CI pipelines using Docker
- Integrating Jenkins with SonarQube for quality gates
- Using shell scripts for custom GitOps-style updates
- Full GitOps delivery with Argo CD

---

## Conclusion

This end-to-end pipeline gives you a production-grade CI/CD workflow using Jenkins and GitOps. You’ve now seen how to:

- Build a Java app with Maven
- Perform code analysis with SonarQube
- Containerize and push Docker images
- Use shell scripts for manifest updates
- Automate Kubernetes deployment with Argo CD

This project can serve as a strong portfolio piece for your GitHub and LinkedIn. Feel free to fork the repository, run the pipeline yourself, and extend it further.
