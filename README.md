## Tutorial
https://concourse-ci.org/getting-started.html

---

## Hello World Pipeline
The simplest Concourse pipeline is made of two objects:

- An unordered list of [`Jobs`](https://concourse-ci.org/jobs.html) which contains...
- An ordered list of [`Steps`](https://concourse-ci.org/steps.html)

The order of the jobs does not matter. **The order of jobs does not define the structure of the pipeline**

Unlike jobs, the order of steps does matter! **Concourse will run the steps in the order that they are listed.**

### What is a Step?
A step is a single container running on a Concourse worker. Each step in a job plan runs in its own container. You can run anything you want inside the container (i.e. run my tests, run this bash script, build this image, etc.).

So if you have a job with five steps Concourse will create five containers, one for each step. Therefore, we need to tell Concourse the following about each step:

- What type of worker to run the task on (linux/windows/darwin)

- What container image to use (Linux only)

- What command to run inside the container

---

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

---

## Resources
Resources make Concourse tick and are the source of automation within all Concourse pipelines. Resources are how Concourse interacts with the outside world

Here's a short list of some things that resources can do:

- You want something to run every five minutes? [Time resource](https://github.com/concourse/time-resource/).
- You want to run tests on every new commit to the main branch? [Git resource](https://github.com/concourse/git-resource).
- Run unit tests on new PR's? [Github-PR resource](https://github.com/telia-oss/github-pr-resource).
- Fetch or push the latest image of your app? [Registry-image resource](https://github.com/concourse/registry-image-resource/)

The main goal of resources is to represent some external system or object in your pipeline. That external thing can then be used as a trigger for your Jobs or your Jobs can push back and modify the external system or object. It all depends on the resource you use and what features its author has implemented.

### Versions
Resources represent the external system or object to Concourse by emitting [versions](https://concourse-ci.org/config-basics.html#schema.version). When a new version is emitted by a resource, that is how Concourse knows to start trigger jobs

### Resource interface
A resource is a container image which contains 3 executables
 - `/opt/resource/check` - implicitly ran when a job contains a [get step](https://concourse-ci.org/get-step.html). Should return the latest version from the external system or object. Its responsibility is to find new versions.
 - `/opt/resource/in` - run in a [get step](https://concourse-ci.org/get-step.html). in is given a specific version (generated by a check or [put step](https://concourse-ci.org/put-step.html)) and retrieves the files representing that version from the external system or object.
 - `/opt/resource/out` - run in a [put step](https://concourse-ci.org/put-step.html). Generates a new version, usually based on some input generated by another step in the job. Depending on the resource, this may mean sending something to the external system. For the git resource, this means pushing commits to the external git repository.

 ### Get Steps
Use the [git resource](https://github.com/concourse/git-resource/) to trigger on every new commit to a repo

Define the resource
 ```yaml
resources:
- name: repo
   type: git
   source: 
     uri: https://github.com/markwillis75/concourse-examples
 ```

 Connect it to the job by adding a [`get step`](https://concourse-ci.org/get-step.html) to the job.  Set the [`trigger`](https://concourse-ci.org/get-step.html#schema.get.trigger) to true to cause the job whenever the resource emits a new version

 ```yaml
 jobs:
  - name: hello-world-job
    plan: 
    - get: repo     # link this job to the repo
      trigger: true # trigger this job whenever new versions are emitted
    - task: hello-world-task
    ...
    ...

 ```

 ### Get Steps and Inputs
 Get steps generate one `output` that can be consumed by tasks as an `input`
 Get steps always generate output artifacts based on their name

 To find out the structure of the artifact generated by a get step you will have to refer to the resource's documentation. The documentation for the [git resource](https://github.com/concourse/git-resource#in-clone-the-repository-at-the-given-ref) tells us that the in script will clone the git repository.


### Put Steps
[Put steps](https://concourse-ci.org/put-step.html) are generally for pushing some change to the external system or object the resource represents.