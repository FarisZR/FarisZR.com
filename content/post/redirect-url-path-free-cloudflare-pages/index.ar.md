---
title: أعادة توجيه مسار URL مجانا على Cloudflare Pages
categories: 
    - quick-snippets
tags:
    - HTTP redirects
    - Cloudflare pages
date: 2024-04-27
slug: redirect-url-path-free-cloudflare-pages
image: thumbnail.jpg
keywords: 
    - cloudflare
    - pages
    - redirect
---

إذا كنت تستخدم Cloudflare وتريد عمل اعادة توجيه لاي شيء, على الأغلب ستستخدم Redirect rules, لكن اذا تريد اعادة توجيه مسار كامل, مثلا `/old/*` الى `/new/*` ستحتاج لاستخدام `regex_replace`, التي تستوجب اشتراك PRO على الأقل.

لحسن الحظ مدونتي تستخدم Cloudflare Pages, التي تملك طريقتها الخاصة [لإعادة التوجية](https://developers.cloudflare.com/pages/configuration/redirects/), وتمكنك من عمل اعادة توجيه لمسارات دون الحاجه للاشتراك.

## استخدام مِلَفّ _redirects
تحتاج لإنشاء مِلَفّ اسمه `_redirects` يكون مضمون بالمجلد النهائي الذي يرفع الى Cloudflare pages, أنا بإضافته عبر GitHub Pages بعد انشاء الموقع, ثم رفعه.
·
### بستخدام Splats

أسهل طريقة هي باستخدام Splat, الsplat هو كل شيء يدرج تحت `*`

_redirects
```
/old/* /new/:splat
```

### بستخدام متغيرات
خيار اخر هي المتغيرات, تقوم بنفس الشيء تقريبا

لكن هناك بعض [الحدود](https://developers.cloudflare.com/pages/configuration/redirects/#placeholders) على استخدامها, لذلك استخدم Splats افضل.

_redirects
```
/movies/:title /media/:title
```

## المصادر

https://developers.cloudflare.com/pages/configuration/redirects/

https://community.cloudflare.com/t/transform-rule-replace-part-of-url/437813