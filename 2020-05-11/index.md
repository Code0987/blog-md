---
date: 2020-05-11
title: Creating a GitHub Action for a Docker Image
subtitle: What are GitHub Actions anyway?
tags: docker, dockerfile, github, github-actions
cover: https://i.imgur.com/mbt9run.jpg
canonical: https://medium.com/@code0987/creating-a-github-action-for-a-docker-image-b00871844bdc
domain: blugreenspace.hashnode.dev
hideFromHashnodeCommunity: true
---

### What are GitHub Actions anyway?

Actions are one of the the smallest executable steps of a job, or you can say a job is composed of steps which can be actions or commands. You can create GitHub Actions based on docker images. [More details](https://github.com/features/actions).

### Creating a Github Action for an existing Dockerfile

Mainly three files are required, `Dockerfile`, `entrypoint.sh`, and `actions.yml`.

Let's start with creating a Dockerfile for our action, I'll use my Android CI Dockerfile. It's a simple Dockerfile, which includes all Android build-related tools, recent SDKs, Java, etc.

The Dockerfile for action uses the existing Docker image and extends it to include `entrypoint` script. The idea for the script is basically to pass the input from a step in workflow to runtime of the Docker in the Github environment.

#### `Dockerfile`
```
# Container image that runs your code
FROM code0987/android-ci:latest
# Copy your code to the filesystem path `/` of the container
COPY entrypoint.sh /
# Grant executable permission to entrypoint file
RUN ["chmod", "+x", "/entrypoint.sh"]
# Code file to execute when the docker container starts up (`entrypoint.sh`)
ENTRYPOINT ["/entrypoint.sh"]
```

#### `entrypoint.sh`

This script runs all the input passed from GitHub Workflow, as a set of commands in non-interactive `sh` spawned inside Docker runtime.

```
#!/bin/sh
set -eu
sh -c "$*"
```

Now let's define the action metadata so that GitHub can recognize this action. The specifics for Dockerfile based actions are, defining -

```
runs:
 using: 'docker'
 image: 'Dockerfile'
 args: …
```

#### `actions.yml`
```
name: 'Hello World'
description: 'Greet someone and record the time'
runs:
  using: 'docker'
  image: 'Dockerfile'
```

### Using above action in a GitHub Workflow
```
name: Android CI
on: [push]
jobs:
  android-ci:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    - name: "Android CI Github Action"
      uses: code0987/android-ci-github-action@master
      with:
        args: |
          chmod 755 gradlew 
          gradlew check
```
![Example](https://i.imgur.com/jg2M0y8.png)

---

_References_

GitHub repo for action used as example, [android-ci-github-action](https://github.com/Code0987/android-ci-github-action).

https://help.github.com/en/actions/building-actions/creating-a-docker-container-action
