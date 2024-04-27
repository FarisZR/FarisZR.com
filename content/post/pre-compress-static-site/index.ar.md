---
title: كيف تضغط موقع ثابت الى صيغة Gzip و Brotli
categories:
  - مقتطفات-سريعة
tags:
  - Caddy
  - Hugo
  - Brotli
  - Gzip
  - static site
date: 2022-09-14
slug: compress-static-site-brotli-gzip
image: thumbnail.jpg
keywords:
  - gzip
  - brotli
summary: أوامر بسيطة لضغط موقع ثابت لصيغة Brotli و Gzip
---

إذا تستخدم خادم ويب مثل مثلا [Caddy](https://caddyserver.com),
قد تريد ضغط موقعك قبل رفعة للخادم, اذا تستخدم Caddy, فستحتاج إن تضغط الموقع يدويا لانه لا يدعم الضغط بنفسه لصيغة Brotli.

## Brotli
```
find public -type f \( -name '*.html' -o -name '*.js' -o -name '*.css' -o -name '*.xml' -o -name '*.svg' \) \
  -exec /bin/sh -c 'brotli -q 11 -o "$1.br" "$1"' /bin/sh {} \;
```

## Gzip
```
find public -type f \( -name '*.html' -o -name '*.js' -o -name '*.css' -o -name '*.xml' -o -name '*.svg' \) \
  -exec /bin/sh -c 'gzip -v -f -9 -c "$1" > "$1.gz"' /bin/sh {} \;
```

## المصدر
https://github.com/maruel/hugo-tidy/blob/main/docker-entrypoint.sh#L49