# Remote access

## Goal

Reach Nextcloud and Vaultwarden from outside my home network without opening ports on the router.

## First attempt: WireGuard (did not work)

My first plan was a WireGuard VPN into the home network. It failed because my ISP puts the connection behind **CGNAT / double NAT**: I do not have a public IP that forwards to my router, so incoming connections never reach the server. A VPN that needs an inbound port cannot work in that situation.

## Solution: Cloudflare Tunnel

With Cloudflare Tunnel, the `cloudflared` service on Orion makes an **outbound** connection to Cloudflare. Requests to my hostnames arrive at Cloudflare and are sent back through that connection to the right container. Because the connection starts from inside my network, CGNAT does not get in the way and no router ports are opened.

```
Internet -> Cloudflare -> tunnel -> cloudflared (Orion) -> Nextcloud / Vaultwarden
```

## Setup outline

`cloudflared` is installed on the host (from the official `.deb` package) rather than in a container. The exact steps depend on how the tunnel is created; the outline is:

1. Install `cloudflared` on Orion.
2. Authenticate it with a Cloudflare account and create a tunnel.
3. Map each public hostname to a local service in the tunnel configuration, for example:

```yaml
# cloudflared/config.example.yml (placeholders only)
tunnel: <TUNNEL_ID>
credentials-file: /path/to/<TUNNEL_ID>.json

ingress:
  - hostname: cloud.example.com
    service: http://localhost:8080
  - hostname: vault.example.com
    service: http://localhost:8181
  - service: http_status:404
```

4. Point the DNS records for those hostnames at the tunnel.
5. Run `cloudflared` as a system service so it starts on boot.

Tunnel credentials (the `.json` file and any `cert.pem`) are secrets and are never committed.

## Trade-offs

- **Depends on a third party.** If Cloudflare is down, remote access is down. Local access still works.
- **Traffic passes through Cloudflare.** TLS is terminated at Cloudflare, which matters when deciding what to expose.
- **Exposing a password manager needs extra care.** Vaultwarden is reachable from the internet, so I keep user sign-ups disabled, use two-factor authentication and protect (or disable) the admin panel.
