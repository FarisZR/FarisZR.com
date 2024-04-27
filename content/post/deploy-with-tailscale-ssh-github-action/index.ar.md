---
title: كيفية تصدير مشاريع لخادم عبر Tailscale SSH وGitHub Actions
categories: 
    - مقتطفات سريعة
tags:
    - github
    - github actions
    - tailscale
    - تصدير
    - deployment
date: 2024-03-31
slug: deploy-with-tailscale-ssh-github-action
image: thumbnail.jpg
canonicalurl: https://discourse.aosus.org/t/topic/3174
keywords: 
    - tailscale
    - github 
    - ssh
---

اذا كنت تريد تصدير (deploy) مشروعك لخادمك دون كشف منفذ ssh او الحاجه لادارة مفاتيح ssh يدويا, بامكانك استخدام Tailscale ssh!

هذا شرح قصير لاستخدام مشروع [tailscale-ssh-deploy](https://github.com/FarisZR/tailscale-ssh-deploy) لتصدير ملفات مستودع بعد البناء الى خادمك عبر Tailscale SSH (ليس SSH العادي)

### تصدير موقع Static

كمثال سنصدر موقع Hugo للخادم.

```yaml
on:
  push:
jobs:
  build-and-deploy:
    name: build and deploy
    runs-on: ubuntu-latest
    steps:
      - name: checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build
        run: hugo --minify -v --gc --destination generated

        - name: Setup Tailscale
        uses: tailscale/github-action@v2
        with:
            hostname: Github-actions
            oauth-client-id: ${{ secrets.TS_OAUTH_CLIENT_ID }}
            oauth-secret: ${{ secrets.TS_OAUTH_SECRET }}
            tags: tag:ci

        - name: Start Deployment
        uses: FarisZR/tailscale-ssh-deploy@v1
        with:
            remote_host: USER@LOCAL-IPV4-ADDRESS-TAILSCALE
            directory: generated
            remote_destination: /var/www/html
            post_upload_command: systemctl restart nginx
```

الجزء المهم فعليا هو اخر خطوتين.
أولا نقوم بتشغيل Tailscale بستخدام, Oauth client ID و secret التي تجدها في صفحة الاعدادت الشبكة الخاصة بك (Tailnet) [تفاصيل اكثر](https://tailscale.com/kb/1215/oauth-clients).

بعد ذلك نقوم بتحديد الخادم و المستخدم.
تاكد من استخدام صيغة مثل المكتوبه بالمثال, واستخدام عنوان IP بدلا من hostname, لانني واجهت مشاكل سابقا مع استخدام hostname في Github Actions.

بعد ذلك حدد الملف الذي يجب رفعة في `directory`وأين يجب ان يُرفع في الخادم `remote_destination`

ايضا بامكانك استخدك `post_upload_command` لتنفيد امر بعد رفع الملفات, كمثال قمت باعادة تشغيل خادم Nginx عبر Systemctl.

والان اصبح لديك تصدير تلقائي لمشروعك دون الحاجه لادارة مفاتيح ssh او كشف الخادم!