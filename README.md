Personal homelab project built on Oracle Cloud Always Free ARM instance (4 cores, 24GB RAM, 200GB storage).

## What I built

A production-ready Linux server configured from scratch with security hardening, containerized services and a self-hosted VPN.

## Stack

- **OS:** Ubuntu 22.04 ARM64
- **Cloud:** Oracle Cloud Infrastructure (Always Free)
- **Containers:** Docker + Portainer
- **Automation:** N8N
- **VPN:** WireGuard via wg-easy
- **Monitoring:** Dashdot

## Project Structure

| File | Description |
|------|-------------|
| [01-hardening.md](01-hardening.md) | User setup, SSH hardening, UFW firewall, Fail2ban |
| [02-docker.md](02-docker.md) | Docker installation and configuration |
| [03-servicios.md](03-servicios.md) | Portainer, N8N, Dashdot deployment |
| [04-vpn.md](04-vpn.md) | Self-hosted WireGuard VPN with wg-easy |

## Key Security Measures

- Non-root user with sudo privileges
- SSH key-only authentication (password disabled)
- Root login disabled
- UFW firewall with minimal open ports
- Fail2ban for brute-force protection

## Status

🟢 Running
