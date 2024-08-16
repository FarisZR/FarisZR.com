---
title: طريقة زيادة سرعة مدير حزم DNF على Fedora
keywords:
  - Linux
  - Fedora
categories:
  - guides
date: 2021-06-22
image: dnf.png
slug: speedup-dnf
canonicalurl: https://discourse.aosus.org/t/topic/1976
tags:
  - fedora
  - linux
  - Redhat Dnf
summary: حل مشكلة سرعة التنزيل المعروفة في DNF على توزيعات Redhat
---

**هذا الموضوع كتب على [مجتمع أسس](https://discourse.aosus.org/t/topic/1976)**

من اشهر مشاكل DNF هي السرعة, سرعة التنزيل فيه بطيئه مع ان مرايا/خوادم فيدورا ممتازة
ومشكلة انه يحتاج لتحديث قائمة الحزم كل مرة تريد تثبيت حزمة جديدة

سبب مشكلة السرعة هو ان DNF اختياره لاسرع المرايا/الخوادم سيء, ويختار مرايا بعيدة عن موقعك

في هذا الشرح سوف اشرح كيف تسرع DNF بعدة طرق وما الفائدة منها.
الشرح على Fedora 36 ومن المفترض يطبق على التحديثات القادمة.

يفضل بعد عمل كل التعديلات عمل امر `sudo dnf clean all`

## تعديلات على اعدادات DNF

نقوم بتعديل اعدادات DNF
```
sudo nano /etc/dnf/dnf.conf
```
### أختيار اسرع مرايا/خادم
لاختيار أسرع مرايا تلقائيا(هذا الخيار بمفردة يرفع السرعة بشكل ملحوظ)
```
fastestmirror=true
```
### تفعيل الكاش
لتفعيل كاش يحتفظ بالحزم وقائمة الحزم ل 24 ساعة(سوف ياخذ مساحة من جهازك)
```
keepcache=true
```

بعد هذه التغييرات, سيقوم DNF باختيار اقرب خادم لديه(ليس دائما الافضل) و سيقوم بحفظ الحزم التي حملتها لمدة 24 ساعة.
## تحديد الدول
كما ذكرنا سابقا DNF سيء  في اختيار اسرع خوادم للتنزيل منها

الحل في ذلك هو تحديد الدول التي يمكن لDNF تنزيل منها فقط.
وذلك عن طريق تعديل ملفات المستودعات repo وتحديد الدول
هناك طريقتين للتعديل, بموجهه الاوامر و الواجهه الرسومية

ضع الدول الاقرب لك هنا, بشكل افتراضي سوف نستخدم de,fr,at وهي المانيا, فرنسا, النمسا وهي الاسرع للشرق الاوسط معظم الوقت.

### واجهه رسوميه
اولا نفتح Nautilus(اذا كانت واجهتك غير جنوم, ضع اسم متصفح الملفات) 
`sudo nautilus /etc/yum.repos.d`

ثم ادخل على ملفات المستودعات(`.repo`) وعدل رابط metalink لتضيف عليه الدول (ليس ضروري تعديل ملفات المستودعات التجريبية (testing) اذا لا تستخدمها)

اضيف `&country=de,fr,at` لاخر روابط ال meta
مثال:
قبل التعديل:
```
metalink=https://mirrors.fedoraproject.org/metalink?repo=fedora-$releasever&arch=$basearch
```
بعد التعديل:
```
metalink=https://mirrors.fedoraproject.org/metalink?repo=fedora-$releasever&arch=$basearch&country=de,fr,at
```

هذا التعديل يطبق على RPMfusion ايضا

### طريقة التيرمنال 
لنعرف ملفات المستودعات الموجودة لديك:
```
ls /etc/yum.repos.d
```
ثم عدل كل ملف مستودع(.repo), ملفات مستودعات تجريبية(testing).
مثال
`sudo nano /etc/yum.repos.d/fedora.repo`
وعدل رابط metalink لتضيف عليه الدول

اضيف `&country=de,fr,at` لاخر روابط ال meta
مثال:
قبل التعديل:
```
metalink=https://mirrors.fedoraproject.org/metalink?repo=fedora-$releasever&arch=$basearch
```
بعد التعديل:
```
metalink=https://mirrors.fedoraproject.org/metalink?repo=fedora-$releasever&arch=$basearch&country=de,fr,at
```
بامكانك اختيار اي دول القريبه لك, في الشرق الاوسط سلك الانترنت يتصل باوروبا اول شيء, لذلك هي اسرع شيء موجود.
في هذه الحالة اخترت المانيا و فرنسا و النمسا

هذا التعديل يطبق على RPMfusion ايضا

## استخدام Aria2 بدلا عن CURL (اختياري)
بشكل افتراضي DNF يستخدم curl للتحميل لكن aria2 يقدم سرعة تحميل افضل بشكل عام.

**هذه الخطوة اختياريه, وهي للاسف تؤثر على مظهر DNF, شخصيا لا ارى لها حاجة,**
**التعديلات السابقة على الاغلب سوف تعطيك سرعة الاتصال الكاملة.**

هذه الطريقه تسرع التحميل على معظم الشبكات, لكن على شبكات الهواتف مثلا قد تجعلها ابطئ
لذلك جربها وفي حالة انه لم تكن السرعة افضل, اخر الشرح طريقة الغاء التغييرات.

نثبت المتطلبات:
```
sudo dnf install patch aria2 git
```
ثم نقوم بنسخ المستودع الي فيه التعديل لنقل DNF من curl الى aria2
```
git clone https://github.com/daemonspudguy/DNF-Faster
```
ندخل الملف
```
cd DNF-Faster
```
ونقوم بعمل الباتش
```
sudo patch -p0 -d/ -b < dnf-faster.patch
```
#### لالغاء التعديل
ادخل لملف DNF-Faster ونفذ هذا الامر
```
sudo patch -p0 -d/ -b -R < dnf-faster.patch
```