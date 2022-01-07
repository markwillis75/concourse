## Tutorial
https://concourse-ci.org/getting-started.html

---

## Hello World Pipeline
The simplest Concourse pipeline is made of two objects:

- An unordered list of `Jobs` which contains...
- An ordered list of `Steps`

The order of the jobs does not matter. **The order of jobs does not define the structure of the pipeline**

Unlike jobs, the order of steps does matter! **Concourse will run the steps in the order that they are listed.**

### What is a Step?
A step is a single container running on a Concourse worker. Each step in a job plan runs in its own container. You can run anything you want inside the container (i.e. run my tests, run this bash script, build this image, etc.).

So if you have a job with five steps Concourse will create five containers, one for each step. Therefore, we need to tell Concourse the following about each step:

- What type of worker to run the task on (linux/windows/darwin)

- What container image to use (Linux only)

- What command to run inside the container

## Inputs and Outputs
Inputs and outputs are directories that get passed between steps. We'll refer to both inputs and outputs as **artifacts.**

Add a `task.config.outputs` element

Concourse makes output directories for you and will pass any contents inside the folder onto later steps.

### Passing outputs to another task
To pass artifacts from one task to another, the first task must declare the artifact as an `output`. The second task must then declare the *same* artifact as an `input`.

### How does Concourse track artifacts
- Concourse runs the task step `output-task` with output named `the-artifact`
- Concourse creates an empty artifact, assigns it the name `the-artifact`, and mounts it inside the task container.
- Concourse runs the task step `input-task` with input `the-artifact`
- Concourse looks up, in its list of artifacts for the build, for an artifact named `the-artifact`, and mounts it inside the task container. **If no input with that name is found then the build would fail.**
