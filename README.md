# Concourse CI training project

## Pre-requisites

This training project depends on a training environment that is published as a set of Kubernetes resources.
We need to install:
- Docker
- `kind` (Kubernetes-in-Docker) to start a Kubernetes cluster
- `kubectl` - used to interact with Kubernetes
- `fly` CLI to log into our Concourse instance

## Web UI

Visit Concourse in a web browser - http://concourse.127.0.0.1.nip.io

## Fly commands

- log into your Concourse instance `fly login --help`
Most fly operations run against a particular Concourse instance, known as a *target*. Log into your Concourse, using the target name `training` and credentials concourse/course.

```
fly --target training login --concourse-url http://concourse.127.0.0.1.nip.io - concourse -p course
```

- tell Concourse about your Pipeline and updates the pipeline

```
fly -t training set-pipeline --pipeline training --config ci/pipeline.yml

```

- unpause the pipeline

Pipelines are created in a paused state, meaning no resources will be checked and thus no jobs will start.
```
fly -t training unpause-pipeline -p training
```

- unpause a job

```
fly -t training unpause-job -j training/build-image
```

- trigger the `build-image` job:

```
fly -t training trigger-job -j training/build-image
```

## Debugging

Use `fly intercept` to establish a connection to the container that ran your job:
```
fly intercept -t training -j training/test
```
You can use:
- `pwd`, `whoami` and `ls` to explore the container
- `ctrl` + `d` to exit the container session
- `cat gopath/src/github.com/EngineerBetter/yml2env/test.sh` to look at the script

### Setting Task Environment Variables

Documentation [parmas: env-vars](https://concourse-ci.org/tasks.html#schema.task.params), [env-vars schema](https://concourse-ci.org/config-basics.html#schema.env-vars)
To set `$FIXTURE_LOCATION` to `fixtures/`
```
params:
  FIXTURE_LOCATION: ""
```
Remember to update the pipeline after every change.

## Chaining jobs

The order of jobs doesn't matter.
The order of tasks in the plan is important.

## Testing Without Committing

We want to find out if this task works without making a commit to version control, and without running the whole pipeline or changing the state of resources. Concourse allows us to do this in the exact same environment as it will run ‘for real’, meaning that we don’t have to try and replicate the tools and configuration of Concourse locally.

Use fly execute to make Concourse run your task in isolation. You will need to provide the local path of the yml2env code repository directory as an input.

```
fly execute -t training -c lint.yml -i cli-code=.
```

## Secrets and Parameters

Concourse can substitute placeholders in pipelines that are demarcated with double parentheses, eg ((password)).
Integrated credentials-manager such as CredHub or Vault are recommended.
When updating the pipeline we need to provide the correct argument to populate the parameters from vars.yml:

```
fly -t training set-pipeline --pipeline training --config ci/pipeline.yml --load-vars-from path/to/vars.yml
```
