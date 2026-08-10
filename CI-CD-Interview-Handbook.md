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

```
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

```
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

```
npm ci
```

Uses `package-lock.json` to install exact dependency versions.

---

## Step 5 -- Lint

```
npm run lint
```

Checks coding standards.

Pipeline stops if lint fails.

---

## Step 6 -- Tests

```
npm test
```

Runs automated tests.

Pipeline stops if tests fail.

### What are the layers of a testing pyramid, and why does the shape matter?

```
        /\
       /E2E\        <- few, slow, expensive, high confidence
      /------\
     /Integr. \     <- moderate number, test interactions between components
    /----------\
   /   Unit     \   <- many, fast, cheap, test individual functions/modules
  /--------------\
```

**Unit tests** — test individual functions/modules in isolation (mocking dependencies). Fast, cheap to run in every CI build, should form the bulk of your suite.

**Integration tests** — test how multiple components work together (e.g., a service layer actually hitting a test database). Slower than unit tests, fewer of them.

**End-to-end (E2E) tests** — test a full user flow through the real (or near-real) system. Slowest, most brittle, most expensive to maintain — used sparingly for critical user journeys, not for exhaustive coverage.

### Why does the pipeline stop on lint/test failure instead of continuing to deploy?

This is the entire point of CI — catching problems before they reach production. If a broken build were allowed to continue to deployment, CI would just be a reporting tool rather than a gate, and bugs would reach users that automated checks had already caught.

### What is Test Coverage, and is 100% coverage a meaningful goal?

Test coverage measures what percentage of your code is executed by your test suite. It's a useful signal for finding completely untested code paths, but 100% coverage doesn't mean bug-free — a test can execute a line of code without actually asserting anything meaningful about its behavior. Most teams target a reasonable threshold (e.g., 70-80%) on critical paths, rather than treating the coverage number itself as the goal.

---

## Step 7 -- Build

```
npm run build
```

Compiles TypeScript and creates production-ready files.

---

## Step 8 -- Docker Build

```
docker build -t backend:1.0 .
```

Docker reads the Dockerfile and creates a self-contained image.

**Interview Question:** Why install dependencies again inside Docker?

**Answer:** Jenkins installs dependencies only for validation. Docker
installs them again because the image must contain everything required
to run on any server.

---

## Step 9 -- Push Image to AWS ECR

```
docker push <image>
```

Production servers pull Docker images from ECR instead of receiving
source code.

---

## Step 10 -- Deploy on EC2

Jenkins connects to EC2 through SSH.

Typical commands:

```
docker pull <image>
docker stop backend || true
docker rm backend || true
docker run -d --name backend ...
```

Or

```
docker compose pull
docker compose up -d
```

### What is a Blue-Green Deployment?

Two identical production environments exist ("blue" = currently live, "green" = new version). The new version is deployed to green and fully tested while blue continues serving all live traffic. Once verified, traffic is switched to green all at once (usually via a load balancer or DNS change). If something's wrong, switching back to blue is immediate.

```
Before: Load Balancer -> Blue (v1, live)
                          Green (v2, idle, being tested)

After:  Load Balancer -> Green (v2, live)
                          Blue (v1, idle, kept as instant rollback option)
```

**Trade-off:** requires running two full production environments simultaneously (cost), but gives instant rollback and zero-downtime cutover.

### What is a Canary Deployment?

Instead of switching all traffic at once, the new version is rolled out to a small percentage of traffic first (e.g., 5%), monitored for errors/performance regressions, and gradually increased to 100% if it looks healthy. If problems appear early, only a small fraction of users were affected, and the rollout can be halted.

```
v2 deployed to 5% of traffic -> monitor error rate/latency
  -> looks good -> increase to 25% -> 50% -> 100%
  -> looks bad  -> roll back immediately, only 5% of users were impacted
```

### Blue-Green vs Canary — when would you choose which?

**Blue-Green** — when you want a clean, instant, all-or-nothing cutover with guaranteed instant rollback, and can afford to run duplicate infrastructure.

**Canary** — when you want to catch problems with real production traffic before fully committing, limiting blast radius progressively rather than switching everything at once. Generally preferred for higher-risk changes where you want real-world validation before full exposure.

### What is a Rollback, and how should it fit into the pipeline?

A rollback reverts a deployment to the previous known-good version, typically by redeploying the previous Docker image tag rather than trying to "undo" code changes. A good pipeline makes rollback fast and low-risk — this is why immutable versioned Docker images (not just overwriting `latest`) matter: you can always redeploy exactly what was running before.

```
docker pull backend:1.4   # previous known-good version
docker stop backend && docker rm backend
docker run -d --name backend backend:1.4
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

# 7. Infrastructure as Code

### What is Infrastructure as Code (IaC), and why does it matter?

IaC means defining infrastructure (servers, networking, databases) in version-controlled configuration files (e.g., Terraform, AWS CloudFormation) instead of manually clicking through a cloud console. This makes infrastructure changes reviewable (via pull requests, like code), reproducible (spin up an identical environment from the same config), and auditable (git history shows exactly what changed and when).

```
# Example: Terraform defining an EC2 instance
resource "aws_instance" "app_server" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t3.medium"
}
```

### Why is "manually SSH-ing into a server to fix something" considered a bad practice in a mature CI/CD setup?

Manual changes create **configuration drift** — the running server no longer matches what's defined in version control, so nobody can reliably reproduce that exact environment again, and the next automated deployment might silently overwrite (or conflict with) the manual fix. The goal of a mature pipeline is that servers are treated as disposable/replaceable ("cattle, not pets") — if something's wrong, you fix the deployment config and redeploy, rather than patching the live server by hand.

---

# 8. Frequently Asked Interview Questions

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
