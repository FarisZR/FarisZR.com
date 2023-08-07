---
title: setup Squid proxy with docker
categories:
  - guides
tags:
  - squid
  - docker
  - ubuntu
  - socks5
date: 2023-07-18
image: thumbnail.jpeg
slug: squid-proxy-docker-setup
draft: true
---

I use proxies very often as a way to do split tunneling on Linux, as easy split-tunneling with WireGuard or OpenVPN isn't there yet.

The quickest way to set up anything IMO is with docker, and here is a quick guide for squid proxy setup using the official Ubuntu-squid image, which seems to be the only up-to-date docker image on Docker hub.

## docker-compose.yml
```yaml
version: "3"
services:
  proxy:
    image: ubuntu/squid
    ports:
      - "3128:3128"
    environment:
      - TZ=UTC
    volumes:
      - ./squid.conf:/etc/squid/squid.conf
    configs:
      - source: squid
        target: /etc/squid/squid.conf

  configs:
    squid:
      file: ./squid.conf
```
I used configs to prevent any permission issues popping up.

## squid.conf
```
http_port 3128
http_access allow all
```

This is a simple setup for a local transparent proxy, it allows all connections, and it doesn't cache or log anything.



## sources
https://medium.com/setup-a-web-proxy-server-with-docker-d5c6942b5575

https://hub.docker.com/r/ubuntu/squid