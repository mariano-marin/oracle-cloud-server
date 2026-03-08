# 04 — Self-Hosted VPN with WireGuard

## Objective

Deploy a personal VPN server using WireGuard via wg-easy, accessible from any device.

## Why WireGuard

WireGuard is a modern VPN protocol designed to be faster, simpler and more secure than OpenVPN or IPSec. It uses UDP instead of TCP, which reduces latency and overhead.

## Open Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| 51820 | UDP | VPN tunnel traffic |
| 51821 | TCP | Web admin panel |
```bash
sudo ufw allow 51820/udp
sudo ufw allow 51821/tcp
```

> Both ports must also be opened in Oracle Cloud Security Lists as Ingress Rules.

## Installation

### 1. Generate password hash

wg-easy v14+ requires a bcrypt hash instead of a plain text password.
```bash
docker run --rm ghcr.io/wg-easy/wg-easy wgpw 'YourPassword'
```

Copy the output hash (starts with `$2b$`).

### 2. Deploy container
```bash
docker run -d \
  --name wg-easy \
  --restart=always \
  -e WG_HOST=YOUR_SERVER_IP \
  -e PASSWORD_HASH='YOUR_BCRYPT_HASH' \
  -p 51820:51820/udp \
  -p 51821:51821/tcp \
  -v ~/.wg-easy:/etc/wireguard \
  --cap-add=NET_ADMIN \
  --cap-add=SYS_MODULE \
  --sysctl="net.ipv4.conf.all.src_valid_mark=1" \
  --sysctl="net.ipv4.ip_forward=1" \
  ghcr.io/wg-easy/wg-easy
```

> The hash must be wrapped in single quotes because it contains `$` characters.

## Admin Panel

Access: `http://SERVER_IP:51821`

From the panel you can create clients for each device (laptop, phone, etc.) and download their configuration files or scan QR codes.

## Client Setup (Mac)

1. Install WireGuard from the App Store
2. Click **+** → **Add Empty Tunnel** or import the config file downloaded from the panel
3. Activate the tunnel

## Verification

With the VPN active, visit [whatismyipaddress.com](https://whatismyipaddress.com) — it should show your server's IP and location (Virginia, USA for Oracle's Ashburn datacenter).

## Result

- Self-hosted VPN running on personal infrastructure
- Traffic routed through Oracle Cloud (Ashburn, Virginia)
- Web admin panel for managing devices
- Clients connectable from Mac, iPhone or any WireGuard-compatible device
