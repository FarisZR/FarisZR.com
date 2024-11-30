---
title: تثبيت خادم Radicale لمزامنة جهات الاتصال والتقويم مع Docker.
categories:
  - guides
tags:
  - docker
  - دوكر
  - compose
  - Linux
  - caldav
date: 2024-11-30
canonicalurl: https://discourse.aosus.org/t/topic/3583
slug: contact-calendar-dav-sync-Radicale-docker
summary: شرح لطريقة تثبيت خادم Radicale لمزامنة جهات الاتصال والتقويم باستخدام Docker مع اعدادات الأمان والربط مع Caddy
image: thumbnail.jpg
keywords:
  - Radicale
  - davx
  - docker
---

كنت استخدم Etesync لمزامنة جهات الاتصال, لكن للاسف تطبيقهم على iOS لم يعد يعمل.

لحسن الحظ وجدت Radicale, خادم مفتوح المصدر يستخدم بروتوكول Caldav/cardav المعروف لمزامنة جهات الاتصال و التقويم و حتى قوائم المهام وحتى ايضا الملاحظات.
طبعا مقال من FarisZR لازم نستخدم فيه دوكر.

مشروع Radicale ليس لديه صورة دوكر رسمية, لكن [tomesquest](https://github.com/tomsquest/docker-radicale) وفر علينا الوقت وانشئ صورة دوكر جاهزة.

## docker-compose.yml
* استخدمت شبكة خارجية باسم Web لربط الخدمة بCaddy, بامكانك كشف المنفذ 5232 مباشرة اذا احتجت.

```yaml
networks:
  web:
    external: true

services:
  radicale:
    image: tomsquest/docker-radicale:latest
    container_name: radicale
    # ports:
    #   - 127.0.0.1:5232:5232
    init: true
    read_only: true
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - SETUID
      - SETGID
      - CHOWN
      - KILL
    deploy:
      resources:
        limits:
          memory: 256M
          pids: 50
    healthcheck:
      test: curl -f http://127.0.0.1:5232 || exit 1
      interval: 30s
      retries: 3
    restart: unless-stopped
    volumes:
      - ./data:/data
      - ./config:/config
    networks:
      - web
```

## config


هناك ملف Config مضمن بشكل افتراضي, لكن سنحتاج لتعديله لنضيف خيارات تسجيل الدخول وكلمات المرور.


أستخدم قالب م[لف config من Github](https://github.com/tomsquest/docker-radicale/blob/master/config), لتنشئ الملف محليا دخل مجلد `config`, قم بتعديله كما يناسبك, لكن تاكد من عدم تعديل اماكن الملفات.

### Auth

تاكد ان قسم Auth في مِلَفّ config يحتوي على هذه الإعدادات:

```ini
[auth]
type = htpasswd
htpasswd_filename = /config/users
htpasswd_encryption = bcrypt
```

### عمل hash لكلمة السر مع bcrypt
في اعدادات auth استخدمنا bcrypt كصيغة الhash لكلمة المرور.
لعمل هاش لكلمة المرور الخاص بك استخدم هذا الامر

```bash
echo -n "your password" | mkpasswd --method=bcrypt
```

ثم انشئ ملف `users` داخل مجلد `config` المربوط بالبرمجية.
محتواه يجب ان يكون مثل هكذا:

```htpasswd
john:$2a$10$l1Se4qIaRlfOnaC1pGt32uNe/Dr61r4JrZQCNnY.kTx2KgJ70GPSm
```

والان بامكانك تشغيل الخادم!

```bash
docker compose up -d
```

## Caddy reverse proxy
```json
sync.example.com {
	reverse_proxy radicale:5232
	encode zstd gzip
}
```
## استيراد البيانات
من تجربتي استيراد البيانات عبر واجهه الويب لم يعمل, قمت باستيراد جهات الاتصال على هاتفي ثم زامنتها وهكذا حللت المشكلة.