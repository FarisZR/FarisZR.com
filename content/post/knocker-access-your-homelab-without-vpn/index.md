---
title: "Knocker: access your homelab from outside without needing a VPN!"
date: 2025-11-08
tags:
    - Homelab
    - knock-based
    - firewall
    - reverse-proxy
    - Knocker
# image: thumbnail.jpg
slug: knocker-access-your-homelab-without-vpn
keywords:
  - Homelab
  - Tailscale
  - VPN
  - Firewall
draft: true
---

I have a homelab, but it's very annoying to access it outside my home network, so i created Knocker!
A knock based access control service for your homelab, that doesn't break the mobile Apps!

Knocker's strong point is its clients, i built a PWA web app, a go CLI and an Android app, so you are covered on all major platforms.

## How does it work?

```mermaid
sequenceDiagram
    participant User
    participant Caddy as Reverse Proxy (Caddy)
    participant Firewall as Firewalld (knocker zone)
    participant Knocker
    participant Service as Protected Service (port 22)

    Note over User,Service: Initial state — service requires whitelisting

    alt Proxy mode (default — Knocker acts as auth for a reverse proxy)
        Note over User,Caddy: Request hits reverse proxy which asks Knocker
        User->>Caddy: HTTP request to protected service
        Caddy->>Knocker: GET /verify (copies X-Forwarded-For)
        Knocker-->>Knocker: check always_allowed_ips / excluded_paths / whitelist
        alt IP whitelisted
            Knocker-->>Caddy: 200 OK (empty body)
            Caddy->>Service: forward request
            Service-->>Caddy: 200 OK
            Caddy-->>User: 200 OK
        else IP not whitelisted
            Knocker-->>Caddy: 401 Unauthorized (empty body)
            Caddy-->>User: 401 Unauthorized
        end

        Note over User,Knocker: Performing a "knock" to add a whitelist entry
        User->>Knocker: POST /knock (X-Api-Key, optional ip_address, ttl)
        Knocker->>Knocker: validate API key, determine client IP
        Knocker->>Knocker: update whitelist.json with expiry
        Knocker-->>User: 200 OK (whitelisted_entry, expires_at, expires_in_seconds)
    else Firewall mode (knocker manipulates host firewall rules)
        Note over User,Firewall: Initial state — monitored port is blocked by default
        User->>Firewall: TCP SYN to Service:22
        Firewall-->>User: DROP (no response)

        Note over User,Knocker: User performs a knock to whitelist their IP
        User->>Knocker: POST /knock (X-Api-Key, optional ip_address, ttl)
        Knocker->>Knocker: validate API key & determine client IP
        Knocker->>Firewall: add rich accept rule for client IP on port 22 with timeout
        Firewall-->>Knocker: success

        Note over Firewall,User: New rule overrides DROP due to higher priority
        User->>Firewall: TCP SYN to Service:22
        Firewall->>Service: forward packet
        Service-->>User: TCP SYN-ACK (connection established)
        Knocker->>Knocker: update whitelist.json with expiry
    end
```

### But Tailscale Already exists!

I already use Tailscale, in fact the IP of my homelab is the tailscale ip because i already added custom routes for the tailscale IPs.

But using a vpn is annoying, it has to be installed on each device, which is a hassle on a smart TV for example and the Tailscale app on android kills the battery life.

With knocker you just need one device to allow the entire network (thanks to NAT).

### Is this as secure as a VPN?

NO.
Knocker is a compromise, it's more convenient than a VPN (IMO), but because of this convenience it's also not as secure, as you can't whitelist devices, rather only Source IPs, and IPs that could be CGNAT IPs.

You are basically making a bet that in that whitelist period, the likelyhood of a hacker finding out about your service and trying to hack it is quite slim.

That's why you also should use short TTLs in public networks
But In general you should put knocker in front of services that have their own auth.

## Setup

Knocker is distributed as a docker container, that optionally uses the systen dbus socket to interact with the FirewallD daemon.