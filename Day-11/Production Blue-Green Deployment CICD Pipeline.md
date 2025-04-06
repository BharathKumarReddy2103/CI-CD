**Blue-Green Deployment Strategy with Kubernetes & Jenkins**

**Introduction**

Blue-Green Deployment is a zero-downtime deployment strategy that enables seamless application upgrades by maintaining two identical environments: **Blue (current production)** and **Green (new version)**. This approach minimizes risk by allowing traffic switching between environments, ensuring continuous service availability.

This project demonstrates an end-to-end **Blue-Green Deployment** implementation using:

•	**AWS EKS** for Kubernetes cluster management

•	**Jenkins** for CI/CD automation

•	**Docker** for containerization

•	**SonarQube** for code quality analysis

•	**Nexus Repository** for artifact storage

•	**Terraform** for infrastructure provisioning

---

**Key Concepts**

**1. Blue-Green Deployment Overview**

•	**Blue Environment**: Represents the currently live application version.

•	**Green Environment**: Holds the new version for testing before traffic is switched.

•	**Traffic Switching**: Load balancer dynamically directs requests to either Blue or Green.

•	**Rollback Mechanism**: In case of failures, traffic is quickly reverted to the Blue environment.

**2. Infrastructure Setup**

•	**AWS EKS** cluster provisioned with Terraform.

•	**EC2 Instances** for Jenkins, SonarQube, and Nexus setup.

•	**Security Groups & IAM Roles** configured for secure access.

**3. CI/CD Pipeline Stages**

•	**Code Checkout**: Retrieves the latest source code from GitHub.

•	**Static Code Analysis**: Uses SonarQube to analyze code quality.

•	**Build & Test**: Compiles the application and runs test cases.

•	**Docker Image Creation**: Builds and pushes Docker images to Docker Hub.

•	**Kubernetes Deployment**: Deploys the application to the Blue or Green environment.

•	**Traffic Switching**: Updates the service selector to switch between versions.

---

**Implementation**

**1. Infrastructure Provisioning**

Use Terraform to provision the EKS cluster:

```sh
terraform init
terraform plan
terraform apply -auto-approve
```

This creates:

•	**VPC, Subnets, and Security Groups** for networking.

•	**EKS Cluster** with node groups for application deployment.

•	**IAM Roles and Policies** for Kubernetes access.

---

**2. Setting Up Jenkins, SonarQube, and Nexus**

**Jenkins Installation**

Deploy Jenkins on an EC2 instance:

```sh
sudo apt update
sudo apt install openjdk-17-jdk -y
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

**Install required plugins:**

•	**Pipeline, SonarQube Scanner, Docker, Kubernetes, and Nexus Artifact Uploader**.

**SonarQube Setup**

Run SonarQube in a Docker container:

```sh
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

Generate a **SonarQube Token** for Jenkins authentication.

**Nexus Setup**

Run Nexus in a Docker container:

```sh
docker run -d --name nexus -p 8081:8081 sonatype/nexus3
```

Configure repositories:

•	**Maven Releases** for stable artifacts.

•	**Maven Snapshots** for development versions.

---

**3. Configuring Jenkins CI/CD Pipeline**

**Jenkinsfile**

```sh
pipeline {
    agent any
    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['blue', 'green'], description: 'Deployment environment')
        string(name: 'DOCKER_TAG', defaultValue: 'latest', description: 'Docker image tag')
        booleanParam(name: 'SWITCH_TRAFFIC', defaultValue: false, description: 'Toggle traffic switch')
    }
    environment {
        IMAGE_NAME = 'your-dockerhub-username/app-image'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', credentialsId: 'git-credentials', url: 'https://github.com/your-repo.git'
            }
        }
        stage('Static Code Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Build & Push Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$DOCKER_TAG .'
                sh 'docker push $IMAGE_NAME:$DOCKER_TAG'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deployFile = params.DEPLOY_ENV == 'blue' ? 'blue-deployment.yaml' : 'green-deployment.yaml'
                    sh "kubectl apply -f k8s/$deployFile"
                }
            }
        }
        stage('Switch Traffic') {
            when {
                expression { params.SWITCH_TRAFFIC }
            }
            steps {
                script {
                    sh "kubectl patch svc app-service -p '{\"spec\":{\"selector\":{\"version\":\"${params.DEPLOY_ENV}\"}}}'"
                }
            }
        }
    }
}
```

---

**4. Kubernetes Deployment**

**MySQL Database Deployment**

```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "rootpass"
```

**Application Deployment (Blue & Green)**

**Blue Deployment**

```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-blue
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: myapp
        image: your-dockerhub-username/app-image:blue
```

**Green Deployment**

```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-green
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: myapp
        image: your-dockerhub-username/app-image:green
```

**Load Balancer Service**

```sh
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

---

**5. Traffic Switching**

Switch traffic to **Green**:

```sh
kubectl patch svc app-service -p '{"spec":{"selector":{"version":"green"}}}'
```

Rollback to **Blue**:

```sh
kubectl patch svc app-service -p '{"spec":{"selector":{"version":"blue"}}}'
```

---

**Conclusion**

This implementation of **Blue-Green Deployment using Kubernetes, Jenkins, AWS EKS, and Docker** provides:

•	**Zero downtime application deployment**.

•	**Seamless rollback and version switching**.

•	**Automated CI/CD pipeline for efficient DevOps workflows**.

---

**Future Enhancements**

✅ Implement **Canary Deployment**

✅ Integrate **Helm Charts** for Kubernetes

✅ Enhance monitoring with **Prometheus & Grafana**

---

**Contributors**

If you found this project useful, give it a ⭐️ **and contribute**.
