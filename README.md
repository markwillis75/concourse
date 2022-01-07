## Tutorial
https://concourse-ci.org/getting-started.html

---

## Hello World Pipeline
The simplest Concourse pipeline is made of two objects:

- An unordered list of Jobs which contains...
- An ordered list of Steps

The order of the jobs does not matter. **The order of jobs does not define the structure of the pipeline**

Unlike jobs, the order of steps does matter! **Concourse will run the steps in the order that they are listed.**

### What is a Step?
A step is a single container running on a Concourse worker. Each step in a job plan runs in its own container. You can run anything you want inside the container (i.e. run my tests, run this bash script, build this image, etc.).

So if you have a job with five steps Concourse will create five containers, one for each step. Therefore, we need to tell Concourse the following about each step:

- What type of worker to run the task on (linux/windows/darwin)

- What container image to use (Linux only)

- What command to run inside the container