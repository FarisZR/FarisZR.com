---
title: How to add support for Element Call with Docker Compose & Caddy
categories:
  - Guides
tags:
  - Caddy
  - Element Call
  - Matrix RTC
  - docker compose
date: 2025-05-27
slug: matrix-rtc-setup-with-docker-caddy
image: thumbnail.jpg
keywords:
  - matrix
  - caddy
  - matrix-rtc
description: setup Matrix RTC (Element Call) Suppport with docker compose and Caddy reverse proxy
---

If you have been getting `MISSING_MATRIX_RTC_FOCUS` on Element X recently when starting calls, the reason is that Element has stopped their free relay for Matrix RTC, AKA Element Call.
It was there just to speed up initial Element X adoption.

Luckily, adding it isn't that hard. The tricky part is the reverse proxy and updating the well-known files for the clients, but don't worry, I already did the hard work for you 😅

We need two domains: one for LiveKit (the RTC service) and one for the JWT auth service for Element X.

## Docker Compose additions

(livekit_api_key) = api key name

(livekit_api_secret) = random 20+ character string.

```yaml
 # JWT Authentication Service for Element Call
  lk-jwt-service:
    image: ghcr.io/element-hq/lk-jwt-service:latest
    restart: unless-stopped
    environment:
      # Port the JWT service listens on internally
      - LK_JWT_PORT=8080
      # Publicly accessible URL for the LiveKit SFU WebSocket endpoint (updated domain)
      - LIVEKIT_URL=wss://matrixrtc.fariszr.com
      # API Key and Secret (must match LiveKit service and secrets)
      - LIVEKIT_KEY=(livekit_api_key)
      - LIVEKIT_SECRET=(livekit_api_secret)
      # Optional: Restrict call creation to users from specific homeservers
      - LIVEKIT_LOCAL_HOMESERVERS=fariszr.com # server name
    networks:
      default:
      web: # Needs to be reachable by Caddy

  livekit:
    image: livekit/livekit-server:v1.8
    container_name: livekit
    # Use config file instead of just environment variables
    command: --config /etc/livekit.yaml
    restart: unless-stopped
    # LIVEKIT_KEYS is still needed here for the server itself
    environment:
      LIVEKIT_KEYS: "(livekit_api_key): (livekit_api_secret)"
      # Remove other env vars now handled by config.yaml
      # - LIVEKIT_WS_URL=wss://livekit.fariszr.com
      # - LIVEKIT_PORT=7880
    ports:
      # Port 7880 is now only needed internally for Caddy proxying to /livekit/sfu
      # - "7880:7880" # Client API and Webhook callback - proxied via Caddy
      - "7881:7881" # TURN/TLS (LiveKit's own TURN, though disabled in config)
      - "50100-50200:50100-50200/udp" # UDP port range for media over WebRTC
    volumes:
      # Mount the new config file
      - ./livekit-config.yaml:/etc/livekit.yaml:ro
    networks:
      default:
      web: # Needs to be reachable by Caddy
```

## Configs
Additions to the Synapse config:

### homeserver.yaml

```yaml
experimental_features:
  # MSC3266: Room summary API. Used for knocking over federation.
  msc3266_enabled: true
  # MSC4222: Needed for syncv2 state_after. Allows clients to correctly track room state.
  msc4222_enabled: true
  # MSC4140: Delayed events are required for proper call participation signalling.
  # If disabled, you might get stuck calls in Matrix rooms.
  msc4140_enabled: true

# The maximum allowed duration by which sent events can be delayed, as per MSC4140.
max_event_delay_duration: 24h

# Adjust rate limiting for call-related events
rc_message:
  # Needs to match at least E2EE key sharing frequency plus headroom.
  # Key sharing events are bursty.
  per_second: 0.5
  burst_count: 30

rc_delayed_event_mgmt:
  # Needs to match at least the heartbeat frequency (5s = 0.2/s) plus headroom.
  per_second: 1.0 # Increased from example's 1 to be safer
  burst_count: 20
```

### livekit-config.yaml:

```yaml
# Basic LiveKit configuration
port: 7880 # Port LiveKit listens on internally
bind_addresses:
  - "0.0.0.0" # Listen on all interfaces within the container

rtc:
  tcp_port: 7881
  port_range_start: 50100 # Match docker-compose port range
  port_range_end: 50200   # Match docker-compose port range
  use_external_ip: false # Rely on reverse proxy/NAT

logging:
  level: info

# Disable LiveKit's internal TURN server, since you probably have something already set up for the classic P2P calls.
turn:
  enabled: false
  # domain: matrixrtc.fariszr.com # Not needed if disabled, updated comment for consistency
  # cert_file: ""
  # key_file: ""
  # tls_port: 5349 # Default
  # udp_port: 443 # Default
  # external_tls: true # If using external certs, but disabled anyway

# Keys are provided via environment variable LIVEKIT_KEYS in docker-compose.yml
# keys:
#   (livekit_api_key): (livekit_api_secret)
```

## Caddy reverse proxy

I had to spend a lot of time figuring out the best reverse proxy setup. It seems that proxying the JWT service and LiveKit on different domains is the easiest solution, because I couldn't get it to work with a subdirectory override for JWT on the same domain.


```caddy
matrix.fariszr.com {
	reverse_proxy synapse:8008
	encode zstd gzip
	# Serves the Matrix client .well-known JSON configuration file
	respond /.well-known/matrix/client `{"m.homeserver":{"base_url":"https://matrix.fariszr.com"},"org.matrix.msc4143.rtc_foci":[{"type":"livekit","livekit_service_url":"https://jwt.matrixrtc.fariszr.com"}]`
}

jwt.matrixrtc.fariszr.com {
	encode zstd gzip
	reverse_proxy lk-jwt-service:8080
}

matrixrtc.fariszr.com {
	encode zstd gzip
	# Route SFU WebSocket requests to the livekit container on its internal port 7880
	reverse_proxy livekit:7880
}
```

**IMPORTANT:** If you are using your delegated domain on the client (i.e., fariszr.com instead of matrix.fariszr.com when logging in), then you **will** have to update the client well-known file served on the delegated domain too.

And now you should have a Matrix RTC-capable home server!

You can take a look at [my server's setup](https://github.com/FarisZR/Server/tree/main/matrix) anytime on my GitOps repo, run with [docker-compose-gitops-action](https://fariszr.com/docker-compose-gitops-github/).

## Sources

https://github.com/element-hq/element-call/blob/livekit/docs/self-hosting.md

https://willlewis.co.uk/blog/posts/deploy-element-call-backend-with-synapse-and-docker-compose