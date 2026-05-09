# GitHub Actions — Learning Notes

> A structured guide to understanding and implementing GitHub Actions for CI/CD workflows.

---

## Table of Contents

- [What is GitHub Actions?](#what-is-github-actions)
- [CI/CD Concepts](#cicd-concepts)
- [Evolution of GitHub & GitHub Actions](#evolution-of-github--github-actions)
- [Workflow Structure](#workflow-structure)
- [Workflow Examples](#workflow-examples)
  - [Hello World](#hello-world)
  - [Multiple Commands in a Step](#multiple-commands-in-a-step)
  - [Multiple Jobs in Parallel](#multiple-jobs-in-parallel)
  - [Multiple Jobs in Sequential Mode](#multiple-jobs-in-sequential-mode)
- [Workflow Triggers](#workflow-triggers)
  - [Types of Triggers](#types-of-triggers)
  - [Push Trigger](#push-trigger)
  - [Pull Request Trigger](#pull-request-trigger)
  - [Combined Triggers](#combined-triggers)
  - [Branch-Specific Triggers](#branch-specific-triggers)
- [Variables and Secrets](#variables-and-secrets)
  - [Environment Variables](#environment-variables)
  - [Default Environment Variables](#default-environment-variables)
  - [Custom Variables](#custom-variables)
  - [Secrets](#secrets)
  - [Using Secrets in a Workflow](#using-secrets-in-a-workflow)

---

## What is GitHub Actions?

**GitHub Actions** is a CI/CD and automation service provided by GitHub. It allows DevOps engineers to streamline the software development lifecycle by automating:

```
Build  →  Test  →  Deploy
```

It is primarily used for implementing **CI/CD workflows** directly within a GitHub repository — no external tools required.

---

## CI/CD Concepts

### Continuous Integration (CI)
A DevOps practice where developers **regularly integrate their code** into a shared repository. Each integration is automatically verified by:
- Automated builds
- Automated tests

This catches bugs early and keeps the codebase stable.

### Continuous Delivery (CD)
Automates the process of **preparing code for production releases**, but requires **manual approval** before the code is actually deployed to production.

### Continuous Deployment (CD)
Fully automates the release process — code is **deployed to production automatically** without any manual intervention, as long as all tests pass.

| | Continuous Delivery | Continuous Deployment |
|---|---|---|
| Build & Test | ✅ Automated | ✅ Automated |
| Release to Production | ⚠️ Manual approval | ✅ Fully automated |

---

## Evolution of GitHub & GitHub Actions

| Year | Event |
|------|-------|
| 2008 | GitHub launched — primarily for hosting repositories, forking, and version control |
| 2018 | Microsoft acquires GitHub |
| Post-2018 | Major features added: |
| | → **GitHub Actions** — CI/CD automation |
| | → **GitHub Advanced Security** — DevSecOps capabilities |
| | → **GitHub Container Registry (GHCR)** — storing Docker build images |
| | → **GitHub Copilot** — AI assistant for writing code |

---

## Workflow Structure

### Beginner-Friendly Answer
| Term | Description |
|------|-------------|
| **Workflow** | The complete automation process in GitHub Actions |
| **Job** | A section of work inside the workflow |
| **Step** | An individual command or action executed inside a job |

### Professional Interview Answer
> A **workflow** is the overall CI/CD automation pipeline defined in YAML. **Jobs** are independent units of execution running on GitHub-hosted runners. **Steps** are the individual commands or reusable actions executed within a job.

### Key Points
- Workflows live inside `.github/workflows/` in your repository
- A single repository can have **multiple workflows** (e.g. one for CI, one for deployment)
- Jobs inside a workflow **run in parallel by default**
- Use `needs:` to run jobs **sequentially**

```
.github/
└── workflows/
    ├── ci.yml        # Build & Test workflow
    └── deploy.yml    # Deploy workflow
```

---

## Workflow Examples

### Hello World

The simplest possible workflow — triggered manually.

```yaml
name: Hello World

on:
  workflow_dispatch   # Trigger workflow manually

jobs:
  hello:
    runs-on: ubuntu-latest

    steps:
      - name: Hello Step
        run: echo "Hello, welcome to GitHub Actions!"
```

---

### Multiple Commands in a Step

Use the `|` (pipe) symbol to run multiple shell commands inside a single step.

```yaml
name: Hello World

on:
  workflow_dispatch

jobs:
  hello:
    runs-on: ubuntu-latest

    steps:
      - name: Hello Step
        run: |
          echo "Command 1 is executing"
          echo "Command 2 is executing"
```

---

### Multiple Jobs in Parallel

By default, all jobs run **simultaneously** (in parallel) on separate runners.

```yaml
name: Parallel Jobs Example

on:
  workflow_dispatch

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build step
        run: echo "Building the code"

  test:
    runs-on: ubuntu-latest
    steps:
      - name: Test step
        run: echo "Testing the code"

  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy step
        run: echo "Deploying the artifact"
```

---

### Multiple Jobs in Sequential Mode

Use `needs:` to make a job wait for another job to finish before it starts.

```yaml
name: Sequential Jobs Example

on:
  workflow_dispatch

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build step
        run: echo "Building the code"

  test:
    runs-on: ubuntu-latest
    needs: build             # Waits for 'build' to complete
    steps:
      - name: Test step
        run: echo "Testing the code"

  deploy:
    runs-on: ubuntu-latest
    needs: test              # Waits for 'test' to complete
    steps:
      - name: Deploy step
        run: echo "Deploying the artifact"
```

> **Flow:** `build` → `test` → `deploy`

---

## Workflow Triggers

Triggers are defined using the `on:` keyword. They determine **which event causes the workflow to run**.

### Types of Triggers

| Type | Description | Example |
|------|-------------|---------|
| **Event-Based** | Triggered by GitHub events | `push`, `pull_request` |
| **Manual** | Triggered manually from the UI | `workflow_dispatch` |
| **Scheduled** | Triggered on a cron schedule | `schedule` |
| **Workflow** | Triggered by another workflow | `workflow_call`, `workflow_run` |

---

### Push Trigger

Triggers the workflow on every push to **any branch**.

```yaml
on:
  push
```

---

### Pull Request Trigger

Triggers the workflow on every pull request to **any branch**.

```yaml
on:
  pull_request
```

---

### Combined Triggers

Trigger on both push and pull request events.

```yaml
on: ["push", "pull_request"]
```

---

### Branch-Specific Triggers

Trigger only on specific branches, or use patterns with wildcards.

```yaml
on:
  push:
    branches:
      - development       # Exact branch name
      - "dev/**"          # Any branch starting with 'dev/'

  pull_request:
    branches:
      - main              # Only PRs targeting main
```

> **Tip:** Use `"dev/**"` as a regular expression pattern to match any branch like `dev/feature-1`, `dev/hotfix`, etc.

---

## Variables and Secrets

### Environment Variables

Environment variables allow you to **pass dynamic values** into your workflow steps without hardcoding them.

They can be defined at three levels:

| Level | Scope | Keyword |
|-------|-------|---------|
| Workflow | Available to all jobs and steps | `env:` at root level |
| Job | Available to all steps in that job | `env:` under a job |
| Step | Available only in that step | `env:` under a step |

---

### Default Environment Variables

GitHub provides built-in variables automatically available in every workflow:

| Variable | Description |
|----------|-------------|
| `$GITHUB_REPOSITORY` | Owner and repo name (e.g. `Nirajpandit19/Boardgame`) |
| `$GITHUB_SHA` | The commit SHA that triggered the workflow |
| `$GITHUB_REF` | The branch or tag ref (e.g. `refs/heads/main`) |
| `$GITHUB_ACTOR` | The username of the person who triggered the workflow |
| `$GITHUB_WORKSPACE` | The path to the checked-out repository on the runner |
| `$RUNNER_OS` | The OS of the runner (e.g. `Linux`) |

```yaml
steps:
  - name: Print default variables
    run: |
      echo "Repo: $GITHUB_REPOSITORY"
      echo "Commit: $GITHUB_SHA"
      echo "Branch: $GITHUB_REF"
      echo "Triggered by: $GITHUB_ACTOR"
```

---

### Custom Variables

You can define your own variables at the workflow, job, or step level.

```yaml
name: Variables Example

on:
  workflow_dispatch

env:
  APP_NAME: "my-app"          # Workflow-level: available everywhere
  ENVIRONMENT: "production"

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      BUILD_VERSION: "1.0.0"  # Job-level: available to all steps in this job

    steps:
      - name: Print workflow and job variables
        run: |
          echo "App: $APP_NAME"
          echo "Environment: $ENVIRONMENT"
          echo "Version: $BUILD_VERSION"

      - name: Step with its own variable
        env:
          STEP_MESSAGE: "Building now..."   # Step-level: only in this step
        run: echo "$STEP_MESSAGE"
```

---

### Secrets

**Secrets** are encrypted variables used to store sensitive information like:
- API keys
- Docker Hub credentials
- SSH private keys
- Database passwords

> ⚠️ **Never hardcode sensitive values directly in your YAML file.** Always use secrets.

#### How to Add a Secret in GitHub:
1. Go to your repository → **Settings**
2. Click **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Add a name (e.g. `DOCKER_USERNAME`) and value
5. Click **Add secret**

---

### Using Secrets in a Workflow

Secrets are accessed using the `${{ secrets.SECRET_NAME }}` syntax.

```yaml
name: Docker Build and Push

on:
  push:
    branches:
      - main

jobs:
  docker:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Log in to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

      - name: Build Docker image
        run: docker build -t ${{ secrets.DOCKER_USERNAME }}/my-app:latest .

      - name: Push Docker image
        run: docker push ${{ secrets.DOCKER_USERNAME }}/my-app:latest
```

#### Key Rules for Secrets:
- Secrets are **masked in logs** — GitHub replaces them with `***`
- Secrets are **not passed to workflows triggered by forks** by default
- Secret names are **case-insensitive** but conventionally written in `UPPER_SNAKE_CASE`
- You can also define secrets at the **organization level** to share across multiple repositories

---

*Notes by Pandit Niraj Raj | GitHub: [Nirajpandit19](https://github.com/Nirajpandit19)*