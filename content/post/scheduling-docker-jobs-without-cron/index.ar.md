---
title: جدولة حاويات Docker دون Cron
categories:
  - شروحات
tags:
  - docker
  - cron
  - container
date: 2023-12-27
slug: scheduling-docker-jobs-without-cron
image: thumbnail.jpg
canonicalurl: https://discourse.aosus.org/t/topic/3020
keywords:
  - cron
  - docker
  - حاوية
  - Ofelia
---

احيانا احتاج لجدولة شيء داخل دوكر, تشغيل حاويه كل فترة معينه, او تنفيذ امر داخل حاوية, و انا اتجنب هذا لانني اكره التعامل مع Cron, خاصة عندما يتعلق الموضوع بحاوية دوكر.

الحمد الله هناك [Ofelia](https://github.com/mcuadros/ofelia) مشروع حديث لجدوله الوظائف داخل الحاويات او حتى على النظام نفسه!

## الاستخدام مع الصفات(labels)
كنت اريد تشغيل سكربت ليقوم بتحديث ملف JSON تلقائيا [لخدمات أسس](https://github.com/aosus/aosus-serv1/blob/d861bd46c40fee2d6f54a62d158099027aa05189/nitter/docker-compose.yml#L29)
تركيبة الصفات لOfelia كالاتي:
`ofelia.<JOB_TYPE>.<JOB_NAME>.<JOB_PARAMETER>=<PARAMETER_VALUE>`

<JOB_TYPE> (نوع الوظيفة)
 - job-exec:

 ينفذ الأمر داخل الحاوية.
 - job-run: 

ينفذ الامر داخل حاوية جديدة مع صوره محدده.
 - job-local:

 تشغيل الأمر على النظام المستضيف ل Ofelia .
 - job-service-run:

 ينفذ الأمر داخل خدمة تعمل لمرة واحده, للاستخدام مع Swarm. 

الخيارات المتوفره حسب نوع الوظيفة [هنا](https://github.com/mcuadros/ofelia/blob/master/docs/jobs.md)

```yaml
services:
  ofelia:
    image: mcuadros/ofelia:latest
    command: daemon --docker
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - file.json:/src/file.json
      - update_json_file.sh:/update_json_file.sh:ro
    labels:
      ofelia.job-local.update_json_file.schedule: '@every 24h'
      ofelia.job-local.update_json_file.command: '/bin/sh -c  "apk add curl jq && sh /update_json_file.sh /src/file.json"'
```

مثال بسيط
نستخدم  `daemon --docker` كأمر لان إضافة `--docker` تسمح لنا باستخدام الصفات لإعداد Ofelia
 بامكانك ايضا استخدام ملف  [بصيغة INI](https://github.com/mcuadros/ofelia?tab=readme-ov-file#ini-style-config) لإعداد Ofelia.

بما ان برمجية Ofelia تعمل داخل الحاوية, وتحتاج وصول لDocker لتشغل الحاويات وتنفذ الأوامر بالاضافة لقراءه الصفات, نحتاج لان نعطي وصول لل docker socket, عبر عمل mount ل `/var/run/docker.sock`

في الصفات حددت وظيفة تعمل كل 24 ساعة, داخل Ofelia (لذلك استخدمت وظيفة نوع local) مع أمر `/bin/sh -c "apk add curl jq && sh /update_json_file.sh /src/file.json"`
احتجت ان اضيف `/bin/sh -c `حتى استطيع استخدام `&&` وتنفيذ اكثر من أمر.

والان جدولت وظيفة دون الحاجه لCron!