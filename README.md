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


## Variables, Secrets and Outputs

The `variables_secrets_outputs` branch demonstrates the usage of variables, secrets and outputs.
Variables and secrets are defined in GitHub repository’s UI under Settings > Secrets and variables > Actions.
They hold central configuration or other repeatedly used values and can be defined on three levels:
* Organization
* Environment
* Repository

Secrets differ from variables in that they can neither be read from the UI nore the workflow logs.
They are invoked with the `${{ VARS.<VAR_NAME> }}` and `${{ SECRETS.<SECRET_NAME> }}` syntax respectively.

Outputs can be declared dynamically during a workflow run and fulfill the purpose of sharing reused values between consequtive jobs.
To declare them, use:

```
outputs:
  output_name: ${{ steps.<step_id>.outputs.output_name }}
```

Assign a value within a step with:
`echo "my_first_output=$my_first_output" >> "$GITHUB_OUTPUT"`

Make the output available in other jobs, by letting a consequtive job depend on the defining job:
`needs: job_id`

Reference the output via:
`${{ needs.job_id.outputs.output_name }}`

Note that an undeclared variable, secret or output does not throw an error.
This can lead to frustrating debugging sessions! :-(


## Matrices

A matrix is a list of values on the job level, each of which creates a distinct run-through of the steps defined afterwards.
It is defined with a `strategy:`, which can also be used to declare the behaviour in a failure case.
For example, it could be set to cancel all parallel runs upon the first error.


## Obtaining Workflow Contexts

GitHub feeds various structured data as contexts into the workflow.
These are available to different parts of the workflow.
E.g., the `steps` context is not available outside of a job, while the `github` context can already be accessed at the top when defining the workflow name.

In some cases, looking through this data can help with debugging an issue.
For example, a variable or secret may have been referenced, leading to an unintended empty variable.

Apart from that, it may just be helpful to get an overview about all available keys.
Afterall, these can vary depending on the triggering event among others.

To log a context to the console during a workflow run, use:
`${{ fromJSON(<context_name>)}}`.
The following contexts exist and may be nested in each other:
`github`, `env`, `vars`, `job`, `jobs`, `steps`, `runner`, `secrets`, `strategy`, `matrix`, `needs`, `inputs`


## Tags and Conditional Jobs

GitHub Actions offers the possibility to deactivate single jobs conditionally with the `if:` key.
This could evaluate information from the contexts or dynamically calculated values from previous jobs within the same workflow run.

These conditions are introduced here alongside with the `tags:` specification on the triggering push event.
This is simply, to provide a sensible use-case, but is not really related otherwise.
Tags in Git are simply additional markers for a specific revision.
The `tags:` key can be used to define valid patterns of tag names.
The patterns can be described as a limited implementation of regular expressions, i.e., some unusual patterns will not be evaluated correctly.

The `tags_and_conditions` workflow introduced in the corresponding branch, also makes use of some new techniques:
* It (effectively) runs at most one of two conditional jobs using the `if:` key.
* It uses the `tags:` event specification to filter for matching tags triggering the workflow
* It evaluates specific context keys to define customized conditions which cannot be produced with the default event specification
* It uses [expressions](https://docs.github.com/en/actions/learn-github-actions/expressions) to match patterns in the context
* It explicitly exports a shell variable to the job runner’s environment in order to use it in another step


## Add integration

The `add_integration` branch adds an exemplary project from an codekata, which was meant to be refactored.
So, do not mind the bad code quality.

The project is merely used as a code base to run automated tests against.
This is what is done in the corresponding workflow file.

It checks out the code base, sets up node and runs `npm install`, `npm ci`, `npm test`, `npm build`.
Additionally, it builds the Docker image without pushing it.
