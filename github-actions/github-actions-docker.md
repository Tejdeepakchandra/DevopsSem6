# Docker & GitHub Actions

## Building Docker Images in CI

GitHub Actions can build Docker images as part of your CI/CD pipeline, ensuring every commit produces a tested, deployable container image.

---

## Building and Pushing to Docker Hub

```yaml
name: Build & Push to Docker Hub

on:
  push:
    branches: [main]

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and Push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ${{ secrets.DOCKERHUB_USERNAME }}/my-app:latest
            ${{ secrets.DOCKERHUB_USERNAME }}/my-app:${{ github.sha }}
```

### Setting up Docker Hub Secrets:
1. Go to Docker Hub → Account Settings → Security → New Access Token
2. In GitHub repo → Settings → Secrets and variables → Actions
3. Add `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`

---

## Pushing to GitHub Container Registry (GHCR)

GHCR is GitHub's own container registry – no external account needed.

```yaml
name: Build & Push to GHCR

on:
  push:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write       # Required for GHCR push

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Log in to GHCR
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}    # Auto-provided by GitHub

      - name: Extract metadata (tags, labels)
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

### GHCR vs Docker Hub:

| Feature         | Docker Hub              | GHCR                    |
|----------------|-------------------------|-------------------------|
| Auth           | Docker Hub token        | GitHub token (automatic)|
| Free tier      | 1 private repo          | 500MB for private       |
| Public images  | Unlimited               | Unlimited               |
| Integration    | Needs secrets setup     | Native GitHub integration|

---

## Complete CI/CD with Docker

A real-world pipeline that builds, tests, and deploys a Docker image:

```yaml
name: CI/CD Pipeline with Docker

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'
      - name: Run tests
        run: mvn test

  build-image:
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push'     # Only build image on push, not PR
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Docker image
        run: docker build -t my-app:${{ github.sha }} .
      
      - name: Run container health check
        run: |
          docker run -d --name test-container -p 8080:8080 my-app:${{ github.sha }}
          sleep 10
          curl -f http://localhost:8080/health || exit 1
          docker stop test-container

  push-image:
    needs: build-image
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:latest

  deploy:
    needs: push-image
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to server
        run: |
          echo "Deploying ghcr.io/${{ github.repository }}:latest"
          # ssh user@server "docker pull ghcr.io/... && docker-compose up -d"
```

---

## Deployment to Cloud/Servers

### Deploy to a Server via SSH:
```yaml
- name: Deploy via SSH
  uses: appleboy/ssh-action@v1
  with:
    host: ${{ secrets.SERVER_HOST }}
    username: ${{ secrets.SERVER_USER }}
    key: ${{ secrets.SSH_PRIVATE_KEY }}
    script: |
      docker pull ghcr.io/${{ github.repository }}:latest
      docker-compose down
      docker-compose up -d
```

### Deploy to AWS ECS:
```yaml
- name: Deploy to Amazon ECS
  uses: aws-actions/amazon-ecs-deploy-task-definition@v1
  with:
    task-definition: task-definition.json
    service: my-service
    cluster: my-cluster
```

---

## Key Takeaways

1. Use `docker/build-push-action` for efficient Docker builds in CI
2. **GHCR** integrates natively with GitHub – uses `GITHUB_TOKEN` automatically
3. **Docker Hub** requires manual secret setup but has wider ecosystem
4. Always **test the image** before pushing (health checks, smoke tests)
5. Use **multi-stage Docker builds** to keep images small
6. Deployment can target servers (SSH), cloud platforms (AWS, Azure), or Kubernetes
