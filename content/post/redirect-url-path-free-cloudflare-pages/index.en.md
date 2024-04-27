---
title: Redirect URL paths on Cloudflare pages for free
categories: 
    - quick-snippets
tags:
    - HTTP redirects
    - Cloudflare pages
date: 2024-04-27
slug: redirect-url-path-free-cloudflare-pages
image: thumbnail.jpg
keywords: 
    - cloudflare
    - pages
    - redirect
---

If you need to do an HTTP redirect for anything while using Cloudflare the first idea that comes to mind will be just to use Redirect Rules, however, if your goal is to redirect a path, like `/old/*` to `/new/*`, you actually can't do that in the free plan, as `regex_replace` requires at least a PRO subscription. 

Luckily this blog is hosted on Cloudflare pages, which has its own [redirect mechanism](https://developers.cloudflare.com/pages/configuration/redirects/), and you can do such redirects without needing a pro subscription.

## using the _redirects file
you just need to create file called _redirects,
which needs to be included in the final directory uploaded to Cloudflare pages. 
There are multiple site generator dependent ways to do this, I just add it manually in GitHub Actions after the site generation step.

### Using Splats

The easiest way to do this is just using splats, a Splat is everything that is matched when using `*` after a path.

_redirects
```
/old/* /new/:splat 301
```

### using Placeholders
Placeholders are another option, they do the same thing, but look cleaner I guess.

However, they have some specific [limitations](https://developers.cloudflare.com/pages/configuration/redirects/#placeholders), so stick to splats when possible.

_redirects
```
/movies/:title /media/:title 302
```

## sources

https://developers.cloudflare.com/pages/configuration/redirects/

https://community.cloudflare.com/t/transform-rule-replace-part-of-url/437813