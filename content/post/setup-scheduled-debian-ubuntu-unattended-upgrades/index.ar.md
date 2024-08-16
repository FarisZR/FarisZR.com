---
title: جدولة تحديثات تلقائية للنظام و النواة على اوبونتو و دبيان
categories:
  - guides
tags:
  - debian
  - ubuntu
  - apt
  - unattended-upgrades
date: 2024-01-31
slug: setup-scheduled-debian-ubuntu-unattended-upgrades
image: thumbnail.jpg
draft: false
canonicalurl: https://discourse.aosus.org/t/topic/3058
keywords:
  - debian
  - ubuntu
  - unattended upgrades
  - apt-get
---

اذا تشغل كل الخدمات على الخادم داخل حاويات, ستكون احتمالية حصول مشاكل بسبب التحديثات التلقائية ضئيله جدا, بسبب فصل الحاويات البرمجيات عن النظام الأساسي.

تنزيل اداة التحديثات التلقائية unattended-upgrades
```bash

sudo apt update

sudo apt install unattended-upgrades

```
## إعداد Unattended-upgrades
بامكانك اختيار مصادر التحديثات كما يناسب

### السماح بالتحديثات الامنية فقط

هذا الاعداد الافتراضي للبرمجيه

/etc/apt/apt.conf.d/50unattended-upgrades
```conf
//      "origin=Debian,codename=${distro_codename}-updates";
//      "origin=Debian,codename=${distro_codename}-proposed-updates";
        "origin=Debian,codename=${distro_codename},label=Debian";
        "origin=Debian,codename=${distro_codename},label=Debian-Security";
        "origin=Debian,codename=${distro_codename}-security,label=Debian-Security";
```

### السماح بكل التحديثات

/etc/apt/apt.conf.d/50unattended-upgrades

```conf
        "origin=Debian,codename=${distro_codename}-updates";
//      "origin=Debian,codename=${distro_codename}-proposed-updates";
        "origin=Debian,codename=${distro_codename},label=Debian";
        "origin=Debian,codename=${distro_codename},label=Debian-Security";
        "origin=Debian,codename=${distro_codename}-security,label=Debian-Security";
```

### اضافة مستودعات اخرى

بامكانك ايضا أتمته التحديثات لحزم من مستودعات اخرى, مثلا مستودعات Docker الرسمية.

اذهب لمجلد  `/var/lib/apt/lists/` وجد اسم مستودع دوكر.
داخل الملف ستجد معلومات كالاتي
```
label: Docker CE
Origin: Docker
Suite: bookworm
```

المهم هو  **origin** و **suite**, لتحديد اسم المستودع و  اي اصدار منه مستخدم في النظام.

لنضيف مستودع دوكر للتحديثات, اضف سطر جديد تحت مستودعات التوزيعة في ملف  `/etc/apt/apt.conf.d/50unattended-upgrades`  بالصيغة الاتية:

`"<origin>:<archive>";`

بحالة دوكر سيكون `"Docker:bookworm";`, 
 لكن اذا كنت لا تريد ان تغير الاسم كل مرة تقوم بترقية النظام, بامكانك استخدام المتغير `${distro_codename}`.

```conf
"Docker:${distro_codename}";
```

والان ستشمل التحديثات التلقائية مستودعات دوكر ايضا!

## تفعيل التحديثات التلقائية للنواة

اذا لا تمانع من توقف الخادم قليلا بامكانك أتمته اعادة تشغيل النظام من اجل تحديث النواة.
لتفعيل اعادة التشغيل من اجل تحديث النواة غير
```conf

// Unattended-Upgrade::Automatic-Reboot "false

```

الى

```conf

Unattended-Upgrade::Automatic-Reboot "true";

```
وبإمكانك تخصيص وقت اعادة التشغيل الى ما يناسبك, افتراضيا الوقت هو الساعة ال02:00 بتوقيت الخادم.

**تأكد من تغيير الإعداد وحذف علامات التعليق قبل الإعداد (الفراغ و //)**

```conf

Unattended-Upgrade::Automatic-Reboot-Time "22:00";

```

## جدولة التحديثات

سنجدول خدمتين SystemD, عملية لجلب اخر التحديثات من المستودع, خدمة اخرى لتطبيق التحديثات على النظام.

### Apt update timer

```bash

sudo systemctl edit apt-daily.timer

```

ثم اضف الأتي في الفراغ بين التعليقات بالأزرق (افتراضا انك كنت تريد جلب اخر التحديثات كل يوم الساعة 00:55 بتوقيت الخادم)

```conf

[Timer]

OnCalendar

OnCalendar=00:55

RandomizedDelaySec=0

```

الان اعد تشغيل الخدمة و تاكد ان العداد يظهر الوقت الصحيح الباقي لتشغيلها.
```bash

sudo systemctl restart apt-daily.timer

sudo systemctl status apt-daily.timer

```

### Apt upgrade timer

نفس الطريقة, فقط تاكد ان هذه الخدمة تبدء بعد الخدمة السابقة, حتى تثبت اخر التحديثات المتوفرة.

```bash

sudo systemctl edit apt-daily-upgrade.timer

```

نفس الشيء مرة اخرى, اضف الاتي بين التعليقات باللون الازرق (افتراضا انك تريد تطبيق التحديثات الساعة 01:00)
```conf

[Timer]

OnCalendar

OnCalendar=01:00

RandomizedDelaySec=0

```

الان اعد تشغيل الخدمة وتاكد ان العداد يعمل بطريقة صحيحة
```bash

sudo systemctl restart apt-daily-upgrade.timer

sudo systemctl status apt-daily-upgrade.timer

```

والان اصبح لديك تحديثات تلقائيه لكل النظام!

## مصادر

https://askubuntu.com/questions/87849/how-to-enable-silent-automatic-updates-for-any-repository

https://wiki.debian.org/StableProposedUpdates

https://linuxiac.com/how-to-set-up-automatic-updates-on-debian/
