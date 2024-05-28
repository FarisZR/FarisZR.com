---
title: How to Make Your Linux Home Server Accessible on the Internet Without Port Forwarding
categories:
  - guides
keywords:
  - TCP tunneling
  - TCP proxy
  - Cloudflared
  - PROXY protocol
  - port-forwarding
  - port forwarding alternative
  - CGnat homelab
  - Rathole
  - Rathole setup
image: thumbnail.jpg
slug: home-server-access-without-port-forwarding
summary: Learn how to make your home server accessible on the Internet without port forwarding using Rathole, a simple TCP proxy that ensures security and privacy.
date: 2024-05-28
---

If you want to make your home server accessible on the Internet but can't use port forwarding (if you are behind CGNAT for example) or just aren't comfortable having your home IP public, there are a few ways to do it.

You could just use Cloudflared, it is free, easy to set up and gives you some security protection through Cloudflare's WAF.
However, Cloudflare is a reverse proxy, meaning that your connection is decrypted, analyzed and then re-encrypted back to its origin, it also has specific T&S limits.

Herein comes [Rathole](https://github.com/rapiz1/rathole), it's a small standalone program written in Rust, it's basically a simple TCP proxy, it doesn't decrypt anything, it just forwards TCP traffic from one host to another using NAT traversal to avoid the need to port-forward anything.

You just need a server with a public IP to tunnel TCP traffic back to your home server.

I am going to use it to proxy Caddy, the reverse proxy I use on my home server, without MITMing any connections.

## Installing Rathole
Rathole is a self-contained package, just download the one that's compiled for your platform and run it from [GitHub](https://github.com/rapiz1/rathole/releases).
You can also run it in a [container](https://hub.docker.com/r/rapiz1/rathole).

This will be split into two parts, as Rathole needs to run on your home server and on the server with a static IP.

### Public server docker-compose.yml

```yaml
services:
  rathole:
    image: rapiz1/rathole
    stdin_open: true
    tty: true
    ports:
      - 8080:8080 # api port
      - 443:443 # forwarded port
    volumes:
      - ./rathole/config.toml:/app/config.toml
    command: --server /app/config.toml
```

## Rathole sever Config

```toml
# server.toml
[server]
bind_addr = "0.0.0.0:8080" # `8080` specifies the port that rathole listens for clients

[server.services.home-proxy]
token = "your_very_secure_shared_token" # Shared value between the client(your home server) and the server(the VPS)
bind_addr = "0.0.0.0:443" # port on the vps routing traffic to your home server "the client"
```

### Home server docker setup
```yaml
# assuming your webserver is connected to a shared docker network called web
networks:
  web:
    external: true

services:
  rathole:
    stdin_open: true
    tty: true
    volumes:
      - /home/lammah/rathole/config.toml:/app/config.toml
    image: rapiz1/rathole
    command: --client /app/config.toml
    networks:
      - web
```

### Home server Rathole Config
```toml
# client.toml
[client]
remote_addr = "yourservers-ip-or-domain:8080" # The address of the server. The port must be the same with the port in `server.bind_addr`

[client.services.home-proxy]
token = "your_very_secure_shared_token" # Must be the same one on the server to pass the validation
local_addr = "caddy:443" # The address of the service that needs to be forwarded
```

Just start them both and you should be up and running!
You will also need to make sure that the domains are pointing to the VPS correctly so that HTTPS and applications work as expected.

## Source IP
There is one caveat though, and it's kind of a big one, the source IP for all connections will be set to 127.0.0.1.
That's a big problem if you want to protect your applications from DOS or endless login attempts.

There is a solution, however, the PROXY protocol

## Forward source IP with PROXY protocol in Rathole

The proxy protocol is a protocol that allows you to forward the source IP without needing too much computation.

Both the TCP proxy and the upstream application must support the PROXY protocol for this to work.
Caddy supports the PROXY protocol, as a [client](https://caddyserver.com/docs/json/apps/http/servers/listener_wrappers/proxy_protocol/) and as a [proxy](https://caddyserver.com/docs/caddyfile/directives/reverse_proxy#the-http-transport).

There is an open [PR](https://github.com/rapiz1/rathole/pull/352) for Rathole which adds support for the PROXY protocol.
You can use my [docker image](https://github.com/FZR-forks/rathole-proxy_protocol/pkgs/container/rathole-proxy_protocol) to use the version with PROXY protocol support, or just build it locally.

Just use the same Configs as before, while adding `enable_proxy_protocol = true` under the `[server.services.home-proxy]` block.
Make sure that the web server/app you're tunneling to supports the PROXY protocol and that it's set up correctly.

Now you should be able to get the correct source IP even with a TCP tunnel in between!