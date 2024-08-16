---
title: كيف تشغل docker-compose على Podman
categories: 
    - guides
date: 2021-09-30
image: thumbnail.jpeg
slug: docker-compose-on-podman
canonicalurl: https://discourse.aosus.org/t/topic/2140
tags:
    - docker
    - podman
    - حاويات
    - fedora
    - redhat
---
**هذا المقال كُتب على مجتمع أسس, كامل التعليقات [هنا](https://discourse.aosus.org/t/topic/2140)**

السلام عليكم ورحمة الله وبركاته.
في هذا الموضوع سوف اشرح كيفية تشغيل docker-compose مع podman وهو محرك حاويات جديد من منظمه Containers التي تدير مشروع Kubernetes وغيرها من مشاريع الحاويات.
تركيز podman هو على الامان, بحيث تم تصميمه لتشغيل الحاويات بدون الحاجة للوصول لصلاحيات الجذر. وتم تصميمه بطريقة مجزئه عكس دوكر.
معظم مطورين Podman هم من رد هات, وهو محرك الحاويات الرسمي لتوزيعات رد هات.

## تاكد من دعم شبكات Podman ل dns.
في بعض الانظمه قد لا تتوفر الحزم المطلوبه حتى تدعم شبكات podman ال dns, حتى يمكن للحاويات تواصل بين بعضها بدون الحاجه لعنوان ثابت.

```
sudo dnf install podman-plugins dnsmasq
sudo apt install podman-plugins dnsmasq
sudo zypper in podman-plugins dnsmasq
```
تشغيل dnsmasq (ملاحظه سوف ياخذ port 53 [بلاغ حول المشكلة](https://github.com/containers/podman/issues/9444))
```
systemctl enable dnsmasq
systemctl start dnsmasq
```

الان اي شبكة يتم انشائها ستدعم ميزه dnsname للحاويات.

## بدون وصول للجذر (Rootless)
هذه الطريقة المنصوح فيها استخدام podman. لانها تعزل الحاويات بشكل كامل بدون اي وصول لكامل صلاحيات النظام.

### تثبيت حزمة podman-docker و Pip.

ثبت حزمة podman-docker وهي حزمة مخصصه لمحاكاه دوكر في بودمان من اجل التوافقية.
اما لتثبيت docker-compose فيستحسن تثبيته من خلال pip, مديرحزم بايثون لانه يعطي اخر اصدار منه.

#### Redhat based
```
sudo dnf install podman podman-docker python3-pip
```

#### Debian based2

```
sudo apt install podman podman-docker python3-pip
```

#### Suse based
```
sudo zypper in podman podman-docker python3-pip
```

### تحديث pip  وتثبيت docker-compose
احيانا اصدار pip في التوزيعة قديم خاصه لو توزيعه ثباته مثلا دبيان او RHEL. سوف يحتاج pip للتحديث حتى تتمكن من تثبيت حزم حديثة
```
sudo python3 -m pip install --upgrade pip
```
ثبت docker-compose

```
python3 -m pip install docker-compose
```

### تشغيل podman socket
حزمة docker-compose تتواصل مع محرك دوكر عبر socket.
لذلك نحتاج ان نحاكي ال socket حتى يعمل docker-compose
```
systemctl --user enable podman.socket
systemctl --user start podman.socket
systemctl --user status podman.socket
```
اخر امر يجب ان يظهر لك ان socket في حاله Active(running)

موقع ال Socket سوف يكون مختلف عن المعتاد لدوكر لانها سوف تعمل بدون وصول لصلاحيات الجذر.

لذلك نقوم بتغيير موقع ال socket عبر هذا الامر:
```
export DOCKER_HOST=unix:///run/user/$UID/podman/podman.sock
```
من اجل حفظ التغيير اضف `DOCKER_HOST=unix:///run/user/$UID/podman/podman.sock` الى اخر ملف `.bash_profile` او `.zsh_profile`.

والان اصبح podman جاهز للتعامل مع docker-compose. فقط جهز ملف docker-compose.yml ثم نفذ امر `docker-compose up -d` وسوف يعمل معك.

في حاله كنت تستخدم ports اقل من 1000 ستحتاج لتغيير قواعد النظام للسماح بمداخل اقل من 1000 لبرامج بدون صلاحيات جذر.
**شرحت كيف تسمح ل ports اقل من 1000 لحاويات بدون جذر [هنا](https://discourse.aosus.org/t/topic/2141)**

## مع الوصول لصلاحيات الجذر

نفس الخطوات تقريبا لكن هناك اختلافين.

### تشغيل Podman.socket
سيتم تشغيل podman.socket في نفس مكان دوكر المعتاد لاننا سوف نعطيه وصول لصلاحيات الجذر.

```
sudo systemctl enable podman.socket
sudo systemctl start podman.socket
sudo systemctl status podman.socket
```
للتاكد من انها تعمل

```
sudo curl -H "Content-Type: application/json" --unix-socket /var/run/docker.sock http://localhost/_ping
```
يجب ان يظهر لك رد OK

### تثبيت docker-compose
```
sudo python3 -m pip install docker-compose
```
**ملاحظه** عند تثبيت الحزمه مع وصول للجذر سوف تحتاج لاستخدام sudo لتعمل معك.

والان بامكانك تشغيل اي مشروع docker-compose. فقط انشئ ملف docker-compse.yml ثم `sudo docker-compose up -d` وسوف يعمل المشروع معك.

## مصادر

https://fedoramagazine.org/use-docker-compose-with-podman-to-orchestrate-containers-on-fedora/

https://podman.io/getting-started/network#using-dns-in-container-networks