# Vaultwarden

Lightweight, self-hosted server compatible with the Bitwarden clients. It stores my password vault on my own hardware.

## Run

```bash
docker compose up -d
```

The web vault listens on host port `8181`. It is reached from the internet through the Cloudflare Tunnel; see [remote-access.md](../../docs/remote-access.md).

## Configuration

| Setting           | Value   | Why                                                                                                                                                       |
| ----------------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SIGNUPS_ALLOWED` | `false` | Nobody can register a new account on a server that is reachable from the internet. Set it to `true` only while creating accounts, then turn it off again. |

There are no secrets in this compose file, so no `.env` file is needed. If secrets are added later (for example an admin token), move them to a `.env` file that is not committed and add a `.env.example` here.

## Data

The vault is stored in the named Docker volume `vaultwarden_data`. It is encrypted, but it is still the most sensitive data on the server: it is never committed to Git and it must be included in the [backups](../../docs/backups.md).

## Security notes

- Two-factor authentication should be enabled on every account.
- Sign-ups are disabled.
- The service is only reachable from outside through the tunnel, never through an open router port.
