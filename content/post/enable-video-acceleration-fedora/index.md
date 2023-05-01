---
title: تشغيل تسريع الفيديو على كرت الشاشه في فيدورا 36+/Centos/RHEL
categories:
  - شروحات
keywords:
  - Video Acceleration
  - Fedora drivers
  - Fedora mesa video acceleration
  - Chromium video acceleration
  - Video acceleration linux firefox
  - CentOS GPU video acceleration
  - RHEL video acceleration
  - H264 video acceleration on fedora
  - تسريع الفيديو على فيدورا
  - استخدام كرت الشاشة لفيديوهات يوتيوب فيدورا
image: thumbnail.jpg
date: 2023-01-31
slug: enable-video-acceleration-fedora
---
**(هذا فقط يؤثر على تعريفات MESA, وليس التعريفات المغلقة)**

قبل اصدار فيدورا 37 بفترة, [تم اقرار حذف تسريع الفيديو لترميزات الفيديو المغلقة](https://src.fedoraproject.org/rpms/mesa/c/94ef544b3f2125912dfbff4c6ef373fe49806b52?branch=rawhide) مثل H264, HEVC وغيرها

السبب هو قوانين حقوق الملكيه في أمريكا وبعض الدول الاخرى, وقد تعاقب التوزيعات قانونيا في حال أستمرار تضمينها بالتوزيعة متجاهلين القوانين عن قصد.

توزيعات في مناطق إخرى لديها سياسات مختلفه بسبب اختلاف قوانين حقوق الملكية, احد اكثر التوزيعات المستفيدة من الاختلاف هي Ubuntu, لإنها تستطيع تضمين التعريفات المغلقة مع النظام, وهو شيء (حسب فهمي) غير مسموح بالقانون الأمريكي.

## تفعيل مستودع RPMfusion
الحزم المستخدمة لتفعيل الترميزات المغلقة غير موجوده بمستودعات فيدورا/RHEL/CentOS الرسمية.
مستودع RPMFusion هو مستودع شهير يحتوي على تعريفات مغلقة, وحزم مغلقه لتوزيعات Red Hat
هو منفصل تماما عن مشروع Fedora, ويجب تفعيلة يدويا.

### تثبيت رسومي

أدخل على الرابط في الأسفل, وحمل الحزمه التي تناسب اصدار توزيعتك.
قم بتثبيت Free ثم بعدها Nonfree

https://rpmfusion.org/Configuration

### موجه ألأومر

#### Workstation

free + non-free
```
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

#### Silverblue
free + non-free

```
sudo rpm-ostree install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

#### RHEL / CentOS

```bash
sudo dnf install --nogpgcheck https://dl.fedoraproject.org/pub/epel/epel-release-latest-$(rpm -E %rhel).noarch.rpm
sudo dnf install --nogpgcheck https://mirrors.rpmfusion.org/free/el/rpmfusion-free-release-$(rpm -E %rhel).noarch.rpm https://mirrors.rpmfusion.org/nonfree/el/rpmfusion-nonfree-release-$(rpm -E %rhel).noarch.rpm
```

CentOS Steam 8 يحتاج خطوة اضافية
```bash
sudo dnf config-manager --enable powertools
```
RHEL 8 أيضا يحتاج خطوة اضافية
```bash
sudo subscription-manager repos --enable "codeready-builder-for-rhel-8-$(uname -m)-rpms"
```

## تنزيل تعريفات مع ترميزات مغلقة

### AMD
```bash
sudo dnf swap mesa-va-drivers mesa-va-drivers-freeworld
sudo dnf swap mesa-vdpau-drivers mesa-vdpau-drivers-freeworld
```

اذا تستخدم مكتبات i686 (لستيم او برمجيات اخرى):
```bash
sudo dnf swap mesa-va-drivers.i686 mesa-va-drivers-freeworld.i686
sudo dnf swap mesa-vdpau-drivers.i686 mesa-vdpau-drivers-freeworld.i686
```

### Intel
```bash
sudo dnf install intel-media-driver
```

#### كروت غير حديثة
```bash
sudo dnf install libva-intel-driver
```

### Nvidia
تعريف انفيديا المغلق لا يدعم VAAPI, لذلك سنحتاج ترجمه طلبات VAAPI الى NVDEC/NVENC

```bash
sudo dnf install nvidia-vaapi-driver
```

## مصادر

https://ask.fedoraproject.org/t/proprietary-video-codecs-are-no-longer-hardware-accelerated-by-default-on-amd-gpus-on-fedora-37/28965

https://rpmfusion.org/Howto/Multimedia

Photo by [Thomas Foster](https://unsplash.com/it/@thomasfos?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)</a> on [Unsplash](https://unsplash.com/photos/vWgoeEYdtIY?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
  