---
title: Enable Video hardware acceleration on Fedora 36+/RHEL/CentOS
categories:
  - Linux
keywords:
  - Video Acceleration
  - Fedora drivers
  - Fedora mesa video acceleration
  - Chromium video acceleration
  - Video acceleration linux firefox
  - CentOS GPU video acceleration
  - RHEL video acceleration
  - H264 video acceleration on fedora
image: thumbnail.jpg
slug: enable-video-acceleration-fedora
date: 2023-01-31
---
**(This only affects MESA, so only people using open source drivers)**
  
Shortly before Fedora 37 release, Fedora [dropped support](https://src.fedoraproject.org/rpms/mesa/c/94ef544b3f2125912dfbff4c6ef373fe49806b52?branch=rawhide) for closed Codecs like H264, HEVC, VC1, this also includes other Red Hat distros like RHEL and CentOS stream, and probably clones like Rocky and Almalinux.

The reason is mainly to avoid [Closed source software patents](https://www.phoronix.com/news/Mesa-Optional-Video-Codecs), Not every Distribution is affected though, as it depends on their legal jurisdiction, Ubuntu for example still includes Closed source codecs and Closed source drivers(when needed).

## Enable RPMfusion
MESA Packages with closed source codecs aren't available on the Official repos, but in RPMFusion, which is totally separate from Fedora and Red Hat, that's why they can upload such packages.

### Using the GUi

Click on the link below, and install the Free then the non-free repository matching your distribution.

https://rpmfusion.org/Configuration

### using the terminal

#### Workstation

```bash
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

#### Silverblue

```bash
sudo rpm-ostree install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```
#### RHEL / CentOS

```bash
sudo dnf install --nogpgcheck https://dl.fedoraproject.org/pub/epel/epel-release-latest-$(rpm -E %rhel).noarch.rpm
sudo dnf install --nogpgcheck https://mirrors.rpmfusion.org/free/el/rpmfusion-free-release-$(rpm -E %rhel).noarch.rpm https://mirrors.rpmfusion.org/nonfree/el/rpmfusion-nonfree-release-$(rpm -E %rhel).noarch.rpm
```

CentOS Steam 8 requires an additional step
```bash
sudo dnf config-manager --enable powertools
```
RHEL 8 also requires an additional step
```bash
sudo subscription-manager repos --enable "codeready-builder-for-rhel-8-$(uname -m)-rpms"
```

## Install Drivers with closed source codecs included

### AMD
```bash
sudo dnf swap mesa-va-drivers mesa-va-drivers-freeworld
sudo dnf swap mesa-vdpau-drivers mesa-vdpau-drivers-freeworld
```

If using i686 compat libraries (for steam or alike):
```bash
sudo dnf swap mesa-va-drivers.i686 mesa-va-drivers-freeworld.i686
sudo dnf swap mesa-vdpau-drivers.i686 mesa-vdpau-drivers-freeworld.i686
```

### Intel
```bash
sudo dnf install intel-media-driver
```

#### not so recent hardware
```bash
sudo dnf install libva-intel-driver
```

### Nvidia
Nvidia's driver doesn't support VAAPI, so we need to use a bridge to translate VAAPI calls into NVDEC/NVENC
```bash
sudo dnf install nvidia-vaapi-driver
```

## Credits

https://ask.fedoraproject.org/t/proprietary-video-codecs-are-no-longer-hardware-accelerated-by-default-on-amd-gpus-on-fedora-37/28965

https://rpmfusion.org/Howto/Multimedia

Photo by [Thomas Foster](https://unsplash.com/it/@thomasfos?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)</a> on [Unsplash](https://unsplash.com/photos/vWgoeEYdtIY?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
  