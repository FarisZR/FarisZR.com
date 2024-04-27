---
title: Scheduling Docker jobs without cron
categories:
  - guides
tags:
  - docker
  - cron
  - container
date: 2023-12-27
slug: scheduling-docker-jobs-without-cron
aliases:
  - /en/scheduling-docker-jobs-without-cron/
image: thumbnail.jpg
draft: false
keywords:
  - cron
  - docker
  - container
  - Ofelia
---

Sometimes you need to automate some things in Docker containers, run a script or check a result online, and I honestly dread dealing with cron to do it.

That's where [Ofelia](https://github.com/mcuadros/ofelia) comes in! It's a modern scheduler built from the ground up for Docker, and it interacts directly with the Docker socket to run a container, run a script in a specific container, or even on the host!

## Usage with labels
I wanted to run a script to update data in a JSON file.
You can configure Ofelia in several ways, but I think by far the easiest is to just use docker labels with compose.

Labels are structured like this:
`ofelia.<JOB_TYPE>.<JOB_NAME>.<JOB_PARAMETER>=<PARAMETER_VALUE>`
<JOB_TYPE>:
> - job-exec: this job is executed inside of a running container.
> - job-run: runs a command inside of a new container, using a specific image.
> - job-local: runs the command inside of the host running ofelia.
> - job-service-run: runs the command inside a new "run-once" service, for running inside a swarm

Available parameters for each job type [here](https://github.com/mcuadros/ofelia/blob/master/docs/jobs.md)

```yaml
services:
  ofelia:
    image: mcuadros/ofelia:latest
    command: daemon --docker
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - file.json:/src/file.json
      - update_json_file.sh:/update_json_file.sh:ro
    labels:
      ofelia.job-local.update_json_file.schedule: '@every 24h'
      ofelia.job-local.update_json_file.command: '/bin/sh -c  "apk add curl jq && sh /update_json_file.sh /src/file.json"'
```

Let me explain the setup here.
First we have `daemon --docker` as command, because adding `--docker` allows us to use labels as configuration, you don't have to do that, you can also use an [INI structured](https://github.com/mcuadros/ofelia?tab=readme-ov-file#ini-style-config) file as config.

And since it's running inside a container, and it needs access to docker to start containers and run commands, in addition to reading container labels, we also mount the docker socket at `/var/run/docker.sock`.

Then in the labels I defined a job to run every 24h, and the command is `/bin/sh -c "apk add curl jq && sh /update_json_file.sh /src/file.json"` I had to manually open a shell with `/bin/sh -c` to use `&&` and run multiple commands.

And that's it! You now have an automated Docker container without having to fiddle with cron!