---
title: كيفية إضافة دعم Element Call مع Docker Compose و Caddy
categories:
  - guides
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
description: إعداد دعم Matrix RTC (Element Call) مع docker compose و Caddy reverse proxy
---

إذا كان يظهر لك خطأ `MISSING_MATRIX_RTC_FOCUS` على Element X حديثا عند بدء المكالمات، فالسبب هو أن شركة Element توقفت عن تقديم خدمة الrelay المجانية لـ Matrix RTC، المبني عليه Element Call.
كانت هذه الخدمة موجودة فقط لتسريع اعتماد Element X في البداية.

لحسن الحظ، إضافتها ليس بالأمر الصعب. الجزء الصعب هو إعدادات ال reverse proxy وتحديث ملفات well-known ولكن، الشرح هذا سيختصر الموضوع عليك 😅

نحتاج إلى نطاقين: واحد لـ LiveKit (خدمة RTC) وآخر لخدمة JWT auth لـ Element X.

## تعديلات Docker Compose

(livekit_api_key) = اسم الAPI key

(livekit_api_secret) = نص عشوائي من 20+ حرف.

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
إضافات لإعدادات Synapse:

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

 أسهل إعداد reverse proxy هو وضع الproxy لخدمة JWT و LiveKit على عناوين مختلفة، لان لم أستطيع جعله يعمل مع proxy ل subdirectory لـ JWT على نفس العنوان.


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
**مهم:** إذا كنت تستخدم العنوان المفوض (delegated) على التطبيقات (أي fariszr.com بدلاً من matrix.fariszr.com عند تسجيل الدخول) **ستحتاج** إلى تحديث ملف client well-known المقدم على العنوان المفوض أيضاً.

والآن  لديك home server يدعم Matrix RTC!

[إعدادات خادمي عامة](https://github.com/FarisZR/Server/tree/main/matrix)، بسبب طريقة GitOps التي تعمل بستخدام [docker-compose-gitops-action](https://fariszr.com/ar/docker-compose-gitops-github/).

## المصادر

https://github.com/element-hq/element-call/blob/livekit/docs/self-hosting.md

https://willlewis.co.uk/blog/posts/deploy-element-call-backend-with-synapse-and-docker-compose