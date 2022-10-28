---
title: تثبيت Cloudflared على Docker بدون الحاجه للوحة التحكم
categories:
  - شروحات
slug: cloudflared-docker-setup
tags:
  - Cloudflare
  - Cloudflare Argo tunnel
  - Cloudflare Tunnel
  - Cloudflared
  - cloudflared docker setup
  - كلاودفلير
  - جسر كلاودفلير
  - دوكر
  - linux
date: 2022-10-28
summary: تثبيت جسر Cloudflared لوصل الخادم بخوادم Cloudflared دون كشف الخادم نفسه, على
  Docker دون جذر او مع جذر.
image: thumbnail.jpeg
---

عندما بحثت عن طريقه تثبيت Cloudflared على Docker, خاصة بدون استخادم لوحه تحكم Zero Trust(بسبب مشاكل الدفع) لم اجد شيء.
لذلك كتبت هذا الشرح البسيط عن عمليه التثبيت.

سنستخدم حاوية لكل موقع, لان برائي هذا اكثر تنظيم و اسهل بالادارة.

## docker-compose.yaml
```
version: '3.1'

services:
    cloudflared:
        image: cloudflare/cloudflared
        restart: always
        environment:
        - TUNNEL_ORIGIN_CERT=/etc/cloudflared/cert.pem
        - TUNNEL_EDGE_IP_VERSION=auto #use IPv4 and IPv6
        volumes:
        - ./cloudflared:/etc/cloudflared
        command: tunnel run $(TUNNEL_NAME)
```

## وصل Cloudflared بعنوانك

### أنشئ مجلد `cloudflared`
يجب عليك انشاء مجلد `./cloudflared` قبل تنفيذ اي امر, لان عند بدء الحاوية, سيتم انشاء المجلد لمستخدم الجذر, بينما حاوية cloudflared تعمل بمستخدم `nonroot`(id 65532).

```
mkdir ./cloudflared
```
#### بجذر
```
sudo chown -R 65532:65532 ./cloudflared
```

#### دون جذر
هذا يفترض ان ال Subuid و subgid الخاصه بك هي: `100000:65536`
```
sudo chown -R 165531:165531 ./cloudflared
```

#### User namespaces remapping
لا اعلم السبب, لكن UserNS يستخدم IDs مختلفه عن وضع بدون جذر, حتى لو كان يستخدم subuid/subgid.
```
sudo chown -R 165532:165532 ./cloudflared
```

### تسجيل دخول

```
docker run -v $PWD/cloudflared:/home/nonroot/.cloudflared cloudflare/cloudflared login
```
افتح رابط تسجيل الدخول في المتصفح وحدد اي عنوان تريد استخدامه في الجسر.
سيتم بعدها انشاء شهاداة الorigin لاستخدامها من قبل النفق.

## إنشاء نفق Cloudflared جديد

```
docker run -v $PWD/cloudflared:/etc/cloudflared cloudflare/cloudflared tunnel create ${TUNNEL_NAME}
```
غيرنا من`/home/nonroot/.cloudflared` الى `/etc/cloudflared` لان ملفات النفق موجوده بمجلد `/etc`
قمنا بتغيير موقع شهادة الOrigin من خلال متغير `TUNNEL_ORIGIN_CERT` في ملف docker-compose.yaml.

الان سيكون في مجلد `./cloudflared` ملف `cert.pem` و ملف `$TUNNEL_ID.json`.
انسخ معرف النفق(Tunnel ID) لان سنحتاجه لاحقا اثناء اعداد Cloudflared وتجهيز سجلات DNS.

## اعداد نفق Cloudflared

انشئ ملف `config.yml` داخل مجلد `./cloudflared`

هذه اعدادات بسيط لحاوية WordPress تعمل على منفذ 80 داخل نفس شبكة Docker التي فيها Cloudflared

```
tunnel: ${TUNNEL_ID}
credentials-file: /etc/cloudflared/${TUNNEL_ID}.json

ingress:
  - hostname: domain.tld
    service: http://wordpress:80
  - hostname: www.domain.tld # بدون هذا الاعداد سيظهر خطأ 404
    service: http://wordpress:80
  - service: http_status:404 # اي عنوان اخر سوف يصل إلى 404
```
بامكانك اضافات خيارات وتخصيص عمل النفق اكثر.

[هنا](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/local-management/) كامل التفاصيل عن اعدادات النفق.

لكن هذا سيكون كافي لمعظم الاستخدامات

## شغل الحاوية
تاكد ان حاوية Cloudflared بنفس شبكة الحاويات الاخرى اذا تستخدم الاسماء(hostnames) وشغل الحاوية!

```
docker compose up -d
```

## سجلات DNS

أضف سجل `CNAME` لكل العناوين التي ستستخدم النفق يوجه ل `${TUNNEL_ID}.cfargotunnel.com`, وتاكد من تشغيل وسيط(proxy) من Cloudflare (السحابه يجب ان تكون برتقالية).

## ملاحظة على Docker Rootless او UserNS

اذا وجد هذا الخطا:
```
WRN The user running cloudflared process has a GID (group ID) that is not within ping_group_range. You might need to add that user to a group within that range, or instead update the range to encompass a group the user is already in by modifying /proc/sys/net/ipv4/ping_group_range. Otherwise cloudflared will not be able to ping this network error="Group ID 65532 is not between ping group 65534 to 65534"
WRN ICMP proxy feature is disabled error="cannot create ICMPv4 proxy: Group ID 65532 is not between ping group 65534 to 65534 nor ICMPv6 proxy: socket: permission denied"

```
هذا غير مهم حاليا, لان Cloudflared لازال لا يدعم [ICMP من خلال QUIC](https://github.com/cloudflare/cloudflared/issues/726).
حاولت حلها عبر تطبيق اعداد `net.ipv4.ping_group_range = 0 2147483647`, لكن مازال التنبيه موجود.
لذلك تجاهله حاليا.

اذا لديك حل اكتبه بالتعليقات.