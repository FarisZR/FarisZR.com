---
categories:
  - quick-snippets
title: "FYI: 2025 Huawei Matepads can't play 4k youtube videos"
date: 2026-01-05
slug: huawei-matepad-pro-4k-youtube-vp9
image: thumbnail.jpg
keywords:
  - Huawei
  - Matepad
  - Youtube
  - 4k
  - VP9
---

I got my hands on a 2025 Huawei Matepad pro 12.2, with a gorgeous display.
I was kinda stocked to use it as my media power house, that's where the weirdness started.
The device can't play 4k videos on YouTube for some reason, and to make it worse, it's a known issue, but it isn't something that gets mentioned in reviews.
It can play 1080p vp9 videos just fine, and you can play 4k video on YouTube through the browser, so what's going on here?

## The reason

Huawei in their ultimate wisdom chose to support the paid h264 and HEVC codecs, but cheeped out on supporting the royalty-free VP9 codec used by the world's most popular streaming Platform, YouTube.
That's why streaming apps like Netflix can play 4k just fine, because they use HEVC.

YouTube uses VP9, and the Kirin chipset has **no hardware VP9 decoder whatsoever**. it can only play 1080p vp9 videos because the software decoder is limited to 2048x2048, aka 1920x1080.

Browsers can include their own software decoder, and that's what happening here, when playing a 4k video, the browser is using the CPU to do the decoding, and that's why it drops frames on 4k 60fps videos.

What an absolutely asinine decision, what makes it crazier is that VP9 is supported by devices going back to **2013**! 

## Confirmed by Huawei support

I contacted the official Huawei support, and they confirmed that this is a known issue with the workaround being playing videos through the browser.

They also said this issue is known for the 2025 13.3 and 12.2 Matepad Pros and the 2025 Matepad 12X, aka all of their 2025 releases. Great!

A flagship tablet that can't play 4k videos in 2025, what a flagship!

## How I found out

I let OpenCode loose with ADB shell access to the device and "we" systematically checked:

1. **Huawei Browser (4K VP9)**: `dumpsys media.resource_manager` showed only software VP9 decoders available (`c2.android.vp9.decoder`, `OMX.google.vp9.decoder`). Hardware decoders exist only for H.264 and HEVC. CPU monitoring revealed `vpx tile worker` threads maxing out cores - clear software decoding.

2. **Codec inventory**: Full decoder list from `dumpsys media.player` confirmed hardware support limited to `OMX.hisi.video.decoder.avc` (H.264) and `OMX.hisi.video.decoder.hevc` (HEVC, 4K capable). VP9 is software-only.

3. **Grayjay (1080p AVC)**: Confirmed hardware acceleration with `OMX.hisi.video.decoder.avc` allocation and efficient MediaCodec threads.

4. **Grayjay (1080p VP9)**: Switched to VP9 stream and confirmed degradation to `c2.android.vp9.decoder` software path. Even at 1080p, no hardware acceleration exists for VP9.