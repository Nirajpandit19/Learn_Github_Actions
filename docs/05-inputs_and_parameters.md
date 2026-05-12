# Inputs / Parameters and Outputs in GitHub Actions

---

## Table of Contents

- [Inputs / Parameters](#inputs--parameters)
  - [Why Do We Need Inputs?](#why-do-we-need-inputs)
  - [Input Types](#input-types)
  - [Workflow Example](#workflow-example---inputs)
  - [How to Pass Inputs](#how-to-pass-inputs)
- [Outputs](#outputs)
  - [Why Do We Need Outputs?](#why-do-we-need-outputs)
  - [How $GITHUB_OUTPUT Works](#how-github_output-works)
  - [Output Scopes](#output-scopes)
  - [Workflow Example](#workflow-example---outputs)

---

## Inputs / Parameters

### Why Do We Need Inputs?

Consider a real-world scenario — you have three environments: `dev`, `test`, and `prod`.

You need a workflow to deploy your application to each environment. Without inputs, you would end up creating **three separate workflow files**:

```
deploy_dev.yaml   ❌
deploy_test.yaml  ❌
deploy_prod.yaml  ❌
```

This is repetitive, hard to maintain, and not scalable. With inputs, **one workflow handles all environments** — you just pass different values at runtime:

```
deploy.yaml  ✅  (pass environment=dev / test / prod as input)
```

The two parameters needed at runtime:
1. `environment` — which environment to deploy to (`dev`, `test`, `prod`)
2. `artifact_tag` — which version of the app to deploy (e.g. `0.0.1`)

---

### Input Types

| Type | Description | Example |
|------|-------------|---------|
| `string` | Free text input | Version number, branch name |
| `choice` | Dropdown with fixed options | Environment selection |
| `boolean` | True / False checkbox | Enable debug mode |
| `number` | Numeric input | Port number, replica count |
| `environment` | Picks from GitHub Environments | Deployment environment |

---

### Workflow Example — Inputs

```yaml
name: Input Demonstration

on:
  workflow_dispatch:        # Manual trigger — inputs only work with workflow_dispatch
    inputs:
      environment:
        description: "Choose the environment to deploy"
        required: true
        default: dev
        type: choice        # Dropdown — restricts to defined options only
        options:
          - dev
          - test
          - prod

      artifact_tag:
        description: "Choose the artifact tag to deploy"
        required: true
        type: string        # Free text — user types the version manually

jobs:
  input-demo:
    runs-on: ubuntu-latest

    steps:
      - name: Deploy the App
        run: echo "Deploying version ${{ inputs.artifact_tag }} application to ${{ inputs.environment }}"
```

> **Note:** Inputs are accessed using the `${{ inputs.INPUT_NAME }}` syntax and are only available when the trigger is `workflow_dispatch`.

---

### How to Pass Inputs

1. Go to your GitHub repository
2. Click **Actions** tab
3. Select the workflow
4. Click **Run workflow**
5. Fill in the input fields → click **Run workflow**

GitHub renders the inputs as a form based on their types — `choice` shows a dropdown, `string` shows a text box, `boolean` shows a checkbox.

---

## Outputs

### Why Do We Need Outputs?

Outputs are **values produced by one job or step that can be consumed by another job or step.**

A common real-world use case:

```
build job   →  generates artifact version (e.g. 0.0.1)
                        ↓
deploy job  →  needs that version to know what to deploy
```

Without outputs, the deploy job has no way of knowing what the build job produced.

---

### How $GITHUB_OUTPUT Works

Each job in GitHub Actions runs on a **separate, isolated runner VM**. This means normal shell variables do not survive between jobs — they die when the job ends.

```bash
# This dies when the job ends ❌
VERSION="0.0.1"

# This persists and is available to other jobs ✅
echo "version=0.0.1" >> $GITHUB_OUTPUT
```

`$GITHUB_OUTPUT` is a **special file** that GitHub reads after each step completes. Any `key=value` pairs written to it are stored by GitHub and made available to subsequent steps and jobs through the outputs context.

---

### Output Scopes

There are two scopes for accessing outputs depending on where you need them:

**Within the same job — use `steps` context:**
```yaml
# Access output from a previous step in the same job
run: echo "Version is ${{ steps.STEP_ID.outputs.version }}"
```

**From a different job — use `needs` context:**
```yaml
# Access output from a previous job
run: echo "Version is ${{ needs.JOB_NAME.outputs.version }}"
```

> For a job's output to be accessible via `needs`, it must be **explicitly declared** under the `outputs:` block of that job.

---

### Workflow Example — Outputs

```yaml
name: Output Demonstration

on: workflow_dispatch

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.step1.outputs.version }}   # Declare job-level output

    steps:
      - name: Build and generate version
        id: step1                                    # id is required to reference this step
        run: echo "version=0.0.1" >> $GITHUB_OUTPUT # Write output to GitHub's output file

      # Accessing output within the same job using steps context:
      # - name: Use output within same job
      #   run: echo "The artifact version is ${{ steps.step1.outputs.version }}"

  deploy:
    runs-on: ubuntu-latest
    needs: build                                     # Wait for build job to complete

    steps:
      - name: Deploy the artifact
        run: echo "Deploying app with version ${{ needs.build.outputs.version }}"
        # Accessing output from a different job using needs context
```

**Flow:**

```
build job
  └── step1 runs → writes "version=0.0.1" to $GITHUB_OUTPUT
  └── job declares output: version = steps.step1.outputs.version
          ↓
deploy job (needs: build)
  └── reads version via needs.build.outputs.version → "0.0.1"
```

---

### Key Rules to Remember

| Rule | Detail |
|------|--------|
| Step must have an `id` | Without `id:` you cannot reference a step's output |
| Job must declare `outputs:` | Without this block, other jobs cannot access the value |
| Use `needs:` to sequence jobs | The consuming job must declare `needs: JOB_NAME` |
| Write using `>>` not `>` | `>>` appends to the file; `>` would overwrite it, breaking multiple outputs |

---

*Notes by Pandit Niraj Raj | GitHub: [Nirajpandit19](https://github.com/Nirajpandit19)*