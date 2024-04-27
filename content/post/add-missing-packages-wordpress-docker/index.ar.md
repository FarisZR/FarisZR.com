---
title: إضافة الحزم المفقودة إلى صور Docker لـ WordPress دون إعادة بنائها
categories:
  - مقتطفات-سريعة
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
  - دوكر
  - ووردبرس
keywords: 
  - docker
  - wordpress
summary: يوضح هذا الدليل المختصر كيفية إضافة الحزم اللينكس المفقودة إلى صور Docker الخاصة بـ WordPress بسهولة دون الحاجة إلى إعادة بنائها مع كل تحديث.
canonicalurl: https://discourse.aosus.org/t/topic/3103
---

هذا الموضوع متوفر على [مجتمع أسس](https://discourse.aosus.org/t/topic/3103)

اذا تشغل مواقع Wordpress داخل حاويات دوكر, قد تواجه احيانا بعض البلاغات من بعض الإضافات أن هناك حزم ناقصة, مثلا أضافه EWWW Image optimizer, التي كانت تطلب حزم محدده لم تكون موجوده بالصورة العادية.

الخِيار المعتاد في هذه الحالة هو اعادة بناء الصورة مع إضافة الحزم, لكن هذا يعني يجب عليك إعادة بناء الصورة يدويا مع كل تحديث, لكن هناك بديل, وهو تعديل امر تشغيل الصورة ليقوم بتنزيل الحزم, ثم تشغيل البرمجيات الأساسية:

## Wordpress:apache (latest)
صورة Wordpress الافتراضية تستخدم توزيعة Debian كأساس, لذلك سنستخدم مدير حزم apt, سنطلب منه تحديث قائمة الحزم المحلية, ثم تنزيل الحزم المطلوبة.
عند انتهائه من تثبيت الحزم سننتقل لتشغيل السكريبت المسؤول عن تشغيل برمجية WordPress.
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
نفس الطريقة مع الصورة الأساسية, فقط مع تغيير الأمر المستخدم لتشغيل البرمجية بعد إضافة الحزم.
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
كما يتضح من اسم الصورة, هي تستخدم توزيعة Alpine كأساس لها, وهذا يعني انه سنحتاج لاستخدام مدير حزم APK بدلا من APT.

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

والان ستضاف الحزم التي تحتاجها دون ان تحتاج لأعاده بناء الصورة يدويا!