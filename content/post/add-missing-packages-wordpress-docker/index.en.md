---
title: Add Missing Linux Packages into WordPress Docker Images Without Rebuilding
categories:
  - quick-snippets
date: 2024-02-29
image: thumbnail.jpg
slug: add-missing-packages-wordpress-docker
tags:
  - docker
  - wordpress
  - wordpress docker image
  - wordpress:latest
  - wordpress missing packages docker
  - EWWW image optimizer
keywords: 
  - docker
  - wordpress
summary: how to add missing Linux packages to WordPress Docker images without the need to rebuild them for each update.
---
If you use WordPress in docker, sometimes you may have some plugins complaining about missing libraries / packages, like for example EWWW image optimizer, that was complaining about missing image libraries.

You could of course just rebuild the image, but if you want to avoid that, you could modify the command that runs at start to install the packages you need, and then start the WordPress process:

## Wordpress:apache (latest)
the default image uses Debian as a base, so we modify the command to update packages lists and then install the packages we need, and after that WordPress Entrypoint should get triggered.
```diff
services:
  wordpress:
    image: wordpress:apache
    restart: always
+   command: sh -c "apt update && apt install -y --no-install-recommends xxxxxxx; docker-entrypoint.sh apache2-foreground"
    environment:
      WORDPRESS_DB_HOST: mariadb
      WORDPRESS_DB_USER: xxxx
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_PASSWORD: xxxxx
    volumes:
      - html:/var/www/html:rw
```

## Wordpress:fpm
Basically the same as the default Apache image, with just a different entrypoint command.
```diff
services:
  wordpress:
    image: wordpress:fpm-alpine
    restart: always
+   command: sh -c "apt update && apt install -y --no-install-recommends xxxxxxx; docker-entrypoint.sh php-fpm"
    environment:
      WORDPRESS_DB_HOST: mariadb
      WORDPRESS_DB_USER: xxxx
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_PASSWORD: xxxxx
    volumes:
      - html:/var/www/html:rw
```

## Wordpress:fpm-alpine
As the tag implies, this image uses Alpine as a base, here we have to use apk, the package manager for alpine.

```diff
services:
  wordpress:
    image: wordpress:fpm-alpine
    restart: always
+   command: sh -c "apk add xx xx xx; docker-entrypoint.sh php-fpm"
    environment:
      WORDPRESS_DB_HOST: mariadb
      WORDPRESS_DB_USER: xxxx
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_PASSWORD: xxxxx
    volumes:
      - html:/var/www/html:rw
```

And now you should have the packages you need without needing to rebuild the image manually on every update!