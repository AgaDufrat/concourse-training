# Steps to set up

## Login and Alias

1. Download [fly CLI](https://concourse-ci.org/fly.html)
2. Most fly operations run against a particular Concourse instance, known as a *target*. Log into your Concourse, using the target name `training` and credentials concourse/course.

```
fly --target training login --concourse-url http://concourse.127.0.0.1.nip.io -u concourse -p course
```

Update the pipeline:
```
fly -t training set-pipeline --pipeline training --config ci/pipeline.yml
```

To trigger the `compile` job:

```
fly -t training trigger-job -j training/compile
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

## TODO: Externalise Compile and Test
Inline tasks are great for iterating on shell scripts, as we don’t need files to be available to Concourse - instead we can just fly set-pipeline and get faster feedback.

What are the downsides of inline tasks? Can you imagine why inline tasks might be dangerous for regulated industries?

Externalised tasks can be re-usable, and typically live in version control. Some Concourse users distribute libraries of common tasks that others can then compose pipelines from. Let’s tidy up our pipeline.yml by externalising our other tasks.

Create an external equivalent of compile

Execute the externalised compile task, and make fly download the compiled binary

Externalise test, then change pipeline.yml to use the new external tasks
>>>>>>> ddea214... TODO: Externalise Compile and Test
