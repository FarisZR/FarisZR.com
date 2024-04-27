---
title: Purge remote media cache in Matrix Synapse
categories: 
  - quick-snippets
date: 2023-09-28
image: thumbnail.jpg
slug: clear-remote-media-cache-matrix-synapse
aliases:
  - /en/clear-remote-media-cache-matrix-synapse/
tags:
  - matrix
  - synapse
  - Curl
  - matrix CURL request
  - generate matrix access token
  - purge remote media api
---

Using the Purge Remote Media API, we can send a POST request to delete all cached remote media before a certain time:

**Your account must be a server admin for this command to work**.

```bash
curl -X POST -H "Authorization: Bearer $ADMIN_ACCESS_TOKEN" "https://matrix.org/_synapse/admin/v1/purge_media_cache?before_ts=1672527600000"
```

`before_ts` takes time in UNIX epoch milliseconds, the one in the template is at 2023-01-01 00:00.
You can generate one [here] (https://currentmillis.com/)z

## Generating an access token
If you don't have an access token, you can also generate one using curl:

```bash
curl -XPOST \
  -d '{"type":"m.login.password", "user":"<userid>", "password":"<password>"}' \
  "https://matrix.org/_matrix/client/r0/login"
```

## Sources
https://matrix-org.github.io/synapse/latest/admin_api/media_admin_api.html#purge-remote-media-api
Bing AI.