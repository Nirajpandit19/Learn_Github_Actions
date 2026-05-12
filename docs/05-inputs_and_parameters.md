# Inputs/Parameters in Github Actions

To understand this concept let's take an example:
Let say we have three environments, which are dev, test, and prod.

So we want a workflow to deploy dev, test, and prod. For better way we can have one workflow which deploy the application on each of the environments so we have two parameters which we have to pass.
1. environment = dev
2. artifact_tag = 2.0.1

If there were no concept of inputs/parameters we endup creating three workflow files for each of the environments. Which is not a better option.

