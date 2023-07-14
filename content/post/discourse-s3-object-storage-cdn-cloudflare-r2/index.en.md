---
title: Free Discourse S3 object storage and CDN with Cloudflare R2
categories:
  - quick-snippets
tags:
  - discourse
  - cloudflare
  - discourse object storage
  - object storage
  - Cloudflare R2
date: 2023-05-01
image: thumbnail.jpeg
slug: discourse-s3-object-storage-cdn-cloudflare-r2
keywords:
  - Cloudflare R2
  - Cloudflare CDN
  - discourse
  - object storage
  - discourse cdn
  - discourse cloudflare R2 setup
---

In the official [discourse S3-compatible object storage setup guide](https://meta.discourse.org/t/configure-an-s3-compatible-object-storage-provider-for-uploads/148916), it mentions that Cloudflare R2 is not supported, because it doesn't handle gzipped files correctly.
But Cloudflare fixed this in the [2023-03-16](https://developers.cloudflare.com/r2/reference/changelog/#2023-03-16) update, and I tested it, and it works great on [discourse.aosus.org](https://discourse.aosus.org).
Even [old broken gzip handling tests](https://gist.github.com/csuhta/0001d1bb74200412bc1d7f9e11ec4ea5) work now, so it looks like the problem is solved!

**EDIT: Discourse doesn't use the [CDN URL for direct downloads](https://meta.discourse.org/t/s3-cdn-url-not-being-used-on-non-image-uploads/175332), rather it uses the S3 API link, which doesn't work for Cloudflare R2, so any attachments not embedded in the post won't work**

And Cloudflare R2 is free for up to 10GB! (10,000,000 reads, 1,000,000 writes), so for your average discourse forum, its probably not going to cost you anything!

## setup
Follow any pre-environment step in [the official setup guide](https://meta.discourse.org/t/configure-an-s3-compatible-object-storage-provider-for-uploads/148916)

## environment setup

```
  DISCOURSE_USE_S3: true
  DISCOURSE_S3_REGION: "us-east-1" #alias to auto
  #DISCOURSE_S3_INSTALL_CORS_RULE: true #it should be supported
  DISCOURSE_S3_ENDPOINT: S3_API_URL
  DISCOURSE_S3_ACCESS_KEY_ID: xxx
  DISCOURSE_S3_SECRET_ACCESS_KEY: xxxx
  DISCOURSE_S3_CDN_URL: your cdn url
  DISCOURSE_S3_BUCKET: BUCKET_NAME
  DISCOURSE_S3_BACKUP_BUCKET: other-private-bucket #optional
  DISCOURSE_BACKUP_LOCATION: s3 #optional
```

I've been using it for a month now, and it's working great, no problems and fantastic speed when viewing images.
There is one caveat though, you can't configure a separate S3 host for backups and the CF R2 isn't the cheapest for storage.