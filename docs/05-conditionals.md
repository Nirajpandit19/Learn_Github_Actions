Here we are going to run the steps and jobs based on certain condition.
Like run this specific step if branch is main, or run this specific step id actor means the user by which the workflow id getting trigger is your_github_user_name.

Two conditions yet OR and AND denoted by OR --> || and AND --> && keep in mind

so here we create the truth table for OR and AND

So lets get started 

1. Running a specific step if the branch is main.

``` yaml
     name: IF Demo
     on:
         push:
             branches:
                 - main # specifing main branch 
     jobs:
         demo:
             runs-on: ubuntu-latest
             steps:
                 - name: run if branch is main
                   if: github.ref == 'refs/heads/main' # condition for this step if branch is main
                   run: echo "run if branch is main"
                 - name: runs on any branch
                   run : echo "It will be running on any branch"

```

2. Running a specific job with two conditions aggregated with or operator.

``` yaml
    name: Or Condition Demo
    on:
        push:
            branches:
                - main
    jobs:
        demo:
            runs-on: ubuntu-latest
            steps:
                - name: run for niraj and on the main branch
                  if: github.ref == 'refs/heads/main' || github.actor == 'Nirajpandit19' 
                  run: echo "It is triggered by either main branch or by Nirajpandit19"

```
If you notice here in the step above we are checking for two conditions one is if the brach is *main* and the other condition is if the actor *the user* by which the job is getting to be  triggred is *Nirajpandit19* or not.

In the both examples we discussed yet we are using conditionals at the *steps level* but now we'll be using the conditions at the *job level*

