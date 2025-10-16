---
title: How to use the Proxy Protocol with Caddy (Get source IP from a TCP proxy)
tags:
  - caddy
  - tcp proxy
  - reverse proxy
date: 2025-10-16
slug: use-proxy-protocol-with-caddy
image: thumbnail.jpg
keywords:
  - Caddy
  - Proxy protocol
categories: 
    - quick-snippets
---

My Homelab often has peering issues when connecting from another ISP.
The solution to this is just to pass the connection through a proxy, which has better speed than a direct connection.
But if I want to do this without MITMing the connection, it can't be a classic reverse proxy sandwich.

That's where tools like [rathole](https://fariszr.com/home-server-access-without-port-forwarding/) come in, they are TCP proxies that allow you to forward any TCP connection to your homelab.
However, there's always the issue of the source IP.
Due to how these proxies work, the source will always be the IP of the rathole client on the homelab (AKA the proxy itself) and not the actual source IP of the connection.

Proxy protocol fixes this by appending headers to the TCP proxied connections that give details about the actual source IP.

## Using the Proxy Protocol with Caddy

Assuming your TCP proxy supports the proxy protocol (example, [modified version of rathole](https://fariszr.com/home-server-access-without-port-forwarding/#forward-source-ip-with-proxy-protocol-in-rathole))

```caddyfile
{
    servers {
        listener_wrappers {
            proxy_protocol {
				allow 172.20.0.0/16 #ip with netmask of the proxy
				fallback_policy reject
			}
			tls # the proxied packets are HTTPS,  so we still need the tls wrapper https://caddyserver.com/docs/caddyfile/options#proxy-protocol
        }
    }
}

example.fariszr.com {
  	reverse_proxy service:8000 {
		header_up X-Forwarded-For {client_ip} # {client_ip} is the source ip in the proxy
		header_up X-Real-IP {client_ip}
		header_up X-Forwarded-Proto {scheme}
		header_up X-Forwarded-Host {host}
	}
}
```

Now your app should get the source IP correctly without actually being aware of the proxy protocol and the connection being proxied through a TCP proxy.