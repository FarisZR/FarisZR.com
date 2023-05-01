---
title: تثبيت Portainer, واجهه رسومية ل docker و docker compose لتسهيل تثبيت الحاويات
categories: 
    - شروحات
date: 2022-03-02
image: https://cdn-cf-discourse.aosus.org/original/2X/d/dd5d1e9eb6459913941d0a37a1261eaee07e76ab.jpeg
slug: portainer-setup
canonicalurl: https://discourse.aosus.org/t/topic/2346
tags:
    - حاويات
    - docker
    - docker-compose
    - containers
    - portainer
---
**هذا المقال يعتمد على ميزات خاصه بمجتمع أسس ليقرئ, اقرأه [هنا](https://discourse.aosus.org/t/topic/2346)**

السلام عليكم ورحمة الله وبركاتة.

![portainer-thumbnail|690x388](https://cdn-cf-discourse.aosus.org/original/2X/d/dd5d1e9eb6459913941d0a37a1261eaee07e76ab.jpeg)

اذا تريد ان تبدأ رحلتك في تثبيت واستضافه الخِدْمَات محليا على خادم منزلي او خادم مستأجر يجب عليك ان تستخدم دوكر.
دوكر هو برمجية تقوم بإنشاء حاوية مخصصه لكل برنامَج لتسهيل وتسريع تثبيت الخِدْمَات والبرمجيات وعزلها.

كامل التفاصيل عن عالم الحاويات و دوكر في هذا الموضوع:

https://discourse.aosus.org/t/topic/2332

برمجية [Portainer](https://portainer.io) تقوم بتسهيل التعامل مع docker وتثبيت الحزم, وذلك عبر واجهه رسومية بسيطة.
تعطيك اهم الخيارات مع ابقاء خِيار التعديل اليدوي ممكن.

## التثبيت
تأكد من تثبيت دوكر و docker compose انظر في هذا الشرح:

https://discourse.aosus.org/t/topic/2223

### تثبيت عبر docker-compose 

**تحذير: سيتم كشف port 9443 و 8000 للإنترنت اذا خادم متصل بالإنترنت مباشرة, ودوكر يتجاوز الجدار الناري داخل النظام**

اذا جهازك داخل شبكة منزلية (خلف مودم او جدار ناري منفصل) سوف يتم كشفها للشبكة المحلية فقط لان الجهاز ليس موصول للإنترنت مباشرة.
الأجهزة الموصولة للإنترنت مباشرة عادة هي الخوادم(Servers) و الخوادم السحابية كالVPS.

### docker-compose.yml
```
version: '3.3'

services:
    portainer:
        ports:
        - '9443:9443'
        - '8000:8000'
        container_name: portainer
        restart: always
        volumes:
            - '/var/run/docker.sock:/var/run/docker.sock'
            - './data:/data'
        image: portainer/portainer-ce:alpine
```

سنستخدم نسخة `alpine` وذلك لانها اصغر بمراحل, وتاق `alpine` يتبع اخر اصدار رسمي من portainer.

### تثبيت عبر امر docker 
**تحذير: سيتم كشف port 9443 و 8000 للإنترنت اذا الخادم متصل بالإنترنت مباشرة, ودوكر يتجاوز الجدار الناري داخل النظام**

اذا جهازك داخل شبكة منزلية (خلف مودم او جدار ناري منفصل) سوف يتم كشفها للشبكة المحلية فقط لان الجهاز ليس موصول للانترنت مباشرة.
الاجهزه الموصولة للانترنت مباشرة عادة هي السيرفرات و السيرفرات في السحابة
```
docker run -d -p 8000:8000 -p 9443:9443 --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    portainer/portainer-ce:alpine
```

سنستخدم نسخة `alpine` وذلك لانها اصغر بمراحل, وتاق `alpine` يتبع اخر اصدار رسمي من portainer.

## تحديث portainer

### Docker compose
في حاله ثبتت باستخدام compose, فكل الي عليك الذَّهاب للمجلد الذي يحتوي على مِلَفّ docker-compose.yml
ووضع هذا الامر:
```
docker compose pull && docker compose up -d
```

### أمر docker

#### وقف الحاوية القديمة
```
docker stop portainer
```
#### احذف الحاوية القديمة
```
docker rm portainer
```

#### نزل الحاوية الجديدة
```
docker pull portainer/portainer-ce:alpine
```
ثم ضع امر الذي شغلت فيه portainer أول مرة.


## اهم ميزات Portainer

### Containers
![image|690x366](https://cdn-cf-discourse.aosus.org/original/2X/5/5b9f86b2f2270fa2055f82ae1e604c44186345fc.png)




هنا يمكنك تثبيت حاويات بمفردها باستخدام خيارات رسومية بالكامل
بإمكانك تحديد ال volumes, المتغيرات, الشبكات, سياسة إعادة تشغيل الحاوية, وميزات النظام.
وغيرها من الإعدادات.

لكن هذه الصفحة لإنشاء حاوية واحده فقط, وهذا نادرا ما يحصل في دوكر, إذ دائما الخِدْمَات المتكاملة مثلا nextcloud تتكون من عدة حاويات
وهنا تأتي ميزه stacks

### Stacks
![image|690x391](https://cdn-cf-discourse.aosus.org/optimized/2X/7/7085051bd3f34e9d6993d662e64422f5575ee1a6_2_690x391.png)



مِيزة stacks تتيح لك تثبيت الخِدْمَات عبر ملفات docker-compose, وهو يمكن لك ربط الخِدْمَات مع بعضها وتحديد الاعتماديات, بالإضافة الى إنشاء شبكة للتواصل الحاويات ببعضها البعض.

وبإمكانك وضع مِلَفّ يدويا, او ربطة بمستودع Git قد توجد داخله ملفات جاهزه
![image|690x391](upload://fVTdPLJiz8whPhaajFzExcMCvfd.png)


### App Templates
![image|690x390](https://cdn-cf-discourse.aosus.org/optimized/2X/6/6dc4f3e95c0e27a3f946d9a3f44167e18e881b25_2_690x390.png)


خلال استخدامك ل portainer ستلاحظ كلمة template كثيرا, وهي قوالب للتطبيقات.
فتوجد قوالب جاهزه لبعض الحاويات, والمشاريع/stacks, وبإمكانك صنع template مخصص لأي حاوية او مشروع عبر وضع مِلَفّ compose او ربط مستودع git

![image|690x390](https://cdn-cf-discourse.aosus.org/optimized/2X/5/5e6b682a7fd70b8613d55373226c454774728788_2_690x390.png)


### Network
![image|690x388](https://cdn-cf-discourse.aosus.org/optimized/2X/d/d157d155e8e16932ee6809fd778b821ff8e2ed94_2_690x388.png)

من هنا بإمكانك أدارة كل الشبكات في دوكر, حذفها و رؤية كامل تفاصيل الشبكة.

وبإمكانك إنشاء شبكة جديدة, وإضافة حاويات لها وغيرها من الخيارات

![image|690x391](https://cdn-cf-discourse.aosus.org/optimized/2X/5/55cebb15c4908858e1f7e629fa2837d64136f35c_2_690x391.png)


### Volumes
أدارة كامل ال volumes في docker, وبإمكانك رؤية Volumes المستخدمة والغير مستخدمة وتفاصيلها

![image|690x388](https://cdn-cf-discourse.aosus.org/optimized/2X/c/c348709b96601b45b76aa54f58490967a3e56d92_2_690x388.png)

بالإضافة الى ذلك بإمكانك إنشاء volume جديد مع امكانية استخدام تخزين عبر الشبكة عبر بروتوكول NFS او CIFS

![image|690x388](https://cdn-cf-discourse.aosus.org/optimized/2X/a/aca6e73d3e8615acc609a8337ec93be5c77935e9_2_690x388.png)

## أدارة عدة أنظمة باستخدام Portainer
أحد أقوى ميزات Portainer هي أمكانية إدارة عدة تثبيتات من Docker او حتى Kubernetes من واجهه واحدة.
بدلا من دخول على ال shell لتعديل ملفات compose او رؤية حالة الحاويات, فقط اختار الخادم وسوف تظهر لك كل خيارات إدارته من Portainer


لتفعيل امكانية إدارة  دوكر من قبل Portainer نحتاج لتثبيت Edge Agent.


### التثبيت
أذهب الى Environment ثم اضغط على زر Add Environment

أختر Edge Agent

![image|690x388](https://cdn-cf-discourse.aosus.org/optimized/2X/3/311fcb5cb4113734af0afa2d188871ee00dd0347_2_690x388.png)


Name: أسم الخادم المثبت علية ال Edge Agent
Portainer server URL: عنوان واجهه Portainer

**دائما يجب عليك عدم كشف Portainer للإنترنت, كشف Portainer للإنترنت خطر أمني كبير.
أستخدم شبكة خاصة, او ثبت VPN للتواصل بين الأجهزة**

#### Compose
![image|690x390](image.png)

قم بنسخ قيم EDGE_KEY  ,EDGE_ID وضعها في الفراغات الأتية. وسيتم تحديثها في امر Compose
[wrap=placeholder key="EDGE_ID_VAR" description="EDGE_ID"][/wrap]
[wrap=placeholder key="EDGE_KEY_VAR" description="EDGE_KEY"][/wrap]
ضع اصدار portainer الظاهر لديك او أبقة فارغ ليستخدم اخر إصدار `latest` مثل امر تثبيت portainer في الموضوع.
[wrap=placeholder key="version" description="أصدار Portainer" default="latest"][/wrap]
```
version: '3.3'
services:
    agent:
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
            - /var/lib/docker/volumes:/var/lib/docker/volumes
            - /:/host
            - portainer_agent_data:/data
        restart: always
        environment:
            - EDGE=1
            - EDGE_ID==EDGE_ID_VAR=
            - EDGE_KEY==EDGE_KEY_VAR=
            - CAP_HOST_MANAGEMENT=1
            - EDGE_INSECURE_POLL=1
        container_name: portainer_edge_agent
        image: portainer/agent:=version=
```

#### docker run

![image|690x390](https://cdn-cf-discourse.aosus.org/optimized/2X/4/4568fa24f72926136e86dc20f585f59a21e55e12_2_690x390.png)

أختر Docker standalone ونفذ الأمر المكتوب.

## مصادر
https://docs.portainer.io/v/ce-2.11/start/install/server/docker/linux
https://docs.portainer.io/v/ce-2.11/advanced/edge-agent