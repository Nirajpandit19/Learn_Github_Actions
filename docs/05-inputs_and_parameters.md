# Inputs/Parameters in Github Actions

To understand this concept let's take an example:
Let say we have three environments, which are dev, test, and prod.

So we want a workflow to deploy dev, test, and prod. For better way we can have one workflow which deploy the application on each of the environments so we have two parameters which we have to pass.
1. environment = dev
2. artifact_tag = 0.0.1

If there were no concept of inputs/parameters we endup creating three workflow files for each of the environments. Which is not a better option.

``` yaml
name: Input Demonstration

on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Choose the environment to deploy"
        required: true
        default: dev
        type: choice
        options:
          - dev
          - test
          - prod

      artifact_tag:
        description: "Choose the artifact tag to deploy"
        required: true
        type: string

jobs:
  input-demo:
    runs-on: ubuntu-latest

    steps:
      - name: Deploy the App
        run: echo "Deploying version ${{ inputs.artifact_tag }} application to ${{ inputs.environment }}"

```

# Outputs
Outputs are the `values returned` by one task which is used by another task.
Using `outputs` we can share data between tsaks/steps.
Sharing data between jobs. 

