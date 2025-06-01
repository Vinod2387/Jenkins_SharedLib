
## 🧾 Jenkins Shared Library Cheat Sheet

### 📁 Shared Library Structure

```
(root)
├── vars/
│   ├── code_checkout.groovy
│   ├── docker_build.groovy
│   ├── docker_push.groovy
│   ├── owasp_dependency.groovy
│   ├── sonarqube_analysis.groovy
│   ├── sonarqube_code_quality.groovy
│   ├── trivy_scan.groovy
│   └── docker_compose.groovy
├── src/                           # Optional - for custom classes
├── resources/                     # Optional - for templated files
```

### 🧪 Usage in Jenkinsfile

```groovy
@Library('your-shared-library') _

pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                script {
                    code_checkout("<GitRepo>", "<Branch>")
                }
            }
        }
    }
}
```

---

## 🔁 Common Function Templates in `vars/` and Their Pipeline Usage

### 1. `vars/code_checkout.groovy`

```groovy
// vars/code_checkout.groovy

def call(String GitUrl, String GitBranch) {
    git url: "${GitUrl}", branch: "${GitBranch}"
}
```

**Pipeline Usage:**
```groovy
stage('Git: Code Checkout') {
    steps {
        script {
            code_checkout("https://github.com/example/repo.git", "main")
        }
    }
}
```

### 2. `vars/docker_build.groovy`

```groovy
// vars/docker_build.groovy

def call(String ProjectName, String ImageTag, String DockerHubUser) {
    sh "docker build -t ${DockerHubUser}/${ProjectName}:${ImageTag} ."
}
```

**Pipeline Usage:**
```groovy
stage("Docker: Build Images") {
    steps {
        script {
            dir('backend') {
                docker_build("my-backend", "${params.BACKEND_DOCKER_TAG}", "mydockeruser")
            }
            dir('frontend') {
                docker_build("my-frontend", "${params.FRONTEND_DOCKER_TAG}", "mydockeruser")
            }
        }
    }
}
```

### 3. `vars/docker_push.groovy`

```groovy
// vars/docker_push.groovy

def call(String Project, String ImageTag, String dockerhubuser) {
    withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'dockerhubpass', usernameVariable: 'dockerhubuser')]) {
        sh "docker login -u ${dockerhubuser} -p ${dockerhubpass}"
    }
    sh "docker push ${dockerhubuser}/${Project}:${ImageTag}"
}
```

**Pipeline Usage:**
```groovy
stage("Docker: Push to DockerHub") {
    steps {
        script {
            docker_push("my-backend", "${params.BACKEND_DOCKER_TAG}", "mydockeruser")
            docker_push("my-frontend", "${params.FRONTEND_DOCKER_TAG}", "mydockeruser")
        }
    }
}
```

### 4. `vars/owasp_dependency.groovy`

```groovy
// vars/owasp_dependency.groovy

def call() {
    dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'OWASP'
    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
}
```

**Pipeline Usage:**
```groovy
stage("OWASP: Dependency check") {
    steps {
        script {
            owasp_dependency()
        }
    }
}
```

### 5. `vars/sonarqube_analysis.groovy`

```groovy
// vars/sonarqube_analysis.groovy

def call(String SonarQubeAPI, String Projectname, String ProjectKey) {
    withSonarQubeEnv("${SonarQubeAPI}") {
        sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=${Projectname} -Dsonar.projectKey=${ProjectKey} -X"
    }
}
```

**Pipeline Usage:**
```groovy
stage("SonarQube: Code Analysis") {
    steps {
        script {
            sonarqube_analysis("Sonar", "myproject", "myproject")
        }
    }
}
```

### 6. `vars/sonarqube_code_quality.groovy`

```groovy
// vars/sonarqube_code_quality.groovy

def call() {
    timeout(time: 1, unit: "MINUTES") {
        waitForQualityGate abortPipeline: false
    }
}
```

**Pipeline Usage:**
```groovy
stage("SonarQube: Code Quality Gate") {
    steps {
        script {
            sonarqube_code_quality()
        }
    }
}
```

### 7. `vars/trivy_scan.groovy`

```groovy
// vars/trivy_scan.groovy

def call() {
    sh "trivy fs ."
}
```

**Pipeline Usage:**
```groovy
stage("Trivy: Filesystem scan") {
    steps {
        script {
            trivy_scan()
        }
    }
}
```

### 8. `vars/docker_compose.groovy`

```groovy
// vars/docker_compose.groovy

def call() {
    sh "docker-compose down && docker-compose up -d"
}
```

**Pipeline Usage:**
```groovy
stage("Docker Compose: Deploy") {
    steps {
        script {
            docker_compose()
        }
    }
}
```

---

## 💡 Tips for Remembering

- Start your own function name map (`code_checkout`, `docker_push`, etc.)
- Keep one task per Groovy file (single `call()` function)
- Match function names to stage names in `Jenkinsfile`
- Use `script {}` block when calling shared lib methods in pipeline
- Use descriptive stage names to match library functions
