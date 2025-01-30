---
title: Setup Contact and Calendar (dav)sync using Radicale and Docker compose.
categories:
  - guides
tags:
  - docker
  - compose
  - Linux
  - paperless
  - Nextcloud
  - AI llm
date: 2025-01-31
slug: go-paperless-with-ai-and-Nextcloud
summary: How to setup contact and calendar syncing across devices using Radicale server and Docker compose, with authentication and reverse proxy configuration.
image: thumbnail.jpg
keywords:
  - Paperless-ngx
  - ocr
  - llms
draft: true
---

Having a scanner is the first step of going paperless, however with time you realize that keeping stuff in order and finding documents quickly becomes a challenge, even when using full text search in cloud search.

Thats why Document management Systems like Paperless-ngx exist.7
and in this post i'm going to setup paperless-ngx with Nextcloud Integration and automated AI tagging and chatting.

## Docker-composez

For the docker compose setup i don't have to explain much, Paperless-ngx has template docker files ready to use in their Git repos:
https://github.com/paperless-ngx/paperless-ngx/tree/main/docker/compose



## Sources
https://www.madebyagents.com/blog/paperless-ngx-nextcloud-integration

https://github.com/icereed/paperless-gpt