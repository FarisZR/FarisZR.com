---
title: Setup Cloudflared using Docker without the ZT dashboard
categories:
  - guides
slug: cloudflared-docker-setup
tags:
  - Cloudflare
  - Cloudflare Argo tunnel
  - Cloudflare Tunnel
  - docker UserNs
  - docker rootless
  - Cloudflared
  - cloudflared docker setup
  - Cloudflare Zero Trust
date: 2022-10-28
summary: Setting up Cloudflared in docker to tunnel to a container without expoing the server.
image: thumbnail-en.jpeg
---

When I wanted to set up Cloudflared, but I couldn't find anything about setting it up in docker, especially without the Zero Trust dashboard (because it kept refusing my credit card for some reason).
So here it is!

I am aiming to set up one tunnel per container, which I think is better and easier to manage then multiple tunnels in one Cloudflared instance.

## docker-compose.yaml
```
version: '3.1'

services:
    cloudflared:
        image: cloudflare/cloudflared
        restart: always
        environment:
        - TUNNEL_ORIGIN_CERT=/etc/cloudflared/cert.pem
        - TUNNEL_EDGE_IP_VERSION=auto #use IPv4 and IPv6
        volumes:
        - ./cloudflared:/etc/cloudflared
        command: tunnel run $(TUNNEL_NAME)
```

## linking Cloudflared with your domain

### Create `cloudflared` dir
firstly you need to create the `./cloudflared` directory before running any commands because on container start It's going to be created as root and Cloudflared runs as the distroless `nonroot`(id 65532) user so you will just end up with permission problems every time.

```
mkdir ./cloudflared
```

#### rootful
```
sudo chown -R 65532:65532 ./cloudflared
```

#### rootless
this is assuming your subid and subgid ranges are `100000:65536`
```
sudo chown -R 165531:165531 ./cloudflared
```

#### User name spaces remapping
for some reason, docker in UserNS mode uses different IDs, even when using the same subuid/subgid.
```
sudo chown -R 165532:165532 ./cloudflared
```

### Login

```
docker run -v $PWD/cloudflared:/home/nonroot/.cloudflared cloudflare/cloudflared login
```
Open the link in your browser and select which domain you would like to authorize it for.
Then it would generate the origin cert.

## Create a new cloudflared tunnel

```
docker run -v $PWD/cloudflared:/etc/cloudflared cloudflare/cloudflared tunnel create ${TUNNEL_NAME}
```
We switched from `/home/nonroot/.cloudflared` to `/etc/cloudflared` because tunnel files are generated in the `/etc` directory.
We overrode the default certificate location in the compose file using the `TUNNEL_ORIGIN_CERT` variable.

Now you will find in `./cloudflared` a `cert.pem` file and a `$TUNNEL_ID.json` file.
Copy the tunnels ID because you are going to need it when configuring Cloudflared and setting up DNS later on.

## Configuring the tunnel

create a `config.yml` file inside the `./cloudflared` directory

This is a basic configuration for a WordPress site inside the same docker network running at port 80.

```
tunnel: ${TUNNEL_ID}
credentials-file: /etc/cloudflared/${TUNNEL_ID}.json

ingress:
  - hostname: domain.tld
    service: http://wordpress:80
  - hostname: www.domain.tld # otherwise its going to end up in a 404
    service: http://wordpress:80
  - service: http_status:404 # any other domain will result in a 404
```

You could further customize the configuration to your liking.

[here](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/local-management/) are the details about it.

But this would be enough for most setups.

## Start it up!
Make sure cloudflared is running in the same network as your other container if you are using hostnames, and it should just work!

```
docker compose up -d
```

## DNS Records

Add a `CNAME` record to your domains that will be using the tunnel pointing to `${TUNNEL_ID}.cfargotunnel.com`, make sure to enable Cloudflare proxy status (the cloud needs to be orange).

## rootless/UserNs note

if you see this error
```
WRN The user running cloudflared process has a GID (group ID) that is not within ping_group_range. You might need to add that user to a group within that range, or instead update the range to encompass a group the user is already in by modifying /proc/sys/net/ipv4/ping_group_range. Otherwise cloudflared will not be able to ping this network error="Group ID 65532 is not between ping group 65534 to 65534"
WRN ICMP proxy feature is disabled error="cannot create ICMPv4 proxy: Group ID 65532 is not between ping group 65534 to 65534 nor ICMPv6 proxy: socket: permission denied"

```
This has no effect currently, as Cloudflared [still doesn't support](https://github.com/cloudflare/cloudflared/issues/726) ICMP over QUIC anyway.
I tried fixing it by setting `net.ipv4.ping_group_range = 0 2147483647`, but it still didn't work, so just ignore it for now.

If you have a solution, write it in the comments!