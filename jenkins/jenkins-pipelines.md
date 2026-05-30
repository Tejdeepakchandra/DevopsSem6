# Jenkins Pipelines

## Declarative vs Scripted Pipelines

Jenkins supports two pipeline syntaxes:

### Declarative Pipeline (Recommended)
- Structured, opinionated syntax
- Easier to read and write
- Built-in error handling with `post` blocks

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
    }
}
```

### Scripted Pipeline
- Full Groovy programming language
- More flexible but complex
- Used for advanced use cases

```groovy
node {
    stage('Build') {
        echo 'Building...'
    }
}
```

---

## Jenkinsfile Structure

A **Jenkinsfile** defines the entire pipeline as code, stored in the project repository.

```groovy
pipeline {
    // ===== Agent: Where to run =====
    agent any                    // Run on any available agent
    // agent { label 'linux' }  // Run on agent with label 'linux'
    // agent { docker { image 'maven:3.9' } }  // Run inside Docker

    // ===== Environment Variables =====
    environment {
        APP_NAME = 'my-application'
        VERSION  = '1.0.0'
        REGISTRY = 'docker.io/myuser'
    }

    // ===== Parameters =====
    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch to build')
        choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Target environment')
        booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Skip tests?')
    }

    // ===== Build Stages =====
    stages {
        stage('Checkout') {
            steps {
                git branch: "${params.BRANCH}",
                    url: 'https://github.com/user/repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile -B'
            }
        }

        stage('Test') {
            when {
                expression { !params.SKIP_TESTS }
            }
            steps {
                sh 'mvn test -B'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests -B'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Docker Build & Push') {
            when {
                branch 'main'    // Only on main branch
            }
            steps {
                script {
                    def image = docker.build("${REGISTRY}/${APP_NAME}:${VERSION}")
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-credentials') {
                        image.push()
                        image.push('latest')
                    }
                }
            }
        }

        stage('Deploy') {
            when {
                allOf {
                    branch 'main'
                    expression { params.ENV == 'prod' }
                }
            }
            steps {
                echo "Deploying to ${params.ENV}..."
                // sshagent(['deploy-key']) {
                //     sh "ssh user@server 'docker-compose pull && docker-compose up -d'"
                // }
            }
        }
    }

    // ===== Post Actions =====
    post {
        success {
            echo 'Pipeline succeeded!'
            // slackSend(message: "Build #${BUILD_NUMBER} succeeded!")
        }
        failure {
            echo 'Pipeline failed!'
            // mail to: 'team@example.com', subject: "Build Failed: ${JOB_NAME}"
        }
        always {
            cleanWs()    // Clean workspace after build
        }
    }
}
```

---

## Pipeline Stages Explained

| Stage | Purpose | Maven Command |
|-------|---------|---------------|
| Checkout | Pull source code from Git | `git clone/checkout` |
| Build | Compile the source code | `mvn compile` |
| Test | Run unit/integration tests | `mvn test` |
| Package | Create JAR/WAR artifact | `mvn package` |
| Post actions | Cleanup, notifications | - |

---

## Multi-Branch Pipelines

Multi-branch pipelines **automatically** create jobs for each branch that has a Jenkinsfile.

### Setup:
1. New Item → Multibranch Pipeline
2. Configure Git source
3. Jenkins scans branches and creates pipelines

### Benefits:
- Each branch gets its own pipeline
- PRs can have their own build/test pipeline
- Branches without Jenkinsfile are ignored
- Old branches can be auto-cleaned

---

## Jenkins and Maven Integration

### Global Tool Configuration:
```
Manage Jenkins → Tools
├── Maven installations
│   └── Name: Maven-3.9
│       Install from Apache (version 3.9.4)
└── JDK installations
    └── Name: JDK-17
        Install from adoptium.net
```

### Using Maven in Pipeline:
```groovy
pipeline {
    agent any
    tools {
        maven 'Maven-3.9'     // Name from Global Tool Config
        jdk 'JDK-17'
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean install -B'
            }
        }
    }
}
```

### Code Coverage & Test Reports:
```groovy
stage('Test') {
    steps {
        sh 'mvn test -B'
    }
    post {
        always {
            // JUnit test reports
            junit '**/target/surefire-reports/*.xml'
            
            // Code coverage with JaCoCo
            jacoco(
                execPattern: '**/target/*.exec',
                classPattern: '**/target/classes',
                sourcePattern: '**/src/main/java'
            )
        }
    }
}
```

---

## Jenkins CI/CD Deployment Flows

### Triggering Builds:

```groovy
pipeline {
    triggers {
        // Poll SCM every 5 minutes
        pollSCM('H/5 * * * *')
        
        // Or use GitHub Webhook (preferred)
        // Configure in GitHub: Settings → Webhooks → Add webhook
        // URL: http://jenkins-url/github-webhook/
    }
}
```

### Jenkins Agents Types:
| Type | Description | Use Case |
|------|-------------|----------|
| SSH | Connect via SSH | Linux/Mac servers |
| JNLP | Java agent connects to master | Behind firewalls |
| Docker | Containers as agents | Isolated, reproducible builds |
| Kubernetes | Pods as agents | Cloud-native, scalable |

---

## Docker and Jenkins Integration

### Building Docker Images in Jenkins:
```groovy
stage('Docker Build') {
    steps {
        script {
            def app = docker.build("myapp:${BUILD_NUMBER}")
        }
    }
}
```

### Docker Inside Jenkins Agents:
```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.9-eclipse-temurin-17'
            args '-v $HOME/.m2:/root/.m2'    // Cache Maven deps
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}
```

### Publishing to Docker Hub/GHCR:
```groovy
stage('Push to Registry') {
    steps {
        script {
            docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-creds') {
                def image = docker.build("user/app:${BUILD_NUMBER}")
                image.push()
                image.push('latest')
            }
        }
    }
}
```

---

## Pipeline Best Practices

1. **Keep Jenkinsfile in the repo** – pipeline as code
2. **Use declarative syntax** – easier to maintain
3. **Fail fast** – run quick checks (lint, compile) before long tests
4. **Use shared libraries** – avoid duplicating pipeline code
5. **Clean workspace** – use `cleanWs()` in post actions
6. **Use credentials** – never hardcode secrets
7. **Parallelize** – run independent stages in parallel
8. **Cache dependencies** – mount Maven/npm caches as volumes

---

## Key Takeaways

1. **Declarative pipelines** are the recommended approach
2. **Jenkinsfile** lives in the repo – version controlled, reviewable
3. **Multi-branch pipelines** auto-create jobs per branch
4. Use **Global Tool Configuration** for Maven/JDK setup
5. **Webhooks** are preferred over pollSCM for build triggers
6. **Docker agents** provide clean, isolated build environments
