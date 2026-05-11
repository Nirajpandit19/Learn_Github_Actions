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