---
title: أرسل رسالة ماتركس بستخدام CURL
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
    - ماتركس
    - بروتوكول ماتركس
---

هنا أمر لإرسال رسالة في بروتوكول [ماتركس](https://discourse.aosus.org/t/topic/2409) باستخدام CURL, و Access token

```
curl 'https://matrix.org/_matrix/client/r0/rooms/!xxxxxx:example.com/send/m.room.message/?access_token=xxxxxxxx' -X PUT --data '{"msgtype":"m.text","body":"hello world"}'
```

## الحصول على access token
إذا لم يكن لديك access token, بإمكانك الحصول عليها من Curl أيضا

```
curl -XPOST \
  -d '{"type":"m.login.password", "user":"<userid>", "password":"<password>"}' \
  "https://matrix.org/_matrix/client/r0/login"
```

## المصدر
https://news.ycombinator.com/item?id=19227859