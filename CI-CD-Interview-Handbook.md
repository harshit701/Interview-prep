# CI/CD Pipeline Interview Handbook

> **Goal:** Understand the complete journey of code from a developer's
> laptop to a live application on AWS EC2.

---

# 1. What is CI/CD?

## Continuous Integration (CI)

Continuous Integration is the practice of automatically validating every
code change after developers push code.

Instead of manually checking whether the application still works, a CI
tool automatically:

- Downloads latest code
- Installs dependencies
- Runs linting
- Runs automated tests
- Builds the application

**Goal:** Catch bugs early before deployment.

---

## Continuous Delivery (CD)

Continuous Delivery automatically prepares the application for
deployment after CI succeeds.

Typical tasks:

- Build Docker image
- Push image to registry (ECR)
- Deploy application to server

**Goal:** Release software quickly and safely.

---

# 2. Important Definitions

## Git

A version control system used to track source code changes.

## GitHub

A cloud platform that stores Git repositories and generates events such
as Push, Pull Request and Merge.

## Webhook

A webhook is an automatic HTTP POST request sent from one application to
another when a specific event occurs.

Example:

Developer pushes code → GitHub creates Push Event → GitHub sends webhook
→ Jenkins starts pipeline

Developers never write code to generate this payload. GitHub does it
automatically.

## Jenkins

Jenkins is an automation server used for CI/CD.

It automates:

- Build
- Test
- Lint
- Docker Build
- Deployment

## Jenkinsfile

A text file stored in the repository describing every pipeline stage.

## Docker

Docker packages an application with all required dependencies so it runs
the same on every machine.

## Docker Image

A blueprint of the application.

## Docker Container

A running instance of a Docker image.

## Docker Hub

Public Docker image registry.

## AWS ECR

Private Docker registry provided by AWS for production images.

## EC2

A virtual machine where the application runs.

## Docker Compose

Defines multiple containers that should run together.

Example:

- Backend
- PostgreSQL
- Redis

---

# 3. Complete CI/CD Flow

```text
Developer
    │
git add
git commit
git push
    │
GitHub Repository
    │
Push Event
    │
Webhook
    │
Jenkins
    │
Checkout Code
    │
Install Dependencies
    │
Run Lint
    │
Run Tests
    │
Build Application
    │
Build Docker Image
    │
Push Image to AWS ECR
    │
SSH into EC2
    │
Pull Latest Image
    │
Stop Old Container
    │
Remove Old Container
    │
Run New Container
    │
Application Live
```

---

# 4. Step-by-Step Explanation

## Step 1 -- Developer Pushes Code

Developer writes code on a feature branch.

```bash
git add .
git commit -m "Add authentication"
git push origin feature/auth
```

After review, Pull Request is merged into `main`.

---

## Step 2 -- GitHub Creates an Event

GitHub detects the push.

If a webhook is configured, GitHub automatically sends an HTTP POST
request to Jenkins containing:

- Repository
- Branch
- Commit SHA
- Commit message
- Author

---

## Step 3 -- Jenkins Starts Pipeline

Jenkins receives the webhook.

It authenticates with GitHub (PAT or SSH Key), checks out the latest
code and reads the `Jenkinsfile`.

---

## Step 4 -- Install Dependencies

```bash
npm ci
```

Uses `package-lock.json` to install exact dependency versions.

---

## Step 5 -- Lint

```bash
npm run lint
```

Checks coding standards.

Pipeline stops if lint fails.

---

## Step 6 -- Tests

```bash
npm test
```

Runs automated tests.

Pipeline stops if tests fail.

---

## Step 7 -- Build

```bash
npm run build
```

Compiles TypeScript and creates production-ready files.

---

## Step 8 -- Docker Build

```bash
docker build -t backend:1.0 .
```

Docker reads the Dockerfile and creates a self-contained image.

**Interview Question:** Why install dependencies again inside Docker?

**Answer:** Jenkins installs dependencies only for validation. Docker
installs them again because the image must contain everything required
to run on any server.

---

## Step 9 -- Push Image to AWS ECR

```bash
docker push <image>
```

Production servers pull Docker images from ECR instead of receiving
source code.

---

## Step 10 -- Deploy on EC2

Jenkins connects to EC2 through SSH.

Typical commands:

```bash
docker pull <image>
docker stop backend || true
docker rm backend || true
docker run -d --name backend ...
```

Or

```bash
docker compose pull
docker compose up -d
```

---

# 5. Environment Variables

Development

- .env
- .env.example
- .gitignore

Production

- Jenkins Credentials
- AWS Secrets Manager
- Environment variables injected during deployment

Never commit secrets to Git.

---

# 6. Controller vs Agent

## Controller

- Schedules jobs
- Manages pipelines
- Coordinates agents

## Agent

- Executes pipeline
- Has tools installed such as Git, Node.js, npm and Docker

Multiple agents allow multiple builds to run simultaneously.

---

# 7. Frequently Asked Interview Questions

### What is CI/CD?

CI automatically validates code changes. CD automatically prepares and
deploys validated code to production.

### What is a Webhook?

A webhook is an automatic HTTP POST request sent when an event occurs.

### How does Jenkins know code changed?

GitHub sends a webhook to Jenkins whenever a configured event
(push/merge) happens.

### Why Docker?

It provides a consistent runtime environment across developer machines,
testing and production.

### Why AWS ECR instead of Docker Hub?

ECR is private, integrates with AWS IAM, and is commonly used for
production deployments.

### Difference between Docker Image and Container?

Image = blueprint. Container = running instance of that blueprint.

### Why Docker Compose?

It manages multiple containers together using one configuration file.

### Explain your deployment pipeline.

Developers push code to GitHub. GitHub sends a webhook to Jenkins.
Jenkins checks out the latest code, installs dependencies, runs lint,
tests and build. It then builds a Docker image, pushes it to AWS ECR,
connects to EC2, pulls the latest image, stops the old container and
starts a new one.

---
