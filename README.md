This is a repository for learning Github Actions.
What is Github Actions ?

Github actions is a service used by Devops Engineers provided by git for streamline the process of software development lifecycle of build ---> Test ---> Deploy activities.
It typically used for implementing CI/CD Workflows.

# Continuos Integration
CI is a devops practice where developers regularly integrate their code to a shared repository. Each integration is automatically verified by automated builds and tests.

# CD - Continuous Deployment or Continuous Delivery
Both operation automate the process of prepairing code for production releases.But they differ in one key aspect.

# Continuous Delivery
Needs manual approval for prepairing code for the production releases.

# Continuous Deployment
Approves the process of prepairing code for the production releases without any manual intervension.

# Evolution of Github and Github Actions

Github was initially launched in 2008. Most tasks performed if for hosting the repositories and fork etc.

Microsoft aquires github in 2018.
Then added features for github ---
    *Github Actions*
    *Github Advanced Security (DevSecOps)*
    *Github Container Registory (For storing the build images)*
    *Github Copilot which is a assistant provided by microsoft for writing the code*

# Github workflow structure
    *Beginner-Friendly Answer*

    Workflow is the complete automation process in GitHub Actions.
    Jobs are different sections of work inside the workflow.
    Steps are individual commands or actions executed inside each job.

    *Professional Interview Answer*

    A workflow is the overall CI/CD automation pipeline defined in YAML.
    Jobs are independent units of execution running on GitHub runners, and steps are the individual commands or reusable actions executed within a job.

    *We can have multiple workflows in a single repository like we can have one workflow for CI(Continuos Integration) which consit of install dependencies, build, test and another workflow for push artifacts, Deploy*

# Hello world program in Github Actions workflow
Requirements:
    Having a github repository.
    Go to actions --> Configure
    Inside .gihub/workflows/hello.yaml
    
    name: Hello World

    on:
        workflow_dispatch   # Trigger workflow manually

    jobs:
        hello:
            runs-on: ubuntu-latest

            steps:
            - name: Hello Step
              run: echo "Hello welcome to GitHub Actions"

# Running multiple shell commands in a single step

    Inside .gihub/workflows/hello.yaml
    name: Hello World

    on:
      workflow_dispatch   # Trigger workflow manually

    jobs:
      hello:
        runs-on: ubuntu-latest

        steps:
          - name: Hello Step
            run: |
                echo "Hello Step 1 command 1 getting execute"
                echo "Hello Step 1 command 2 getting execute"

# Creating multiple jobs in a single workflow
    name: Hello World

    on:
      workflow_dispatch   # Trigger workflow manually

    jobs: # Jobs are independent units of execution running on GitHub runners
      build:
        runs-on: ubuntu-latest # A virtual machine where the jobs got executed

        steps: # steps are the individual commands or reusable actions executed within a job
            - name: Build step
              run: echo "Building the code"

      test:
        runs-on: ubuntu-latest # A virtual machine where the jobs got executed

        steps: # steps are the individual commands or reusable actions executed within a job
            - name: Build step
              run: echo "Testing the code"

      deploy:
        runs-on: ubuntu-latest # A virtual machine where the jobs got executed

        steps: # steps are the individual commands or reusable actions executed within a job
            - name: Build step
              run: echo "Deploying the Artifact"

# Running multiple jobs in a sequential mode
    name: Hello World

    on:
      workflow_dispatch   # Trigger workflow manually

    jobs: # Jobs are independent units of execution running on GitHub runners
      build:
        runs-on: ubuntu-latest # A virtual machine where the jobs got executed

        steps: # steps are the individual commands or reusable actions executed within a job
            - name: Build step
              run: echo "Building the code"

      test:
        runs-on: ubuntu-latest # A virtual machine where the jobs got executed
        needs: build # "needs" define here the previous declared job should be executed
        steps: # steps are the individual commands or reusable actions executed within a job
            - name: Build step
              run: echo "Testing the code"

      deploy:
        runs-on: ubuntu-latest # A virtual machine where the jobs got executed
        needs: test
        steps: # steps are the individual commands or reusable actions executed within a job
            - name: Build step
              run: echo "Deploying the Artifact"

# Github actions workflow triggers & types
In github actions triggers are defined using *on* keyword which determine on which action a workflow should run.

Example: on:
      workflow_dispatch   # Trigger workflow manually

    Types of triggers
    1. Event-Based Triggers
    2. Manual Triggers
    3. Scheduled Triggers
    4. Workflow Triggers


Now we'll see how we can use the push and pull_requests triggers

