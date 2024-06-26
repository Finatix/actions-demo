# GitHub Actions showcase

This repository showcases the CICD tool “Github Actions” and was created for a presentation in the DevOps guild.
The repository’s history evolves to introduce changes demonstrating one feature after the other.

The remainder of this README will grow along with every commit, to provide further explanations to the changes.


## Hello World

GitHub Actions are ran in so called workflows, which are defined as YAML files.
They are triggered by certain repository events and can run arbitrary scripts or call other GitHub Actions.

Their basic structure is:

```
name: Name of the workflow
run-name: Name of the run

on:                               # List of event types that trigger a workflow run
  push:                           # Exemplary common event type
    branches: [ 'main' ]          # Further event type conditions (in this case the branch to be pushed to)

jobs:                             # List of separated tasks, each being run on a new container
  job1:                           # job object
    name: Name of the job
    runs-on: ubuntu-22.04         # Image to start the job runner with
    steps:                        # List of steps to perform
      - name: Checkout            # Checkout/Clone the repository (not contained within the image)
        uses: actions/checkout@v3 # Call to an action from the GitHub Marketplace

      - name: Name of the second step
        run: echo "Hello World!"
```

By convention, workflows reside in `.github/workflows/`.

The `hello_world` branch introduces the file `.github/workflows/hello_world.yaml`.
It contains a simple workflow which is triggered, when a new HEAD is pushed onto the `hello_world` branch.
Two jobs are being run to provide an overview of the basic workflow structure.
The first one also declares an environment (`TEST`) and an environment variable (`name`).
Note that each job is being ran in its dedicated runner container.
This means, the declared variables (within scripts or the `env` object) are only available within the same job, since that environment only exists within the runner.

There are two further things to take note about:
* A runner does not contain the repository by default.
  Any interaction with the repository code, necessitates a checkout step prior to that.
  Typically this should be done at the beginning of the job.
* Jobs are being run in parallel unless they are specified to depend on another job by using the `needs:` key at the job’s top level.
