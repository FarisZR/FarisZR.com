---
title: تثبيت Squid proxy مع دوكر
categories:
  - شروحات
tags:
  - squid
  - docker
  - ubuntu
  - socks5
  - دوكر
  - بروكسي
  - بروكسي لينكس 
date: 2023-08-29
image: thumbnail.jpeg
slug: squid-proxy-docker-setup
---

انا استخدم البروكسي اكثر شيء لفصل التطبيقات عن اتصال VPN, لان للاسف صعب فصل التطبيقات عن الاتصال في لينكس.

وبما ان تثبيت اي شيء عبر دوكر هو تقريبا دائما اسرع طريقه لي, كتبت شرح بسيط لتثبيت squid بدوكر بستخدام صورة اوبونتو الرسمية, وهي الصورة الوحيدة لsquid التي تتلقى التحديثات.

## docker-compose.yml
```yaml
version: "3"
services:
  proxy:
    image: ubuntu/squid
    ports:
      - "3128:3128"
    environment:
      - TZ=UTC
    volumes:
      - ./squid.conf:/etc/squid/squid.conf
    configs:
      - source: squid
        target: /etc/squid/squid.conf

  configs:
    squid:
      file: ./squid.conf
```
استخدمت `configs` لتجاوز اي مشكلات صلاحيات قد تظهر لاحقا في ملفات الاعدادات.

## squid.conf
```
http_port 3128
http_access allow all
```

هذه الاعدادات مخصصه لبروكسي محلي, تسمح بجميع الاتصالات دون اي سجلات او كاش.


## المصادر
https://medium.com/setup-a-web-proxy-server-with-docker-d5c6942b5575

https://hub.docker.com/r/ubuntu/squid