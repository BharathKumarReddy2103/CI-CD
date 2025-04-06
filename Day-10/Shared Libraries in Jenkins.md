**Shared Libraries in Jenkins**

**1. What are Shared Libraries in Jenkins?**

Jenkins Shared Libraries are reusable Groovy scripts that can be used across multiple Jenkins pipelines. These libraries help in maintaining modularity, reducing redundancy, and standardizing CI/CD pipelines across different projects. Instead of duplicating code in each Jenkinsfile, you can extract common logic into a shared library and use it in multiple pipelines.

**2. Why Use Shared Libraries?**

•	**Code Reusability**: Avoid repetition of common pipeline logic.

•	**Maintainability**: Centralized updates instead of modifying multiple pipelines.

•	**Modularity**: Segregate pipeline functionalities into separate modules.

•	**Security**: Restrict execution of unapproved scripts by using approved libraries.

•	**Scalability**: Works well for large-scale CI/CD setups.

---

**3. Components of a Shared Library**

A Shared Library consists of the following main components:

**i. Directory Structure**

The library should be stored in a Git repository with a specific structure:

```bash
└── shared-library-repo/
    ├── vars/
    │   ├── deployApp.groovy
    │   ├── testApp.groovy
    │   ├── buildApp.groovy
    ├── src/
    │   └── com/example/Utils.groovy
    ├── resources/
    │   ├── config.yaml
    ├── test/
    │   ├── testScripts.groovy
    ├── README.md
```

•	vars/: Contains global pipeline functions (entry points).

•	src/: Contains helper Groovy classes (organized under packages).

•	resources/: Contains external configuration files.

•	test/: Contains test scripts for unit testing the library.

---

**ii. Writing a Shared Library Function**

Example of a Shared Library function inside vars/deployApp.groovy:

```bash
def call(String appName, String environment) {
    pipeline {
        agent any
        stages {
            stage('Deploy') {
                steps {
                    script {
                        echo "Deploying ${appName} to ${environment}"
                        sh "kubectl apply -f deployment.yaml"
                    }
                }
            }
        }
    }
}
```

This function can be invoked from a Jenkinsfile:

```bash
@Library('my-shared-library') _
deployApp('my-app', 'staging')
```

---

**iii. Writing a Utility Class**

If you want a helper function to be used across multiple library files, you can define it in src/com/example/Utils.groovy:
package com.example

```bash
class Utils {
    static void printMessage(String message) {
        println "INFO: ${message}"
    }
}
```

You can call this in vars/deployApp.groovy:

```bash
import com.example.Utils

def call(String appName, String environment) {
    pipeline {
        agent any
        stages {
            stage('Deploy') {
                steps {
                    script {
                        Utils.printMessage("Deploying ${appName} to ${environment}")
                        sh "kubectl apply -f deployment.yaml"
                    }
                }
            }
        }
    }
}
```

---

**4. Loading a Shared Library in Jenkins**

**i. Configuring Global Shared Libraries**

1.	Navigate to **Jenkins Dashboard → Manage Jenkins → Configure System**.

2.	Scroll to **Global Pipeline Libraries**.

3.	Click **Add** and enter:

o	**Name**: my-shared-library

o	**Source Code Management**: Select Git

o	**Repository URL**: https://github.com/myorg/shared-library.git

o	**Default Version**: main (or specify a tag/branch)

o	**Load Implicitly**: (Optional) Loads the library automatically in pipelines.

---

**ii. Loading Libraries in Jenkinsfile**

There are two ways to load the shared library in Jenkinsfile:

**1. Implicit Loading**

If you checked "**Load Implicitly**" in Jenkins settings, the library will be available automatically:

```bash
deployApp('my-app', 'production')
```

**2. Explicit Loading**

If "**Load Implicitly**" is unchecked, load it using @Library annotation:

```bash
@Library('my-shared-library') _
deployApp('my-app', 'production')
```

You can also specify a branch/tag:

```bash
@Library('my-shared-library@develop') _
deployApp('my-app', 'staging')
```

For multiple libraries:

```bash
@Library(['my-shared-library', 'another-library']) _
```

---

**5. Versioning and Tagging Shared Libraries**

To ensure stability and avoid breaking changes, you should version your shared library using Git tags or branches.

•	**Stable Versions**: Use Git tags (v1.0.0, v1.1.0)

•	**Development Versions**: Use branches (feature-x, dev, staging)

Example:

```bash
@Library('my-shared-library@v1.0.0') _
```

---

**6. Security in Shared Libraries**

•	**Approved Scripts**: Use **In-process Script Approval** to review new scripts before execution.

•	**Restrict Execution**: Set permissions so only certain teams can modify libraries.

•	**Use @NonCPS Annotation**: When using closures or certain Groovy constructs that may break serialization.

---

**7. Best Practices for Jenkins Shared Libraries**

1.	**Follow a Modular Approach**: Split functionality into different Groovy files.

2.	**Use src/ for Complex Logic**: Keep vars/ simple and delegate logic to src/.

3.	**Version Control**: Always use specific versions to avoid breaking changes.
   
4.	**Testing**: Implement unit tests for library functions.

5.	**Logging & Debugging**: Use meaningful logging (echo, println).

6.	**Access Control**: Restrict who can edit the shared library repository.

---

**8. Example: Full Jenkinsfile with Shared Library**

```bash
@Library('my-shared-library@v1.2') _
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                buildApp('my-app')
            }
        }
        stage('Test') {
            steps {
                testApp('my-app')
            }
        }
        stage('Deploy') {
            steps {
                deployApp('my-app', 'production')
            }
        }
    }
}
```

---

**9. Real-World Use Cases of Shared Libraries**

•	**Standardizing CI/CD Pipelines**: Common pipeline logic for all applications.

•	**Reusable Deployment Scripts**: Kubernetes, Docker, AWS CloudFormation, etc.

•	**Security & Compliance Checks**: Automate security scans across projects.

•	**Multi-cloud Deployment**: Centralized functions to handle AWS, Azure, GCP.

•	**Integration with Monitoring Tools**: Standardized logging and alerting mechanisms.

---

**Conclusion**

Jenkins Shared Libraries are a powerful way to modularize and reuse pipeline code. By centralizing logic into a single repository, teams can improve maintainability, enhance security, and streamline CI/CD workflows. Proper versioning, testing, and security measures ensure a robust pipeline setup across multiple projects.
