---
title: Setup Contact and Calendar sync using Radicale and Docker compose.
categories:
  - guides
tags:
  - gnome
  - Dark mode
  - flatpak
  - Linux
draft: true
date: 2024-11-25
slug: contact-calendar-sync-Radicale-docker
summary: Sometimes there are issues with flatpak apps not following Dark mode on gnome, here are few fixes and workarounds for this. 
image: thumbnail.png
keywords:
  - gnome
  - dark mode
  - Flatpak
  - Ubuntu
---

I have been looking for a syncing solution to sync my contacts across devices.
Etesync seemed to be great for this, but it seems the IOS app isn't maintained anymore.

So i found Radicale, a light and easy to setup server for Caldav/cardav syncing, and in true fariszr fashion, I'm going to install it with docker compose.

Now radicale doesn't seem to have an official docker image, but luckily for us [tomesquest](https://github.com/tomsquest/docker-radicale) did the hard part for us and created a docker image for Radiacle.

## docker-compose.yml
*note this will use an external network called `web` to connect the service to the reverse proxy, you can also just expose it.

```yaml
networks:
  web:
    external: true

services:
  radicale:
    image: tomsquest/docker-radicale:latest
    container_name: radicale
    # ports:
    #   - 127.0.0.1:5232:5232
    init: true
    read_only: true
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - SETUID
      - SETGID
      - CHOWN
      - KILL
    deploy:
      resources:
        limits:
          memory: 256M
          pids: 50
    healthcheck:
      test: curl -f http://127.0.0.1:5232 || exit 1
      interval: 30s
      retries: 3
    restart: unless-stopped
    volumes:
      - ./data:/data
      - ./config:/config
    networks:
      - web
```

## config

There's a template config file for the docker image, we still need to change somethings to be able to setup auth through htpasswd.

create a file called config with the raw content of [the config file from github](https://github.com/tomsquest/docker-radicale/blob/master/config)

## Auth

## importing data
For me importing data from the webui didn't work, i just re-imported my contacts again on my phone and it worked.
