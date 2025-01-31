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

Thats why Document management Systems like Paperless-ngx exist.
and in this post i'm going to setup paperless-ngx with Nextcloud Integration and automated AI tagging and chatting.

## Docker-composez

For the docker compose setup i don't have to explain much, Paperless-ngx has template docker files ready to use in their Git repos:
https://github.com/paperless-ngx/paperless-ngx/tree/main/docker/compose

## setting up separate directories for each user
unless you want every user to be able to see each others documents, you need to make sure paperless seperates the resulting files into a per user dir.

you can do this with worksflows and storage Paths, or make it the default and just use workflows to assign documents to users when importing("consuming") docs.

### make Paperless-ngx use a seperate directory for each user 
Using the Placeholders details in the Paperless [docs](https://docs.paperless-ngx.com/advanced_usage/#filename-format-variables) you can define the default file formating in paperless using an envirnoment variable

```diff
services:
  app:
      ....
      environment:
        .....
+        PAPERLESS_FILENAME_FORMAT='{{ owner_username }}/{{ title }}'
```
this will keep the file name and will saved in an owner-specfic directory under the paperless archive directory.

### Assigning ownership based on subdirectory in the consumption directory

by default when importing from the consumption directory, the file is available to all users, but with workflows we can tell paperless to assign ownership based on the file dir when importing it.

#### Workflow trigger
![workflow trigger](workflow-trigger.png)

Trigger type: Consumption started
Filter Sources: Consumption folder

Filter path: `/usr/src/paperless/consume/$USERNAME`
This matches any file imported from a specifc directory

you can also use wildcards to match any file with the username in its directory by just writing `*$USERNAME*`

#### Workflow action
![workflow action](workflow-action.png)


the action should be to assign the ownership and rights to the user.

## Sources
https://www.madebyagents.com/blog/paperless-ngx-nextcloud-integration

https://github.com/icereed/paperless-gpt