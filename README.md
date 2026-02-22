# cgnat-selfhosted-infrastructure

Personal self-hosted infrastructure, designed to operate securely behind Carrier-Grade NAT (CGNAT) using Tailscale mesh networking and Cloudflare Tunnel.

# Services

### Vaultwarden

- Usage: Self hosted password manager
- Access: Tailscale.

### Adguard

- Usage: adblocking at DNS level
- Access: LAN

## Remote access

This setup is designed to run behind ISP CGNAT (no public IP available so impossible to forward ports) at least expense with minimal compromise to security and convenience.

### Tailscale

All remote access to services with higher security requirements is done through tailscale. This is creates a private network between devices (downside is that clients need to be installed on all devices.)

### Cloudflared-tunnel

Any services where convenience is a higher priority than security are routed through a cloudflare tunnel. This creates a persistent connection to cloudflare's edge which a domain can be pointed to and route external traffic through CGNAT.
