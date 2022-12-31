---
title: مشاركة مجلد Samba يظهر بالشبكة المحلية بستخدام docker-compose
categories: 
    - شروحات
tags:
    - linux
    - smb
    - samba
    - samba local network share
    - مشاركة مجلد داخل شبكه محليه عبر SAMBA
    - docker
    - wsdd
    - avahi
    - docker compose
    - podman
    - linux containers
    - docker container
    - cloud computing
date: 2022-12-31
image: thumbnail.jpg
summary: 
slug: docker-smb-network-discovery
keywords: 
    - smb
    - samba 
    - docker
---

شرح بسيط عن كيفيه تجهيز SMB share يظهر في الشبكة المحلية, بسبب Docker اصبح الموضوع اسهل بكثير!

الشكر يعود ل [crazymax](https://crazymax.dev/) لعمله في تطوير حاويات جعلت الشرح ممكن.

## تجهيز SMB share
كما ذكرت سابقا, سوف نستخدم حاويه samba من crazymax, وينصح باستخدام وضع شبكه `host`.

### docker-compose.yml

غير `$TIMEZONE` للمنطقة الزمنية الخاصة بك, [هنا قائمة المناطق الزمنيه بصيغة TZ](https://wikipedia.org/wiki/List_of_tz_database_time_zones)

```yaml
## https://fariszr.com/docker-smb-network-discovery/

version: "3.5"

services:
  samba:
    image: crazymax/samba
    container_name: samba
    network_mode: host
    volumes:
      - ./smb:/data
      - ./downloads:/downloads
    environment:
      - "TZ=$TIMEZONE"
      - "SAMBA_LOG_LEVEL=0"
    restart: always
```

### الاعدادت

بعكس SMBd, الحاوية تستخدم صيغة YAML للاعدادت.

هنا اعدادات بسيطه لمشاركة مفتوحه.

```yaml
auth:
  - user: root
    group: root
    uid: 0
    gid: 0
    password: bar

global:
  - "force user = root"
  - "force group = root"

share:
  - name: downloads
    path: /downloads
    browsable: yes
    readonly: no
    guestok: yes
    veto: no
    recycle: yes 
```
قمت باجبار استخدام مستخدم `root` لان المِلَفّات مملوك له, لتجنب مشكلات الصلاحيات.

ضع مِلَفّ الاعدادات في نفس مِلَفّ `/data`, اي في حاله مِلَفّ compose في الأعلى الموقع سيكون: `./smb/config.yml`

بامكانك إضافة مستخدمين او مجلدات اكثر او غيرها, تفاصيل حول اعدادات docker-smb في [مستودع المشروع](https://github.com/crazy-max/docker-samba) على GitHub 

بعد تشغيل الحاوية, سيصبح لديك مجلد مشارك عبر SMB, لكن لن يمكنك اكتشافه من اجهزه اخرى.

## WSDD, SMB اكتشاف عبر الشبكة لنظام Windows

أضف هذا لمِلَفّ `docker-compose.yml`

```yaml
  wsdd:
    image: jonasped/wsdd
    network_mode: host
    environment:
      - 'HOSTNAME=$HOSTNAME'
    restart: always
```
غير `$HOSTNAME` إلى اسم الخادم.
وشغل الحاوية

```bash
docker compose up -d
```
ألان من المفترض ان يظهر المجلد داخل خادم تحت اسم `$HOSTNAME` في صفحه الشبكة على ويندوز (اذا كان استكشاف الأجهزة المحلية مفعل).

## Avahi, اكتشاف مجلد SMB في الشبكة المحلية لأنظمة Linux و MacOS

اضف هذا لمِلَفّ `docker-compose.yml`, وتأكد من تغيير `$HOSTNAME` إلى اسم الخادم.

```yaml
  avahi:
    image: ydkn/avahi
    hostname: $HOSTNAME
    network_mode: host
    volumes:
      - ./avahi-services:/etc/avahi/services:ro
    restart: always
```

### avahi-services/smb.service
أنشئ مِلَفّ `smb.service` بهذا المحتوى:

```xml
<?xml version="1.0" standalone='no'?>
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
 <name replace-wildcards="yes">%h</name>
 <service>
   <type>_smb._tcp</type>
   <port>445</port>
 </service>
</service-group>
```

أحفظ المِلَفّ و شغل الحاوية
```bash
docker compose up -d
```

والآن لديك مجلد SMB يمكن اكتشافه تقريبا من كل الأجهزة داخل الشبكة!


## المصدر

https://github.com/crazy-max/docker-samba/issues/1
