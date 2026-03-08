# 03 — Self-Hosted Services

## Objective

Deploy containerized services using Docker on the hardened server.

## Open Ports Reference

| Service | Port | Protocol |
|---------|------|----------|
| Portainer | 9000 | TCP |
| N8N | 5678 | TCP |
| Dashdot | 3001 | TCP |

> Each port must be opened both in UFW and in Oracle Cloud Security Lists.

## 1. Portainer

Visual Docker management interface.
```bash
docker volume create portainer_data

docker run -d \
  -p 9000:9000 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce
```
```bash
sudo ufw allow 9000/tcp
```

Access: `http://SERVER_IP:9000`

## 2. N8N

Workflow automation platform.
```bash
docker run -d \
  --name n8n \
  --restart=always \
  -p 5678:5678 \
  -e N8N_SECURE_COOKIE=false \
  -v n8n_data:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
```
```bash
sudo ufw allow 5678/tcp
```

> `N8N_SECURE_COOKIE=false` is required when accessing via HTTP without a domain. For production use, configure HTTPS with a reverse proxy.

Access: `http://SERVER_IP:5678`

## 3. Dashdot

Real-time server monitoring dashboard.
```bash
docker run -d \
  --name dashdot \
  --restart=always \
  -p 3001:3001 \
  --privileged \
  mauricenino/dashdot
```
```bash
sudo ufw allow 3001/tcp
```

Access: `http://SERVER_IP:3001`

## Result

- Portainer: visual Docker management
- N8N: workflow automation ready to use
- Dashdot: real-time CPU, RAM, disk and network monitoring
