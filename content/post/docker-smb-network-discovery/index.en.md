---
title: setup a Samba share with network discovery using docker-compose
categories: 
    - guides
tags:
    - linux
    - smb
    - samba
    - samba local network share
    - docker
    - wsdd
    - avahi
    - docker compose
    - podman
    - linux containers
    - docker container
    - cloud computing
date: 2022-12-31
image: thumbnail.jpg
summary: Easily setup an SMB share with network discovery for MacOS and windows using docker and Docker-compose
slug: docker-smb-network-discovery
keywords: 
    - smb
    - samba 
    - docker
---

I am going to show you how to quickly set up a discoverable SMB share, it's much easier than you think!
Massive thanks to [crazymax](https://crazymax.dev/) for his awesome docker containers that make this possible.

## set up the SMB share
we're going to use crazymax's smb container, pretty easy to setup and docker native.
its recommended to use host networking for it.

### docker-compose.yml

```yaml
## https://fariszr.com/en/docker-smb-network-discovery/
version: "3.5"

services:
  samba:
    image: crazymax/samba
    container_name: samba
    network_mode: host
    volumes:
      - ./smb:/data
      - ./downloads:/downloads
    environment:
      - "TZ=$TIMEZONE"
      - "SAMBA_LOG_LEVEL=0"
    restart: always
```
don't forget to change $TIMEZONE to your local timezone, [list of tz time zones](https://wikipedia.org/wiki/List_of_tz_database_time_zones)

### Configuration

Unlike normal SMBd, the container uses YAML for configuration.

```yaml
auth:
  - user: root
    group: root
    uid: 0
    gid: 0
    password: bar

global:
  - "force user = root"
  - "force group = root"

share:
  - name: downloads
    path: /downloads
    browsable: yes
    readonly: no
    guestok: yes
    veto: no
    recycle: yes 
```
Here is a basic config for an open local share.
All my files are owned by root, that's why i need to force root as the default user.

Put the config in the mounted data folder, so in the case of compose file above, its `./smb/config.yml`

You can add more shares or more users and far more in the config.
More details about docker-smb's configuration options are in the project's [GitHub repo](https://github.com/crazy-max/docker-samba)

After this you should be able to use the SMB share, though not discoverable.

## WSDD, SMB network discovery for Windows

Add this to the `docker-compose.yml` file

```yaml
  wsdd:
    image: jonasped/wsdd
    network_mode: host
    environment:
      - 'HOSTNAME=$HOSTNAME'
    restart: always
```
just replace `$HOSTNAME` with your hostname and then start the container

```bash
docker compose up -d
```

Now you should see the share with the `$HOSTNAME` as its name in the network tab (if local device discovery is enabled) in Windows file explorer.

## Avahi, SMB network discovery for MacOS and Linux

Add this to `docker-compose.yml`, make sure to change `$HOSTNAME` to whatever you want the share's name to be.

```yaml
  avahi:
    image: ydkn/avahi
    hostname: $HOSTNAME
    network_mode: host
    volumes:
      - ./avahi-services:/etc/avahi/services:ro
    restart: always
```

### avahi-services/smb.service
create the file `smb.service` with the following content:

```xml
<?xml version="1.0" standalone='no'?>
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
 <name replace-wildcards="yes">%h</name>
 <service>
   <type>_smb._tcp</type>
   <port>445</port>
 </service>
</service-group>
```

save it and start the container
```bash
docker compose up -d
```

You should now have Network discovery working on almost every OS!

## Sources

https://github.com/crazy-max/docker-samba/issues/1

https://github.com/crazy-max/docker-samba