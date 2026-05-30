# GitHub Actions - Fundamentals

## Understanding Workflow Automation

GitHub Actions is a **CI/CD platform** built directly into GitHub. It allows you to automate build, test, and deployment workflows triggered by events in your repository.

### Why GitHub Actions?
- No external CI/CD server needed (unlike Jenkins)
- Native GitHub integration
- Free for public repositories
- Marketplace with thousands of pre-built actions

---

## Workflow Directory Structure

All workflows live in the `.github/workflows/` directory:

```
.github/
└── workflows/
    ├── ci.yml           # Continuous Integration
    ├── cd.yml           # Continuous Deployment
    └── release.yml      # Release workflow
```

---

## Key Components

### 1. Workflows
A **workflow** is an automated process defined in a YAML file. It consists of one or more **jobs**.

```yaml
name: CI Pipeline          # Workflow name
on: push                   # Trigger event
jobs:                      # Jobs to execute
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Hello CI!"
```

### 2. Jobs
A **job** is a set of steps that run on the same runner. Jobs run in **parallel** by default.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest    # Runner type
    steps: [...]
  
  test:
    runs-on: ubuntu-latest
    needs: build              # Run after 'build' job
    steps: [...]
```

### 3. Steps
**Steps** are individual tasks within a job. Each step either runs a shell command or uses an action.

```yaml
steps:
  - name: Checkout code
    uses: actions/checkout@v4      # Use a marketplace action
  
  - name: Run tests
    run: mvn test                  # Shell command
  
  - name: Upload artifact
    uses: actions/upload-artifact@v4
    with:
      name: build-output
      path: target/*.jar
```

### 4. Actions
**Actions** are reusable units of code. They can be:
- **Official**: `actions/checkout`, `actions/setup-java`
- **Community**: From GitHub Marketplace
- **Custom**: Your own actions

### 5. Runners
**Runners** are servers that execute your workflows.

| Type | Description | Examples |
|------|-------------|----------|
| GitHub-hosted | Managed by GitHub, fresh VM each run | `ubuntu-latest`, `windows-latest`, `macos-latest` |
| Self-hosted | Your own servers, you manage them | On-prem servers, cloud VMs |

**Self-hosted runner setup:**
```bash
# Download and configure
./config.sh --url https://github.com/USER/REPO --token TOKEN
./run.sh
```

**Runner security considerations:**
- Never use self-hosted runners for public repos (anyone can run code)
- Use labels to control which jobs run on which runners
- Keep runner software updated

---

## Workflow Triggers

### Push & Pull Request
```yaml
on:
  push:
    branches: [main, develop]
    paths:
      - 'src/**'               # Only trigger when source code changes
  pull_request:
    branches: [main]
```

### Schedule (Cron)
```yaml
on:
  schedule:
    - cron: '0 2 * * 1'       # Every Monday at 2 AM UTC
```

### Manual Trigger (workflow_dispatch)
```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deploy to which environment?'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
```

---

## Matrix Strategies

Run the same job with **multiple configurations** (e.g., different Java versions, OS):

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        java-version: [11, 17, 21]
      fail-fast: false          # Don't cancel other jobs if one fails
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: ${{ matrix.java-version }}
      - run: mvn test
```

This creates **6 jobs** (2 OS × 3 Java versions).

---

## Caching for Faster Builds

```yaml
steps:
  - uses: actions/checkout@v4
  
  - name: Cache Maven dependencies
    uses: actions/cache@v4
    with:
      path: ~/.m2/repository
      key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
      restore-keys: |
        ${{ runner.os }}-maven-

  - run: mvn clean install
```

**How caching works:**
1. First run: Downloads all dependencies, saves cache
2. Subsequent runs: Restores cache, skips downloads
3. Cache invalidated when `pom.xml` changes

---

## Multi-Job Workflows

```yaml
name: Full CI/CD Pipeline

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'
      - run: mvn clean package -DskipTests
      - uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: target/*.jar

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: mvn test

  deploy:
    needs: [build, test]           # Runs after BOTH build and test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to server
        run: echo "Deploying to production..."
```

---

## Key Takeaways

1. Workflows are YAML files in `.github/workflows/`
2. **Jobs** run in parallel by default; use `needs` for sequential execution
3. **Matrix strategies** test across multiple configurations efficiently
4. **Caching** dramatically speeds up builds by preserving dependencies
5. **Triggers** include push, PR, schedule (cron), and manual dispatch
6. **Runners** can be GitHub-hosted (easy) or self-hosted (more control)
