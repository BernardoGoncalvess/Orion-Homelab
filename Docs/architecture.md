# Architecture

Orion is a single home server running Ubuntu Server. Every service runs in its own Docker container, defined in its own folder under [`docker/`](../docker).

![Orion architecture](images/architecture.png)

## Components

| Component   | Role                                                   | Host port | Status  |
| ----------- | ------------------------------------------------------ | --------- | ------- |
| Nextcloud   | File storage and sync (macOS, Windows and iOS clients) | 8080      | Running |
| MariaDB     | Database for Nextcloud, not exposed outside Docker     | none      | Running |
| Vaultwarden | Self-hosted password manager                           | 8181      | Running |
| cloudflared | Outbound tunnel to Cloudflare                          | none      | Running |
| Pi-hole     | DNS-level ad and tracker blocking                      | n/a       | Planned |
| Gitea       | Self-hosted Git server                                 | n/a       | Planned |

## Traffic flows

**From the internet (Nextcloud and Vaultwarden)**

1. A client requests the service's hostname.
2. The request reaches Cloudflare.
3. Cloudflare forwards it through the tunnel to `cloudflared` on Orion.
4. `cloudflared` passes it to the right container on the local host.

No inbound ports are opened on the router. See [remote-access.md](remote-access.md) for why.

**From the home network**

Devices on the local network can reach the services directly. Once Pi-hole is set up, home devices will also use it as their DNS server, and Gitea will be available on the local network.

## Design choices

- **One folder per service.** Each service can be deployed on its own and has its own `.env.example`.
- **Secrets stay out of Git.** Real `.env` files, tunnel credentials and data folders are ignored; only placeholder examples are committed.
- **Rebuilt from scratch.** The current setup is a rebuild of an earlier version, with a cleaner structure. Services are being brought back one at a time, Pi-hole next, then Gitea.
