---
title: Docker compose Gitops مع Github Actions
categories: 
    - شروحات
tags:
    - linux
    - docker
    - gitops
    - docker compose
    - linux containers
    - docker container
    - cloud computing
date: 2023-03-09
image: thumbnail.jpg
summary: عمل Docker compose Gitops بستخدام Github actions
slug: docker-compose-gitops-github
keywords: 
    - github-actions
    - github 
    - docker
    - gitops
    - docker gitops
    - gitops without K8S
---

السلام عليكم ورحمة الله وبركاته.
اغلب المهتمين بالحوسبة السحابية قد سمعو سابقا بمصطلح Gitops, وكيف انه يعدّ نقله نوعية بإدارة الحاويات, بدلا من ادارتها يدويا.
لكن عندما نبحث عن مصطلح Gitops نجد اغلب المصادر تستخدم عن Kubernetes.
لكن ماذا عن Compose؟ الذي مازال ابسط بمراحل ويكفي في العديد من الاستخدامات؟

لذلك قمت بعمل مشروع [docker-compose-gitops-action](https://github.com/FarisZR/docker-compose-gitops-action),  وهو Github Action يقوم بتشغيل الحاويات على الخادم عن بعد, ويسمح لك بإدارتها من داخل مستودع Git على Github.
وهو يدعم استخدامات متقدمه, مثل Swarm أو رفع ملفات مرافقه للحاويه. وحتى يمكن استخدامه على خادم منزلي دون فتح منافذ بواسطة [Tailscale SSH](https://tailscale.com/blog/tailscale-ssh/).

**هل Tailscale مفتوح المصدر؟**
واجهه وخادم Tailscale الرسمي ليس مفتوح المصدر, لكن كل تطبيقاتهم مفتوحة, والآن [اصبحُ يدعمون تطوير Headscale بشكل رسمي](https://tailscale.com/blog/opensource/), وهو خادم مهندس بشكل عكسي ليقدم كامل خِدْمَات Tailscale لكنه مفتوح المصدر و يمكن استضافته محليا, و [سيدعم Tailscale SSH قريبا](https://github.com/juanfont/headscale/issues/661)

## تجهيز الخادم
الaction يدعم وضعين للوصول, اما SSH عادي, او Tailscale SSH
فائدة Tailscale SSH الأساسية هي عدم الحاجه لفتح منافذ للوصول للخادم, بالإضافة لعدم الحاجه لإدارة مفاتيح يدويا.

ستحتاج لمستخدم له وصول لDocker **[دون Sudo](https://discourse.aosus.org/t/topic/2223#docker-sudo-27)**, قد يكون مستخدم غير المستخدم الأساسي على الخادم.

## تجهيز المستودع

هناك طريقتين لاستخدام الAction

## مجلد لكل مشروع
يعني سيكون هناك مجلد لكل مشروع/خدمة, داخله يكون Docker-compose.yml بالإضافة الى أي ملفات إضافية مثل ملفات للإعدادات, وسيتم تشغيل الAction في حاله حصول اي تعديل داخل هذا المجلد.
ولن تحتاج لتحديث كل شيء اذا تعديل لمشروع محدد فقط.

## مِلَفّ واحد لكل المستودع
هذا الخِيار يناسب الخوادم صغيره. 
مِلَفّ Compose واحد في نصف المستودع, يحتوي كل الحاويات والشبكات المستخدمة.

## استخدام الAction
أولا يجب اضافه معلومات الوصول للخادم, سواءا مفتاح SSH **بصيغة PEM** أو مفتاح Tailscale لاستخدامات مؤقته "ephemeral".


**ماهو مفتاح لاستخدامات مؤقته "ephemeral key"**
في حاله actions, عندما يتم تشغيله كل مره, يعدّ جهاز جديد وفقط يعمل مره واحد ثم يحذف.
لذلك يعتبر جهاز مؤقت "ephermal node" واستخدام مفتاح مؤقت, يجعل اي جهاز يستخدم مؤقت ويحذف بعد توقفه بفترة قصيرة.


```yaml
name: deploy-project

on:
  push:
    paths:
      - 'project/**'
      - '.github/**'
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    if: github.event_name != 'pull_request'
    steps:
      - name: checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Tailscale
        uses: tailscale/github-action@ce41a99162202a647a4b24c30c558a567b926709
        with:
          authkey: ${{ secrets.TAILSCALE_AUTHKEY }}
          hostname: Github-actions

      - uses: aosus/docker-compose-gitops-action@v1
        name: Remote Deployment
        with:
          remote_docker_host: root@100.107.201.124
          args: -p project up -d
          tailscale_ssh: true
          compose_file_path: docker-compose.yml
          upload_directory: true
          #post_upload_command: touch "upload_command.txt"
          docker_compose_directory: project
```
عند تعديل ملف داخل `.github` او تعديل شيء داخل `project ` سيتم تشغيل الaction.
بما أنني فعلت `upload_directory` سيقوم برفع الملفات في `project`  الى الخادم  ثم تشغيل الحاويات من `docker-compose.yml`.

اذا تريد استخدام SSH العادي, فقط استبدل `tailscale_ssh: true`:
```yaml
        ssh_public_key: ${{ secrets.SSH_KEY }}
        ssh_private_key: ${{ secrets.SSH_PRIVATE_KEY }}
```
وعين القيم في صفحه القيم السرية في Github.
اذا تحتاج لإصلاح الصلاحيات الخاصة بالملفات قبل تشغيل الحاوية, بإمكانك استخدام `post_upload_command`, لكن تغيير صلاحيات الملفات معظم الوقت يحتاج ل`sudo`.

والآن من المفترض ان يصبح كل شيء جاهز!, اصبح لديك Gitops مع Docker-compose!
