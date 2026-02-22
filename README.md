# Homelab infrastructure

Personal self-hosted infrastructure, designed to operate securely behind Carrier-Grade NAT (CGNAT) using Tailscale mesh networking and Cloudflare Tunnel.

## Host

|            |                         |
| ---------- | ----------------------- |
| **Device** | Lenovo x250             |
| **OS**     | Ubuntu Server 24.04 LTS |
| **Arch**   | x86_64                  |

Repurposed laptop chosen for low power consumption.

# Services

## Vaultwarden

Password manager accessible by web or app.

|            |                                |
| ---------- | ------------------------------ |
| **Access** | Tailscale                      |
| **URL**    | https://tailscale-hostname:443 |

### Setup

**Prereqs**:

- Reverse proxy handling HTTPS must already be installed and routing HTTPS traffic from tailscale domain to this container

**Configuration**:

- Run the following docker container to generate an admin token based on a password of your choice.

```bash
docker run --rm -it vaultwarden/server /vaultwarden hash
```

- Store this into the ADMIN_TOKEN env variable.

**HTTPS handling**:

- Access is done via reverse proxy, hence there is no port binding from container to host.
- Ensure that container is part of homelab docker network to allow reverse proxy to resolve the vaultwarden container internally.

### Architecture

Routing:
Client -> Tailscale -> Caddy -> Vaultwarden

## Caddy

Reverse proxy handling automatic TLS certificate provisioning for internal services

### Setup

**Prereq**:

- Docker network to bridge containers should exist.

**Configuration**:

- Reverse proxy should be bind to host tailscale IP on port 443.
- Check caddy/Caddyfile.example for reverse proxy config to vaultwarden.
- Ensure that https certificates and magicDNS are enabled on tailscale admin panel.

### Troubleshooting

- Caddy should self generate a HTTPS certificate but requires access to tailscale socket at /var/run/tailscale/tailscaled.sock to do so. This needs to bind as a volume inside container from host. This should fix any log errors regarding /var/run/tailscale/tailscaled.sock.

### Architecture

Routing:
Client -> Tailscale -> Caddy -> Vaultwarden/Other services

### Maintenance

- Caddy should automatically handle certificate renewal/obtain unlike Nginx, so no external jobs required to do this.

## Adguard

Network-wide DNS-level ad and tracker blocking. Acts as the DNS server for the entire LAN, intercepting DNS queries from all devices on the network without requiring any per-device configuration.
| | |
| ---------- | -------------------- |
| **Access** | LAN |
| **URL** | http://192.168.50.221:80 |

### Architecture

Client -> Router(LAN) -> Host(LAN)

### Setup

**Prereqs**

- Ensure that host has a static LAN IP, all devices on home network may fail DNS resolution if IP address changes.

**Configuration**

- Set WAN DNS server on home network router to the host's private IP.

### Troubleshooting

- Default upstream DNS server might be slow, consider using Cloudflare for upstream if DNS queries feel sluggish.
- If devices on home network experience loss of DNS, check adguard container status and fall back to ISP default DNS on router.

## Syncthing

|            |                                   |
| ---------- | --------------------------------- |
| **Access** | LAN/tailscale(data transfer only) |
| **URL**    | http://192.168.50.221:8384/       |

Decentralized peer-to-peer file synchronization between devices, with no third-party
server involvement. Files are synced directly between devices over LAN or Tailscale.

### Setup

**Prereq**:

- Docker network bindings for both LAN and Tailscale on host.

**Configuration**:

- Pair devices through web UI.

### Troubleshooting

**Volume permission errors**

By default docker creates volumes with root as owner, causing permission issues when reading/writing.

Change the owners of syncthing/data/\*and syncthing/config/\* to mitigate issues.

```bash
chown -R 1000:1000 data config
```

### Architecture

The data transfer ports are bound on both LAN and Tailscale, allowing full LAN speed if on LAN whilst retaining access when on WAN.

Client -> LAN -> Host

Client -> Tailscale -> Host

WebUI is only bound on LAN.

# Remote access

This setup is designed to run behind ISP CGNAT and achieves remote access with no additional hardware or paid services, using Tailscale free tier and Cloudflare free tunneling.

### Tailscale

All remote access to services with higher security requirements is done through tailscale. This creates a private network between devices.

**Downside** : All clients require Tailscale client installed for remote access to services.

**Configuration**:

- Ensure MagicDNS + HTTPS Certificates are enabled on tailscale admin panel

### Cloudflared-tunnel

Any services where convenience is a higher priority than security are routed through a cloudflare tunnel. This creates a persistent connection to cloudflare's edge which a domain can be pointed to and route external traffic through CGNAT.

Although this doesn't require a client to be installed on devices, there is a security trade off as it is now publicly exposed.

## Uptime Kuma

Service uptime monitoring and alerting dashboard for all hosted services.

|            |                                  |
| ---------- | -------------------------------- |
| **Access** | Tailscale                        |
| **URL**    | `http://tailscale-hostname:3001` |

### Setup

**Prereqs**:

- None

**Configuration**:

- Use sqlite as DB for ease of configuration (No need for a separate DB instance for homelab)
- Make sure to setup notifications and monitors on docker containers via web UI

### Architecture

Client -> Tailscale -> Host

## filebrowser

```bash
touch filebrowser.db
echo '{}' > settings.json
```

Run this in dir before hand for config.
