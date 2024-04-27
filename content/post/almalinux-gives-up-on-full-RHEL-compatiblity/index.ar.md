---
title: توزيعة Almalinux تعلن عن تخليها عن التوافقية التامة مع RHEL
categories: 
    - اخبار
date: 2023-07-14
image: thumbnail.png
slug: almalinux-gives-up-on-full-RHEL-compatiblity
canonicalurl: https://discourse.aosus.org/t/topic/2844
tags:
    - Red hat
    - RHEL
    - RHEL source code
    - RHEL open source
    - رد هات
    - مصدر رد هات
    - مجتمع أسس
    - Almalinux
    - الما لينكس
    - لينكس
---

**هذا المقال كُتب على مجتمع أسس [هنا](https://discourse.aosus.org/t/topic/2844)**


مازال عالم توزيعات لينكس يتعامل مع تاثيرات [قرار Red hat الأخير بإيقاف نشر مصدر توزيعة RHEL لغير المشتركين](https://discourse.aosus.org/t/topic/2819), اعلنت توزيعة Almalinux[ خططها المستقبلية سابقا](https://discourse.aosus.org/t/topic/2832), بنفس الوقت مع [توزيعات اخرى مثل Rocky Linux و Cloud Linux](https://discourse.aosus.org/t/topic/2832).
في هذا الأسبوع [أصدرت شركة Oracle ردها الساخر من Red hat](https://discourse.aosus.org/t/topic/2838), و اوضحت خططها المستقبلية المتحدية لقرار Red hat, وأكبر اعلان كان من [شركة SUSE عن اشتقاقها كود RHEL الأخير وإنشاء توزيعة جديده منه.](https://discourse.aosus.org/t/topic/2839)

الان بدئنا نحصل على تفاصيل اكثر من هذه التوزيعات, فمعظم الردود باستثناء Rocky لم تكن جدا مفصلة, انما تتكلم عن القيم و سلبيه قرار Red hat على المجتمع.


##  التخلي عن التوافقية الكاملة مع RHEL

أعلنت توزيعة Almalinux اليوم في مقال " The Future of AlmaLinux is Bright" عن تخليها عن التوافقية التامة مع توزيعة RHEL (ما يسمى أحيانا توافقية عله لعله 1:1 bug compatible).
https://almalinux.org/blog/future-of-almalinux/
التوزيعة ستستهدف التوافقية على مستوى ABI, لجعل التطبيقات التي تدعم RHEL تعمل على Almalinux أيضا, لكن ستبقى هناك اختلافات في العلل, وان الحلول قد يتم قبولها خارج جدول اصدارات RHEL.

## تغييرات بطريقة تطوير التوزيعة
الانتقال من كون التوزيعة نسخه الى توزيعة من منبع CentOS Stream سوف يؤثر على طريقة تطويرها.
أول التغييرات هي بَدْء ذكر مصدر الترقيع(Patch) في  التعليقات, لإعطاء شفافية اكثر.
أيضا ستبدأ التوزيعة بالطلب من مبلغين العلل اختبار العلة في CentOS stream, للتركيز على حل العلة في المصدر "Upstream".

## شراكات قادمة
لمحت كاتبة المقال benny Vasquez, رئيسه منظمة Almalinux عن وجود شراكات وتطويرات تحت العمل قد يعْلَنَ عنها خلال الأسابيع و الشهور القادمة.
قد تكون لها علاقة بدعوة Oracle Linux العامة لتوزيعة Almalinux و Rocky Linux للنسخ منها.
https://twitter.com/OracleLinux/status/1678501850868334592

## دعم Upstream
نهاية المقال توضح رئيسة المنظمة عن عدم تغيير موقف Almalinux من توزيعات CentOS Stream و Fedora, وانهم سيرفعون كل حلولهم لها.

**هل يؤثر هذا القرار على احتماليه استخدامك لتوزيعة Almalinux؟  هل ترى هناك توزيعات افضل بعد التغييرات الأخيرة؟**