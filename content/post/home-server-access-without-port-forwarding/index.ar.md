---
title: اوصل لخادمك المنزلي من الانترنت دون الحاجة ل port forwarding مع Rathole
categories:
  - guides
keywords:
  - Tunneling
  - TCP tunneling
  - Rathole
  - Cloudflared
  - بروتوكول PROXY
  - خادم خلف CGNAT
  - CGNAT انترنت
  - خادم منزلي
  - port forwarding
  - الوصول إلى الخادم المنزلي
image: thumbnail.jpg
slug: home-server-access-without-port-forwarding
summary: إذا كنت ترغب في الوصول إلى خادمك المنزلي دون كشف عنوان IP الخاص بك، هناك العديد من الحلول المتاحة. يوفر Cloudflared طريقة سهلة وآمنة باستخدام WAF الخاص بهم، لكنه يعتمد على نموذج reverse proxy. بدلاً من ذلك، يمكن استخدام Rathole، وهو برنامج صغير مكتوب بلغة Rust، لإنشاء نفق لاتصالات TCP دون فك تشفير البيانات. يتضمن المقال خطوات تثبيت وإعداد Rathole على الخادم العام والمنزلي، بالإضافة إلى كيفية تفعيل بروتوكول PROXY لتمرير عنوان IP المتصل.
canonicalurl: https://discourse.aosus.org/t/topic/3239
date: 2024-05-28
---

اذا كنت تريد الوصول لخادمك المنزلي خارج الشبكة المنزلية دون عمل Port-forwarding, هناك عدة طرق.

بإمكانك استخدام Cloudflared, مجاني, وسهل التثبيت
بالإضافة انك تحصل على حماية من الWAF الخاص بهم.
لكن Cloudflare بالنهاية هو Reverse proxy, سيقوم بفك تشفير الاتصالات ثم اعادة تشفيرها لخادمك المنزلي.
وأيضا هناك حدود على استخداماته بناءا على شروط الاستخدام الخاصة بهم.

لكن هناك حل, [Rathole](https://github.com/rapiz1/rathole) برمجية صغيرة مكتوبة بلغة Rust, نفق لاتصالات TCP, دون فك تشفير اي شيء.
فقط تقوم بإعادة توجيه الاتصالات على منفذ معين على خادم الى خادمك المنزلي دون الحاجه لكشف خادمك مباشرة الى الإنترنت.

سوف استخدمه لعمل نفق لخادم Caddy, الذي استخدمه كreverse proxy في خادمي المنزلي.

## تثبيت Rathole
Rathole هو عباره عن حزمة صغيرة, نزل الحزمة المناسبة لمنصتك من [GitHub](https://github.com/rapiz1/rathole/releases).
بإمكانك أيضا تشغيلها داخل [حاوية](https://hub.docker.com/r/rapiz1/rathole).

سوف نثبته على الخادم العام أولا, ثم على الخادم المنزلي.
### docker-compose.yml على الخادم العام

```yaml
services:
  rathole:
    image: rapiz1/rathole
    stdin_open: true
    tty: true
    ports:
      - 8080:8080 # api port
      - 443:443 # forwarded port
    volumes:
      - ./rathole/config.toml:/app/config.toml
    command: --server /app/config.toml
```

## Rathole Config على الخادم العام

```toml
# server.toml
[server]
bind_addr = "0.0.0.0:8080" # `8080` هو المنفذ المستخدم للتواصل مع البرمجية

[server.services.home-proxy]
token = "your_very_secure_shared_token" # كلمة سر مشتركه بين خادمك المنزلي و خادم العام
bind_addr = "0.0.0.0:443" # المنفذ الموصل لخادمك المنزلي.
```

### docker-compose.yml للخادم المنزلي
```yaml
# بافتراض ان خادم الويب يستخدم شبكة دوكر مشتركة اسمها Web
networks:
  web:
    external: true

services:
  rathole:
    stdin_open: true
    tty: true
    volumes:
      - /home/lammah/rathole/config.toml:/app/config.toml
    image: rapiz1/rathole
    command: --client /app/config.toml
    networks:
      - web
```

### Rathole Config للخادم المنزلي
```toml
# client.toml
[client]
remote_addr = "yourservers-ip-or-domain:8080" # عنوان الخادم العام, المنفذ يجب ان يكون نفسه(8080)

[client.services.home-proxy]
token = "your_very_secure_shared_token" # كلمة السر المشتركة
local_addr = "caddy:443" # اسم الحاوية والمنفذ الذي سيستلم الاتصالات من الخادم العام.
```

فقط شغلهم الاثنين واذا تم اعداد كل شيء بشكل صحيح, سيتصلون ببعض وسيصبح بامكانك الوصول الى خادمك المنزلي عبر الخادم العام.

## عنوان IP المتصل
لكن هناك مشكلة, برمجية Rathole لن تظهر عنوان IP المتصل للتطبيق على الخادم المنزلي.
هذه سيسبب مشاكل اذا كنت تنوي حماية خادمك من محاولات تسجيل الدخول العشوائية او هجمات dos.

لكن لحسن الحظ هناك حل, بروتوكول PROXY.

## توصيل عنوان IP المتصل عبر بروتوكول Proxy مع Rathole

بروتوكول PROXY هو بروتوكول حديث نسبيا, يسمح للبرمجيات بعمل PROXY لاتصالات TCP مع إرسال عنوان المتصل الاصلي, دون الحاجة لمعالجه كثيره تبطئ سرعة الاتصال.

برمجية بروكسي / نفق TCP والبرمجية المستلمة يجب ان يدعموا البروتوكول حتى تتمكن من الحصول على عنوان المتصل الاصلي.

خادم ويب Caddy يدعم PROXY protocol [كمستلم](https://caddyserver.com/docs/json/apps/http/servers/listener_wrappers/proxy_protocol/) و [كبروكسي (مرسل)](https://caddyserver.com/docs/caddyfile/directives/reverse_proxy#the-http-transport).

هناك [طلب دمج](https://github.com/rapiz1/rathole/pull/352) ل rathole ليضيف دعم PROXY protocol.

بامكانك استخدام [صورة دوكر الخاصة بي](https://github.com/FZR-forks/rathole-proxy_protocol/pkgs/container/rathole-proxy_protocol) لإستخدام النسخه التي تدعم PROXY protocol او بناء الحُزْمَة محليا بنفسك.

استخدم نفس الاعدادات السابقة مع اضافة `enable_proxy_protocol = true` اسفل قسم `[server.services.home-proxy]`.

تأكد ان التطبيق المستلم يدعم بروتوكول PROXY وانه يستخدم الإعدادات المطلوبة لتفعيله, والآن من المفروض يظهر لك عنوان المتصل حتى بعد استخدام النفق او البروكسي.