---
title: removing secrets from a git repo using BFG
categories: 
    - today I learned
tags:
    - linux
    - docker
    - docker compose
    - linux containers
    - docker container
    - cloud computing
date: 2023-03-14
image: thumbnail.jpg
slug: docker-compose-gitops-github
keywords: 
    - github 
    - gitops
    - docker gitops
    - gitops without K8S
---

java -jar ~/Downloads/bfg-1.14.0.jar --no-blob-protection --replace-text ~/test.txt

