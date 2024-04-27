---
title: اضافه دعم IPv6 في Docker (حل مشكلة عدم ظهور عنوان الاتصال الصحيح)
categories: 
  - شروحات    
canonicalurl: https://discourse.aosus.org/t/topic/2936
tags:
  - لينكس
  - docker
  - docker compose
  - حاويات لينكس
  - docker container
  - حوسبة سحابية
date: 2023-09-26
image: thumbnail.jpeg
slug: docker-ipv6-setup-with-propagation
keywords: 
  - docker
  - IPv6
  - عنوان IPv6
---

نحن في أسس كنا نعاني من مشكلة, ان بعض الحسابات تظهر ان تم تسجيلها من عنوان شبكة دوكر.
وهو شيء غير ممكن, ويجب فعليا ان يظهر عنوان المصدر, و حتى تغيير خادم الويب لم يحل المشكلة, فما مصدرها؟

السبب كان اتصالات IPv6, لان عكس المتوقع, كشف الحاويه لجميع الشبكات و اضافه سجل DNS جديد ليس كافي لتفعيل دعم IPv6 في دوكر.

اذا لم تقم باي خطوات اضافية, جميع اتصالات IPv6 ستظهر وكائنها من Gateway شبكة Docker, وبمعظم الحالات العنوان يبدء ب172.

الحل لهذه المشكلة هو ان تكون كل شبكات Docker تدعم IPv6, من شبكة الجسر الى الشبكات الداخليه.

## تفعيل دعم IPv6 في دوكر

/etc/docker/daemon.json
```json
{
  "experimental": true,
  "ip6tables": true
}
```
هذا سيفعل  ipv6tables في دوكر, وهي تساعد بتوجيه المنافذ وعزل الشبكات, لكن يجب تفعيل الوضع التجريبي لتشغيلها. 

## تفعيل IPv6 على شبكة الجسر الافتراضية (Bridge Network)

تفعيل وضع IPv6 في دوكر من المفترض ان يكن كافيا, لكن لنتأكد ان جميع الشبكات تستخدم IPv6, 
نفعله في شبكة الجسر الافتراضية ايضا.

/etc/docker/daemon.json
```json
{
  "ipv6": true,
  "fixed-cidr-v6": "2001:db8:1::/64",
  "experimental": true,
  "ip6tables": true
}
```

## توزيع  IPv6 Subnets تلقائيا 

هذه الخطوة اختيارية, لكن بدونا ستضطر لتحديد الsubnet لكل شبكة يدويا داخل Docker - compose او بأمر `docker network create --ipv6 --subnet=xxx $NETWORK_NAME`.

أذا كنت ان تحصل على تجربة تلقائية شبيهه بطريقة عمل دوكر مع IPv4, فيجب عليك تحديد مجال العناوين المستخدم لتوزيع الSubnets للشبكات الجديدة تلقائيا.

/etc/docker/daemon.json
```json
{
  "ipv6": true,
  "fixed-cidr-v6": "2001:db8:1::/64",
  "experimental": true,
  "ip6tables": true,
  "default-address-pools": [
    { "base": "172.17.0.0/16", "size": 16 },
    { "base": "172.18.0.0/16", "size": 16 },
    { "base": "172.19.0.0/16", "size": 16 },
    { "base": "172.20.0.0/14", "size": 16 },
    { "base": "172.24.0.0/14", "size": 16 },
    { "base": "172.28.0.0/14", "size": 16 },
    { "base": "192.168.0.0/16", "size": 20 },
    { "base": "2001:db8::/104", "size": 112 }
  ]
}
```
يحبذ عدم استخدام `2001:db8::/104` كمجال حقيقي, لانه مخصص للشروحات فقط.

أذا كن تريد مجال عناوين للشبكات المحلية فقط, افتح موقع [simpledns.plus](https://simpledns.plus/private-ipv6) و استبدل اول مربعين بالعنوان الظاهر الموقع(عشوائي)

أعد تشغيل ال Docker Daemon:
```bash
sudo systemctl restart docker
```

## أضافة دعم IPv6 لحاويات أو مشاريع موجودة مسبقا
أعادة تشغيل ال Docker Daemon لن يكون كافي, خاصة عندما تكون لديك حاويات تستخدم شبكات Docker أخرى غير الافتراضية.

أذا كنت تستخدم Compose, فستضطر لحذف الحاويات:
```bash
docker compose down
```
ثم اضافة `enable_ipv6: true` تحت كل شبكة داخل المشروع.
اذا تستخدم فقط الشبكة الافتراضية, تقوم بتحديدها ثم اضافة الخيار لها:
```yaml
networks:
  default:
    enable_ipv6: true
```

شبكات Docker الخارجيه ستضطر لحذفها ثم اعادة انشائها مع خيار `--ipv6`
```bash
docker network rm $NETWORK && docker network create --ipv6 $NETWORK
```

الان من المفترض اصبح لديك دعم كامل لIPv6 مع ظهور مصدر الاتصال بشكل صحيح.

## تحقق من حل المشكلة

للتحقق ان المشكلة فعلا أنحلت, شغل حاوية Python موصولة بشبكة Docker التي تستخدمها, واتصل بها من جهاز خارجي عبر IPv6 (استخدم منفذ 8080)

```bash
docker run -p 8080:8080 --rm -it --network $NETWORK python:latest python3 -m http.server 8080 --bind ::
```

أذا مازال عنوان يظهر عنوان المصدر كعنوان IPv4 محلي, فجرب تشغيل نفس الحاوية لكن دون وصلها لشبكة أخرى:

```bash
docker run -p 8080:8080 --rm -it python:latest python3 -m http.server 8080 --bind ::
```

أذا ظهر العنوان بشكل صحيح, فيعني هذا ان الشبكة التي كنت تستخدمها لا تدعم IPv6, اذا مازال لا يظهر مصدر الاتصال, فتأكد ان التغييرات طُبقت!

## المصدر

https://docs.docker.com/config/daemon/ipv6/