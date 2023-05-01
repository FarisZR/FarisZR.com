---
categories:
  - شروحات
title: كيفية تثبيت Docker و Docker-compose
date: 2021-12-17
tags:
  - docker
  - docker compose
  - تثبيت دوكر
  - تثبيت دوكر لينكس
  - مجتمع أسس
lastmod: 2022-09-01
image: docker-thumbnail.jpg
slug: install-docker-compose
summary: شرح كيفية تثبيت Docker و Docker-compose على توزيعات لينكس الشهيرة
canonicalurl: https://discourse.aosus.org/t/topic/2223
keywords:
  - docker
  - docker compose
  - Docker-compose
  - دوكر
  - مجتمع أسس
---
**هذا الموضوع كتب على [مجتمع أسس](https://discourse.aosus.org/t/topic/2223)**

السلام عليكم ورحمة الله وبركاتة.
اليوم سوف اشرح كيفية تثبيت Docker على لينكس. وهذا شرح مبدئي لسلسله عن Docker سوف اكتبها.

## ماهو Docker ؟
دوكر هو أشهر محرك حاويات في الساحة, وهو بدء مفهوم السحابة الحالي.
بامكانك التفكير بDocker كانه نظام وهمي, لكن بدون اخذ موارد كثيرة, وذلك عبر استخدام نفس كيرنل النظام الاساسي و جعل النظام داخل الحاويه بسيط جدا ولا يحتوي الا على الاشياء الضرورية.

الفائدة من هذه الطريقة انه بامكانك تشغيل خدمات بثواني. بدلا من تثبيت قاعدة بيانات postgres و انشاء مستخدم وتشغيلها يدويا.
فقط نزل حاويتهم الرسميه, ضع الاسم وكلمة المرور وقاعدة البيانات وانتهى. لا تحتاج ان تثبت اي شيء على النظام الاساسي ولا تحتاج ان تثبت اي حزم محليا او اي اعتماديات. كل شيء معزول داخل الحاوية.
ويكون سهل جدا نقل و تكرار البرمجيه على عدة اجهزة. لانك تشحن كامل البرنامج مع كل اعتمادياتة. لذلك السحابة كلها تعتمد على الحاويات.

## ما هو Docker Compose ?
سابقا, كانت الطريقة المعروفة لتشغيل الحاويات هي اوامر طويله.
ولتحديثها يجب عليه ان توقف الحاوية, تحذفها, ثم تنزل نسخه احدث منها وتضع نفس الامر الطويل لتشغيلها.
Docker compose هو معيار مبني على yaml لعمل ملف يختصر كل هذه الاشياء.
ملف يوجد به كل الحاويات المستخدمة, واعداداتها.
بحيث بامر واحد تحذفها,تشغلها أو تحدثها بدون الحاجة لاوامر معقدة.
فقط امر `docker compose up -d` لتشغيل الحاويات و `docker compsoe pull` لتنزيل صور احدث من الحاويات وغيرها من الاوامر.

لذلك استخدام Docker دون compose مزعج فعليا. لدرجة ان احد الاشخاص عمل موقع يقوم بتحويل اوامر Docker الطويلة الى ملف docker compose, يعطيك ملف يعمل معظم الوقت.
https://www.composerize.com

## الانظمة المدعومة بشكل رسمي من Docker.
قائمة محدثه في تاريخ كتابة الموضوع.
| التوزيعة | arm64 / aarch64 | x86_64 / amd64 | arm (32-bit) | s390x |
|---|---|---|---| ---|
| [CentOS](#centos-7) | [x] | [x] |  |   |
| [Debian](#debian-11) | [x] | [x] | [x] |   |
| [Fedora](#fedora-16) | [x] | [x] |  |
| RHEL |  |  |  |  [x] |
| SLES |  |  |  |  [x] |
| [Ubuntu](#ubuntu-20) | [x] | [x] | [x] |  [x] |
| Binaries | [x] | [x] | [x] |  [x] |

لن اشرح RHEL و SLES او binaries لانك لو تستخدمها المفروض تعرف تتبع الشرح الرسمي :sweat_smile:. 

اختيار التوزيعه لا يفرق كثيرا في تجربة Docker قد يساعد وجود كيرنل حديثه على دعم ميزات متقدمة. بشكل عام أختر الذي لديك خبرة فيه.
**التوزيعات المبنيه على التوزيعات المدعومة مثل linux mint او  Rocky Linux وغيرها تتبع نفس اوامر التوزيعه الام**

هناك توزيعات تقدم حزم غير رسميه, مثل Opensuse او حتى التوزيعات المدعومة نفسها قد تقدم حزم خاصة لها في مستودعاتها.
هذه الحزم غير مدعومة من Docker وقد تكون متاخرة في التحديثات او الميزات.

## تثبيت Docker engine
معظم الوقت ما تشير له بدوكر, هو Docker engine ويعني محرك دوكر.
وهي الحزمة الاساسية لدوكر وداخلها كل البرمجية.

هناك طريقتين لتثبيت Docker.

### سكربت تلقائي التثبيت لكل التوزيعات المدعومة
هذا سكربت مجهز مسبقا من دوكر, كل الي عليك تحميله وتشغيلة وسوف يتعرف على التوزيعه ويضيف المستودعات ويحمل دوكر والحزم المساعدة و Docker compose تلقائيا.

**شركة Docker لاتنصح باستخدام هذا السكربت في سيرفر في وضع الانتاج(production)**

اذا لديك تحفظات امنية حول تشغيل سكربت بصلاحيات الجذر عبر الانترنت. ربما الطريقة الاخرى تناسبك اكثر.

لتثبيت دوكر عبر سكربت غير تفاعلي الرسمي.
```
 curl -fsSL https://get.docker.com -o get-docker.sh
 sudo sh get-docker.sh
```

### اضافة المستودعات والتثبيت عبر مدير الحزم

#### CentOS
- تاكد من عدم تعطيل مستودع `centos-extras` 
- محرك تخزين `overlay2' هو المنصوح به

##### أضف المستودع
```
sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

##### تثيبت Docker Engine
```
sudo yum install docker-ce docker-ce-cli containerd.io
```

##### شغل Docker
```
sudo systemctl start docker
```
لجعل docker يعمل عند الاقلاع
```
sudo systemctl enable docker
```

عندما يظهر طلب موافقه على مفتاح GPG, تاكد ان البصمة(Fingerprint) توافق `060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35` اذا توافقه اقبل به.

#### Debian

##### نثبت الحزم الضرورية و اضافه دعم https
```
 sudo apt-get update
 sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
#####  نضيف مفتاح GPG الرسمي لدوكر

```
 curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

```

#####  نضيف المستودع
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

##### تثبيت Docker Engine
```
 sudo apt-get update
 sudo apt-get install docker-ce docker-ce-cli containerd.io
```
#### Fedora

##### أضف المستودع
```
 sudo dnf -y install dnf-plugins-core

sudo dnf config-manager \
    --add-repo \
    https://download.docker.com/linux/fedora/docker-ce.repo
```

##### تثيبت Docker Engine
```
sudo dnf install docker-ce docker-ce-cli containerd.io
```

عندما يظهر طلب موافقه على مفتاح GPG, تاكد ان البصمة(Fingerprint) توافق `060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35` اذا توافقه اقبل به.


##### شغل Docker
```
sudo systemctl start docker
```
لجعل docker يعمل عند الاقلاع
```
sudo systemctl enable docker
```

عندما يظهر طلب موافقه على مفتاح GPG, تاكد ان البصمة(Fingerprint) توافق `060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35` اذا توافقه اقبل به.

#### Ubuntu

##### نثبت الحزم الضرورية و اضافه دعم https
```
 sudo apt-get update
 sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
#####  نضيف مفتاح GPG الرسمي لدوكر

```
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

```

#####  نضيف المستودع
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

##### تثبيت Docker Engine
```
 sudo apt-get update
 sudo apt-get install docker-ce docker-ce-cli containerd.io
```

## بعد التثبيت

### اختبار دوكر
```
sudo docker run hello-world
```
### استخدام Docker بدون Sudo.
Docker يحتاج لصلاحيات الجذر ليعمل بشكل افتراضي, لذلك مع كل امر سوف تحتاج sudo.

بامكانك تجاوز هذا عبر اضافه المستخدم لمجموعة Docker, اضافه المستخدم لهذه المجموعه سوف يعطيه صلاحيه الجذر على docker لذلك لا تضيفه الا اذا كنت تثق بالمستخدم.

#### انشاء مجموعة Docker
```
 sudo groupadd docker
```
#### اضف نفسك للمجموعة
```
 sudo usermod -aG docker $USER
```

### تشغيل دوكر عند الاقلاع.
في توزيعات دبيان و أوبونتو يتم تشغيل دوكر بشكل تلقائي عند الاقلاع.
على التوزيعات الاخرى هذه الطريقة.
```
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

### اختبر تشغيل دوكر بدون sudo
```
 docker run hello-world
```

## تثبيت Docker Compose

اذا لم تستخدم السكربت المساعد, بامكانك تثبيت Compose عبر تثبيت حزمة `docker-compose-plugin`

### Debian/Ubuntu
```
sudo apt install docker-compose-plugin
```

### Centos/Fedora/RHEL
```
sudo dnf install docker-compose-plugin
```

### تجربة 
 ```
 docker compose version
```

اذا تريد استخدام دوكر بدون جذر
هنا شرح عن الطريقة لتثبيت دوكر rootless

https://discourse.aosus.org/t/topic/2228

تعديل1: تم تحديث طريقة تثبيت Compose مع استخدام حزمة `docker-compose-plugin` الجديدة
## المصادر

https://docs.docker.com/engine/install/

https://docs.docker.com/compose/cli-command/

**هل تستخدم دوكر؟**