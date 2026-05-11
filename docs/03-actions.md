In github actions Actions is a resuable application or a reusable block of code which perform some pecific task in a workflow.

Like : Checking out the code.
In github actions are prewritten and are configurable.

Using checkout action in our workflow: actions

"with" keyword is used for passing inputs to the actions.

```yaml

    name : Actions Review
    on:
      pull_request:
        branches:
          - main    

    jobs:
      checkout:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout the code
            uses: actions/checkout@v4 # using checkout action for current repository

          - name: print index.html
            run: cat index.html
```
Now we'll be deploying a static website on AWS S3 Bucket, so let's get started
Steps: 
    1. Configure S3 bucket to host a static website
    2. Write github actions workflow to deploy to S3 bucket
    3.  