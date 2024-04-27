---
title: How to set "Cache-Control" Header for static files in Caddy
categories:
  - quick-snippets
tags:
  - Cache-Control
  - Caddy
  - Google page speed insights
  - Pagespeed
  - Serve static assets with an efficient cache policy
slug: set-cache-header-caddy-files
aliases:
  - /en/set-cache-header-caddy-files/
summary: A quick way to set the "Cache-Control" header in Caddy for static files
lastmod: 2022-09-30
image: thumbnail.jpg
---

If you are seeing the "Serve static assets with an efficient cache policy" notice in PageSpeed, you need to add the `Cache-Control` header to your website.

![](pagespeed.webp)

Here is how to do it if you are using Caddy.

## Cache-Control 
To set a `Cache-Control` header only for your static files like, .js, .jpg, and others, just add this to the relevant part of your Caddyfile(under the site block if the Caddyfile includes multiple websites)

```
@static {
  file
  path *.ico *.css *.js *.gif *.webp *.avif *.jpg *.jpeg *.png *.svg *.woff *.woff2
}
header @static Cache-Control max-age=5184000
```

The files are going to be cached for `5184000` seconds, so 60 days, you can change it if you want.

## Source
https://caddy.community/t/correct-way-to-set-expires-on-caddy-2/7914/13