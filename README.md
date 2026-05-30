# DevOps Semester 6 - Study Notes & Resources

This repository contains notes, examples, and configurations for the **DevOps** course (Semester 6, Session 2025-26).

## 📚 Topics Covered

### Unit IV - Maven Build Automation
- Build lifecycle phases (validate, compile, test, package, verify, install, deploy)
- Project Object Model (POM) and directory structure
- Dependency management, scopes, and transitive dependencies
- Maven plugins (Compiler, Surefire, Shade)
- Maven wrapper (`mvnw`)
- Maven and Docker integration

### Unit V - Continuous Integration with GitHub Actions
- Workflow automation and YAML syntax
- Events, triggers, jobs, steps, and runners
- Matrix strategies and caching
- Docker image builds in CI
- Deploying to Docker Hub and GHCR

### Unit VI - CI/CD with Jenkins
- Jenkins architecture (Master/Agent model)
- Declarative vs Scripted pipelines
- Jenkinsfile structure and pipeline stages
- Docker and Jenkins integration
- Jenkins and Maven integration
- CI/CD deployment flows

## 📂 Repository Structure

```
├── README.md
├── maven/
│   ├── maven-build-lifecycle.md
│   ├── maven-pom-and-dependencies.md
│   ├── maven-plugins.md
│   └── sample-pom.xml
├── github-actions/
│   ├── github-actions-fundamentals.md
│   ├── github-actions-docker.md
│   └── workflows/
│       └── sample-ci.yml
├── jenkins/
│   ├── jenkins-architecture.md
│   ├── jenkins-pipelines.md
│   └── Jenkinsfile
├── learning.txt
└── maven.txt
```

## 🎯 Course Outcomes
- **CO4**: Analyze and implement automated builds using Maven
- **CO5**: Apply CI workflows using GitHub Actions
- **CO6**: Develop end-to-end CI/CD pipelines using Jenkins
