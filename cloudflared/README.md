# Cloudflare Tunnel

`cloudflared` runs directly on the host (not in a container) as a systemd service. It opens an outbound connection to Cloudflare and forwards requests for my hostnames to the local services. Why a tunnel and not a VPN or port forwarding is explained in [remote-access.md](../docs/remote-access.md).

## Files

| File                 | Location on the server                    | In Git?                                                              |
| -------------------- | ----------------------------------------- | -------------------------------------------------------------------- |
| Tunnel configuration | `/etc/cloudflared/config.yml`             | Only as [`config.example.yml`](config.example.yml) with placeholders |
| Tunnel credentials   | `/etc/cloudflared/<TUNNEL_ID>.json`       | **Never**                                                            |
| Account certificate  | `~/.cloudflared/cert.pem`                 | **Never**                                                            |
| systemd unit         | `/etc/systemd/system/cloudflared.service` | Example below                                                        |

## Routes

| Public hostname         | Local service                        |
| ----------------------- | ------------------------------------ |
| `vault.example.com`     | Vaultwarden, `http://localhost:8181` |
| `nextcloud.example.com` | Nextcloud, `http://localhost:8080`   |
| `ssh.example.com`       | SSH, `ssh://localhost:22`            |

## Setup outline

1. Install `cloudflared` on the host from the official `.deb` package published by Cloudflare.
2. Authenticate with `cloudflared tunnel login` and create a tunnel with `cloudflared tunnel create <name>`. This generates the credentials file.
3. Write the configuration to `/etc/cloudflared/config.yml` (see the example) and place the credentials file next to it.
4. Create a DNS record for each hostname with `cloudflared tunnel route dns <name> <hostname>`.
5. Install the systemd unit below, then enable and start it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now cloudflared
```

### systemd unit

```ini
[Unit]
Description=Cloudflare Tunnel client
After=network-online.target
Wants=network-online.target

[Service]
TimeoutStartSec=15
Type=notify
ExecStart=/usr/bin/cloudflared --no-autoupdate --config /etc/cloudflared/config.yml tunnel run
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

## Security notes

- The credentials file and `cert.pem` give control over the tunnel. They stay on the server and are covered by `.gitignore`.
- Exposing SSH through the tunnel needs extra protection: key-only authentication, no password login, no root login, and ideally a Cloudflare Access policy in front of the SSH hostname.
- Real hostnames and the tunnel ID are replaced by placeholders in this repository.
