---
title: تخزين S3 و CDN مجاني ل Discourse مع Cloudflare R2
categories:
  - مقتطفات-سريعة
tags:
  - discourse
  - cloudflare
  - تخزين S3
  - دسكورس
  - Cloudflare R2
date: 2023-05-01
image: thumbnail-ar.jpeg
slug: discourse-s3-object-storage-cdn-cloudflare-r2
canonicalurl: https://discourse.aosus.org/t/topic/2789
keywords:
  - Cloudflare R2
  - Cloudflare CDN
  - discourse
  - object storage
  - discourse cdn
  - discourse cloudflare R2 setup
---

**هذا المقال متوفر ايضا على مجتمع أسس [هنا](https://discourse.aosus.org/t/topic/2789)**

في شرح Discourse الرسمي حول [إعداد مقدمين خدمات تخزين متوافقين مع S3](https://meta.discourse.org/t/configure-an-s3-compatible-object-storage-provider-for-uploads/148916) يذكر ان Cloudflare R2 غير مدعوم لانه لا يتعامل مع ملفات مضغوطه مسبقا مع gzip بشكل صحيح.
لكن في تحديث [2023-03-16](https://developers.cloudflare.com/r2/reference/changelog/#2023-03-16) قامت Cloudflare بحل المشكلة, و [مجتمع أسس](https://aosus.org) يستخدم R2 الان ويعمل دون مشاكل. [discourse.aosus.org](https://discourse.aosus.org).
حتى [اختبارات مشكلة gzip قديمة](https://gist.github.com/csuhta/0001d1bb74200412bc1d7f9e11ec4ea5) اصبحت تعمل!

## الاعداد
أتبع كامل الخطوات قبل متغيرات البيئة في [الشرح الرسمي](https://meta.discourse.org/t/configure-an-s3-compatible-object-storage-provider-for-uploads/148916)

أيضا Cloudflare R2 مجاني حتى 10GB! (10 مليون عملية قراءة و 1 مليون عملية كتابة), لذلك مجتمعات مثل أسس على الأغلب لن تدفع اي شيء!

## متغييرات البيئة

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

أسس يستخدم هذه الاعدادات منذ شهر, دون اي مشاكل مع سرعة ممتازه بتحميل الصور و ملفات js وغيرها.
المشكلة الوحيده ان لا يمكن تحديد مقدم تخزين منفصل للنسخ الاحتياطيه, والتخزين لدى Cloudflare R2 ليس الارخص.