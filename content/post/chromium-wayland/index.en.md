---
title: how to fix screen sharing in Chromium browsers on Wayland
keywords:
  - Linux
  - wayland
  - chromium
categories:
  - Linux
tags:
  - Chromium
  - Gnome
  - Linux
  - Wayland
  - Brave
date: 2022-09-02
image: image.jpg
slug: fix-screensharing-wayland-chromium
summary: how to share your display in Chromium browsers on Wayland using Pipewire
lastmod: 2022-10-28
---

If you use a distro with Wayland, you might have noticed, that screen sharing might not work, it will probably show a black screen, instead of your display

## Why

This is because Wayland doesn't give the permission for a program to capture the screen the same way X.ORG does.

Since your distro/Desktop environment uses Wayland, it probably uses Pipwire too, we will use it to share our screen.

## Make Chromium use Pipewire
open your chromium browser and open this link `chrome://flags/#enable-webrtc-pipewire-capturer`

![](screenshot.png)

And Then just restart your browser from the restart option, and you should be OK.

## Notes

It's still not seamless though, when you share your screen, there will be two pop-ups instead of only one.
One from the browser and then one from Pipewire.

And when you select your screen, It's going to prompt you to select it again when its actually going to share it to the application, and not just preview it.

As for Electron apps, this Option should be enabled by default on new version, however on old ones, you might need to find a way to enable it manually.

But most electron apps have a web version anyway, so just use it instead of the app!