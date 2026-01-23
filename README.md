# v2ray + Caddy
A step-by-step configuration guide for setting up VPN tunneling through v2ray on both server and client sides. Intended for use in jurisdictions with restricted internet access.

## Legal / License

This project is provided under the terms of the MIT License (see `LICENSE` in the repository root). Use of tools that enable bypassing network controls may be restricted or illegal in some jurisdictions. You are responsible for ensuring your use complies with local laws. The software is provided "as is", without warranty of any kind; to the maximum extent permitted by law the authors disclaim all liability for damages arising from its use.

Third-party components (for example Docker, Caddy, v2ray, and related libraries) are distributed under their own licenses and remain subject to those license terms.

## Introduction
This repository is separated into two separate parts, server and client. See the README files in the respective folders for detailed, runnable instructions:

- `server/README.md` - server configuration, `docker-compose.yml`, `Caddyfile`, `server.json`.
- `client/README.md` - client configuration examples and `client.json`.

## Prerequisites

- A domain name that you control (for example, `example.com`).
- A server reachable from the Internet (VPS or a machine on your LAN with port forwarding).
- Docker should be installed on the server.

## Generate a UUID (client credential)

The UUID is the secret credential clients use to authenticate with v2ray. Keep it private and store it somewhere safe.

On Linux/macOS:

```bash
uuidgen
# or
cat /proc/sys/kernel/random/uuid
```

Also see: https://github.com/net4people/bbs/issues/561

On Windows (PowerShell):

```powershell
[guid]::NewGuid().ToString()
```

Replace the `id` field in `server.json` with the generated UUID.

## Server setup

1. Edit `server.json` and replace the placeholder client `id` with your generated UUID.
2. Edit `Caddyfile` and replace the site (example.com) with your domain.
3. Ensure ports 80 and 443 are open and routed to the server. If you host at home, configure port forwarding for TCP 80 and 443 to the server.
4. On the server, start the services with:

```bash
docker compose up -d
```

5. Verify services and view logs:

```bash
docker container ls
docker logs -f caddy
docker logs -f v2ray
```

6. Verify TLS: open https://your-domain/ in a browser. Caddy should serve a valid TLS certificate.

## Notes

- In this configuration Caddy listens on host ports 80 and 443. Caddy forwards requests for `/vpn` to the `v2ray` container on the internal Docker network and container port 443. v2ray is not directly exposed on host port 443.
- If certificate issuance fails, confirm that ports 80 and 443 are reachable from the Internet and not blocked by the host or provider.

## Client setup

Create a `client.json` (or configure your GUI client) and set the following fields:

- `address`: your domain (for example, `example.com`)
- `port`: `443`
- `id`: your UUID
- `path`: `/vpn`

Start the client so it listens locally and provides a local SOCKS proxy (for example, `127.0.0.1:10808`). Configure your browser or system to use that SOCKS proxy and verify your external IP at a site such as https://ipinfo.io.

## Troubleshooting

- TLS certificate errors: ensure TCP ports 80 and 443 are open and reachable; check Caddy logs with `docker logs -f caddy`.
- Authentication/connection errors: ensure the UUID in `client.json` matches `server.json`, the domain is correct, and the `/vpn` path matches on both sides.
- Service errors: inspect v2ray logs with `docker logs -f v2ray` and Caddy logs with `docker logs -f caddy`.

## Security note

Treat your UUID as a secret. Anyone who has the UUID and your server address can route traffic through the server. If you suspect the UUID has been exposed, replace it and update the server and client configuration.


## Repository layout

- Server files and instructions: `server/README.md` (includes `docker-compose.yml`, `Caddyfile`, and `server.json`).
- Client files and instructions: `client/README.md` (includes `client.json` example).

## Automated setup

This repository includes a convenience shell script `setup.sh` at the repository root. The script prompts for a UUID (or generates one), prompts for your domain, and optionally collects `/etc/hosts` mappings. It reads the templates in `server/server_template.json` and `client/client_template.json`, fills them, and writes active configs to `server/server.json` and `client/client.json`.

Usage:

```bash
bash setup.sh
```

The script will not modify `/etc/hosts` automatically. If you provide host mappings it will write them to `hosts_additions.txt` and display a command you can run to append them to `/etc/hosts` after review.

Requirements: `bash` and `python3` are required (Python is used to safely modify the JSON templates).

Note: `Server.md` and `Client.md` in the repository root have been replaced with pointers to the new locations.
