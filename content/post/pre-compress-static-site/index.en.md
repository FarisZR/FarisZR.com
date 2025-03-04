---
title: How to compress static site to Brotli and Gzip
categories:
  - quick-snippets
tags:
  - Caddy
  - Hugo
  - Brotli
  - Gzip
  - static site
date: 2022-09-14
slug: compress-static-site-brotli-gzip
aliases:
  - /en/compress-static-site-brotli-gzip/
image: thumbnail.jpg
keywords:
  - gzip
  - brotli
description: Simple commands to compress a static site to brotli and Gzip
---

If you deploy your static site to a Web server like [Caddy](https://caddyserver.com), you might need to pre-compress it, especially if you want to use Brotli with Caddy as it's only supported for pre-compressed assets.


## Brotli
```
find www.new -type f \( -name '*.html' -o -name '*.js' -o -name '*.css' -o -name '*.xml' -o -name '*.svg' \) \
  -exec /bin/sh -c 'brotli -q 11 -o "$1.br" "$1"' /bin/sh {} \;
```

## Gzip
```
find www.new -type f \( -name '*.html' -o -name '*.js' -o -name '*.css' -o -name '*.xml' -o -name '*.svg' \) \
  -exec /bin/sh -c 'gzip -v -f -9 -c "$1" > "$1.gz"' /bin/sh {} \;
```

## Source
https://github.com/maruel/hugo-tidy/blob/main/docker-entrypoint.sh#L49