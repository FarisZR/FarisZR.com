---
title: مسح كاش من خوادم أخرى في خادم Synapse  ل Matrix
categories: 
  - quick-snippets
date: 2023-09-28
image: thumbnail.jpg
slug: clear-remote-cache-matrix-synapse
tags:
  - matrix
  - synapse
  - Curl
  - matrix CURL request
  - ماتركس
  - بروتوكول ماتركس
  - purge remote media api
---

عبر Purge remote media API, بامكاننا حذف الملفات المخزنة محليا من خوادم اخرى قبل وقت معين:

**يجب ان يكون حسابك لديه صلاحيات ادارية لمسح الملفات**

```bash
curl -X POST -H "Authorization: Bearer $ADMIN_ACCESS_TOKEN" "https://matrix.org/_synapse/admin/v1/purge_media_cache?before_ts=1672527600000"
```

`before_ts` يتطلب الوقت بصيغة UNIX epoc بالملي ثانية.
الوقت بالمثال هو 2023-01-01 00:00.
بامكانك تحويل تاريخ لصيغة UNIX epoc [هنا](https://currentmillis.com/)

## الحصول على access token
إذا لم يكن لديك access token, بإمكانك الحصول عليها من Curl أيضا

```bash
curl -XPOST \
  -d '{"type":"m.login.password", "user":"<userid>", "password":"<password>"}' \
  "https://matrix.org/_matrix/client/r0/login"
```

## المصدر
https://matrix-org.github.io/synapse/latest/admin_api/media_admin_api.html#purge-remote-media-api
Bing AI.