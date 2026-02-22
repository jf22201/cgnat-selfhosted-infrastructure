# cgnat-selfhosted-infrastructure

Personal self-hosted infrastructure, designed to operate securely behind Carrier-Grade NAT (CGNAT) using Tailscale mesh networking and Cloudflare Tunnel.

# Services

## Vaultwarden

- Usage: Self hosted password manager
- Access: Tailscale.

### Generating the Vaultwarden admin token

Run the following docker container to generate a token based on a password of your choice.

```bash
docker run --rm -it vaultwarden/server /vaultwarden hash
```

### Vaultwarden HTTPS setup

HTTPS is handled by caddy acting as reverse proxy in front of vaultwarden. Refer to caddy/Caddyfile.example for configuration.

Routing is as such:
Bitwarden Client -> Tailscale -> Caddy -> Vaultwarden

## Adguard

- Usage: adblocking at DNS level for local network.
- Access: LAN

### Routing

Client -> Router(LAN) -> Host(LAN)

### LAN setup:

Router DHCP server is assigned as the host to act as the WAN DNS server for the entire network. (If Adguard is not working might need to change this back to default values so that other devices can continue to access the network.)

## Remote access

This setup is designed to run behind ISP CGNAT (no public IP available so impossible to forward ports) at least expense with minimal compromise to security and convenience.

### Tailscale

All remote access to services with higher security requirements is done through tailscale. This is creates a private network between devices (downside is that clients need to be installed on all devices.)

#### Note: MagicDNS as well as HTTPS Certificates should be enabled on the tailscale DNS admin panel to allow for HTTPS for vaultwarden.

### Cloudflared-tunnel

Any services where convenience is a higher priority than security are routed through a cloudflare tunnel. This creates a persistent connection to cloudflare's edge which a domain can be pointed to and route external traffic through CGNAT.

## Host

|            |                         |
| ---------- | ----------------------- |
| **Device** | Lenovo x250             |
| **OS**     | Ubuntu Server 24.04 LTS |
| **Arch**   | x86_64                  |

### Static IP for LAN

Router settings are configured such that the same LAN IP is reserved for the host's MAC address.
