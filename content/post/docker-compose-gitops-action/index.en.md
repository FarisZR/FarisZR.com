---
title: Docker compose Gitops using Github Actions
categories:
  - guides
tags:
  - linux
  - smb
  - docker
  - docker compose
  - linux containers
  - docker container
  - cloud computing
date: 2023-03-09
image: thumbnail.jpg
description: Gitops with the simplicity of Docker-compose using Github actions
slug: docker-compose-gitops-github
aliases:
  - /en/docker-compose-gitops-github/
keywords:
  - github-actions
  - github
  - docker
  - gitops
  - docker gitops
  - gitops without K8S
lastmod: 2023-05-01
---
  
So you have heard about how good GitOps is, and how much better it is than having to manage your containers by hand. But almost every time GitOps is mentioned, it means using K8S. But what about the rest of us mere mortals who are still using Docker Compose?

That's why I started working on [docker-compose-gitops-action](https://github.com/FarisZR/docker-compose-gitops-action), a GitHub action to do GitOps with Docker compose!, it should cover most edge cases for Docker compose CI deployments, swarm, uploading sidecar files, post-upload shell commands, non-exposed servers like Homelabs and more.

## Server side setup

The action supports two access modes, SSH and Tailscale SSH.
Tailscale SSH is particularly useful for servers behind NAT or without an exposed SSH demon, allowing you to grant SSH access without manually generating keys or exposing the server.
Both Tailscale SSH and SSH require a user with access to the Docker socket **[i.e. without the need for sudo](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)**. This user can be different from the main user.

## Repo Setup
This action can be used in two ways

### Per-project directories
In this case, each project/service will have its own directory with sidecar files and docker-compose.yml.

This gives you more flexibility in configuring actions for deploying specific directories, so I think this is the way to go. And if there's a small update, you don't have to redeploy the whole repo.

### Single compose file
If the server is only running a single project, you can put the docker-compose.yml file in the root of the repo and have the action run on every push.

## Setting up the GitHub action

Before we can do anything, we need to set up the GitHub secrets for authentication.
Mainly SSH private key (PEM format) + public key, or an ephemeral Tailscale API key.

```yaml
name: deploy-project

on:
  push:
    paths:
      - 'project/**'
      - '.github/**'
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    if: github.event_name != 'pull_request'
    steps:
      - name: checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Tailscale
        uses: tailscale/github-action@ce41a99162202a647a4b24c30c558a567b926709
        with:
          authkey: ${{ secrets.TAILSCALE_AUTHKEY }}
          hostname: Github-actions

      - uses: aosus/docker-compose-gitops-action@v1
        name: Remote Deployment
        with:
          remote_docker_host: root@100.107.201.124
          args: -p project up -d
          tailscale_ssh: true
          compose_file_path: project/docker-compose.yml
          upload_directory: true
          #post_upload_command: touch "upload_command.txt"
          docker_compose_directory: project
```

Whenever the action file is edited or the project folder is changed on the main branch, this action is triggered.
I'm using Tailscale SSH, so I need to have the official Tailscale action as a pre-deploy step, to not have to specify any ssh keys or actually open any ports.

Just replace `tailscale_ssh` with this if you want to use pure ssh:

```yaml
        ssh_public_key: ${{ secrets.SSH_KEY }}
        ssh_private_key: ${{ secrets.SSH_PRIVATE_KEY }}
```
Be sure to set up these secrets before running the action!

I also enabled `upload_directory` and set `docker_compose_directory` as `project`.
The `project` directory will be uploaded to the server. This is useful if a container needs extra config files, and you don't want to rebuild a custom image with the config files on every image update.
It will also upload the docker-compose file since it is included in the directory.

If you need to change file permissions after uploading, you can use `post_upload_command` to `chmod` or `chown` files, although in most cases you will need to use sudo.

### managing secrets in compose files
But what about secrets, you don't want to upload your secret plain text into the repo right?, well you can just use GitHub secrets with environment variables!

Don't set a value for the variable in the compose file:
```
environment: 
  - VAR
```
set the variable in GitHub actions
```
- name: Start Deployment
  uses: FarisZR/docker-compose-gitops-action@v1
  env:
    MARIADB_PASSWORD: ${{ secrets.gnulinuxsa_wordpress_mariadb_password }}
  with:
    remote_docker_host: ${{ secrets.server_address }}
```

That should do the trick, just create an action for each of your projects, and you should have your very own Compose-powered Gitops workflow!

## Credits
Photo generated by Lexica AI, and no Lexica, you can't copyright [AI generated images](https://www.theverge.com/2022/2/21/22944335/us-copyright-office-reject-ai-generated-art-recent-entrance-to-paradise).