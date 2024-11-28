---
title: Setup Contact and Calendar sync using Radicale and Docker compose.
categories:
  - guides
tags:
  - docker
  - compose
  - Linux
draft: true
date: 2024-11-29
slug: contact-calendar-sync-Radicale-docker
summary: Sometimes there are issues with flatpak apps not following Dark mode on gnome, here are few fixes and workarounds for this. 
image: thumbnail.png
keywords:
  - Radicale
  - davx
  - docker
---

I have been looking for a solution to sync my contacts across devices.
Etesync seemed great for this, but it seems the IOS app is no longer maintained.

And then I found Radicale, a lightweight and easy to setup server for Caldav/Cardav syncing, and in true FarisZR fashion, I'm going to install it with docker compose.

Now radicale doesn't seem to have an official docker image, but luckily for us [tomesquest](https://github.com/tomsquest/docker-radicale) did the hard part for us and created a docker image for Radiacle.

## docker-compose.yml
*Note that this uses an external network called `web` to connect the service to the reverse proxy, you could just expose the ports directly.

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


There's a template config file for the docker image, we still need to change something to be able to set up htpasswd auth.

Create a file named config with the raw content of [the config file from github](https://github.com/tomsquest/docker-radicale/blob/master/config)

and modify it to your liking
## Auth
Make sure the auth section looks something like this:

```ini
[auth]
type = htpasswd
htpasswd_filename = /config/users
htpasswd_encryption = bcrypt
```

### Hashing your password with bcrypt
Now, as you probably noticed, htpasswd encryption is set to bcrypt, which means we need to hash the password in bcrypt format.

```bash
echo -n "your password" | mkpasswd --method=bcrypt
```

and then create a file called `users' in the mounted `config' directory.
It should be formatted like this:

```htpasswd
john:$2a$10$l1Se4qIaRlfOnaC1pGt32uNe/Dr61r4JrZQCNnY.kTx2KgJ70GPSm
```

and now it should be ready to run!

## Caddy reverse proxy
```json
sync.example.com {
	reverse_proxy radicale:5232
	encode zstd gzip
}
```
## Importing data
For me, importing data from the web UI didn't work, I just re-imported my contacts on my phone and it worked.
