---
title: How to send a matrix message using CURL and an access token
categories: 
    - quick-snippets
date: 2023-07-23
image: thumbnail.png
slug: send-matrix-message-curl
tags:
    - matrix
    - synapse
    - Curl
    - matrix CURL request
    - generate matrix access token
    - matrix message with curl
---

Here is a how you can send a simple matrix message with CURL:

```
curl 'https://matrix.org/_matrix/client/r0/rooms/!xxxxxx:example.com/send/m.room.message/?access_token=xxxxxxxx' -X PUT --data '{"msgtype":"m.text","body":"hello world"}'
```
## generate an access token
if you don't have an access token, you can generate one using curl too:

```
curl -XPOST \
  -d '{"type":"m.login.password", "user":"<userid>", "password":"<password>"}' \
  "https://matrix.org/_matrix/client/r0/login"
```

## Sources
https://news.ycombinator.com/item?id=19227859
https://webapps.stackexchange.com/questions/131056/how-to-get-an-access-token-for-element-riot-matrix