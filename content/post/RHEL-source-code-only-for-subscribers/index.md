---
title: شركة Red Hat توقف نشر مصدر RHEL لغير المشتركين
categories: 
    - شروحات
date: 2023-06-21
image: thumbnail.png
slug: RHEL-source-code-only-for-subscribers
canonicalurl: https://discourse.aosus.org/t/topic/2819
tags:
    - Red hat
    - RHEL
    - RHEL source code
    - RHEL open source
    - رد هات
    - مصدر رد هات
    - مجتمع أسس
---

**هذا المقال كُتب على مجتمع أسس [هنا](https://discourse.aosus.org/t/topic/2819)**

أعلنت Red hat اليوم توقف نشر مصدر توزيعة RHEL للعامة, وجعل مصدر CentOS Stream هو الوحيد المتوفر للعامة.

أي ان الشفرة المصدرية لتوزيعة RHEL ستصبح فقط للمشتركين, وهو مما سيصعب على توزيعات تنسخ RHEL مثل Oracle Linux, Rocky Linux, Almalinux وغيرها إبقاء التوافقية التامة معها.

وهذا القرار يتبع سلسلة من القرارات من الشركة التي لم تعجب المجتمع, بدئت من [إغلاق CentOS](https://arstechnica.com/gadgets/2020/12/centos-shifts-from-red-hat-unbranded-to-red-hat-beta/) في 2020, وحصره لنسخه CentOS Stream مع تحديثات متدحرجة, تشكل الأساس النهائي لإصدارات RHEL.
ولذلك تم تأسيس توزيعة Rocky linux و Almalinux, لتكون بديل لتوزيعة CentOS باستخدام نفس الكود المصدري لRHEL, مع بعض التعديلات لحذف العلامات التجارية التابعة لRed Hat.

## ماذا عن ترخيص GPL?
لينكس مفتوح المصدر بالكامل بسبب ترخيص GPL, الذي يفرض نشر المصدر مع نشر نسخه معدلة من البرمجية.
لكن هل هذا التغيير يكسر الترخيص؟ فعليا لا, بسبب ان الترخيص يطلب توفر المصدر للمستخدم, وفي هذه الحالة كل من سيستخدم توزيعة RHEL سيكون مشترك, لذلك سيكون له وصول للمصدر ايضا.
ويذكر طبعا ان ليست كل الحزم تحت ترخيص GPL, الكثير منها تحت ترخيص Apache2 او MIT, وهذه تراخيص لا تشترط وجود المصدر للنسخ المعدلة.

لكن مالمانع من اشتراك توزيعات مثل Rocky بRHEL ثم نسخ المصدر؟
ربما قد يتم حظر حساباتهم, لكن لا يمكن منعهم من الوصول للمصدر, لأنه يعدّ خرق للترخيص.

توزيعة Rockylinux و Almalinux اصدرو تصريحات حول هذا القرار.
بشكل عام لا توجد اي طرق واضحه لتجاوز التغيير.
https://rockylinux.org/news/2023-06-22-press-release/
https://almalinux.org/blog/impact-of-rhel-changes/


## RHEL مجاني
هناك بديل اخر, وهو استخدام الاشتراك المجاني لRHEL المخصص للمطورين.
يسمح باستخدام 16 جهاز, لكن يجب تجديد الترخيص سنويا.

المشكلة أن العديد من شركات الاستضافة لا تقدم صور RHEL, وهذا يجعل استخدامه على السحابة معقد.
https://developers.redhat.com/articles/faqs-no-cost-red-hat-enterprise-linux#general

## مصادر
https://www.redhat.com/en/blog/furthering-evolution-centos-stream
https://www.phoronix.com/news/Red-Hat-CentOS-Stream-Sources