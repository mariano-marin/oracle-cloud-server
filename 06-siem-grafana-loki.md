# 06 — SIEM with Grafana + Loki

## Objective

Build a lightweight SIEM (Security Information and Event Management) system to collect, store and visualize server logs in real time.

## Stack

| Component | Role |
|-----------|------|
| Loki | Log aggregation and storage |
| Promtail | Log collector agent |
| Grafana | Visualization and dashboards |

## Installation

Create config directory:
```bash
mkdir -p /home/mariano/loki
sudo chown -R mariano:mariano /home/mariano/loki
```

Create Promtail config:
```bash
cat > /home/mariano/loki/promtail-config.yaml << 'EOF'
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*log

  - job_name: docker
    static_configs:
      - targets:
          - localhost
        labels:
          job: docker
          __path__: /var/lib/docker/containers/*/*log
EOF
```

Create Docker Compose file:
```bash
cat > /home/mariano/loki/docker-compose.yaml << 'EOF'
version: "3"

services:
  loki:
    image: grafana/loki:latest
    container_name: loki
    restart: always
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml

  promtail:
    image: grafana/promtail:latest
    container_name: promtail
    restart: always
    volumes:
      - /var/log:/var/log:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /home/mariano/loki/promtail-config.yaml:/etc/promtail/config.yaml
    command: -config.file=/etc/promtail/config.yaml

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: always
    ports:
      - "3030:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana

volumes:
  grafana_data:
EOF
```

Deploy:
```bash
cd /home/mariano/loki
sudo docker compose up -d
```

## Open Ports
```bash
sudo ufw allow 3030/tcp
sudo ufw allow 3100/tcp
```

Add both ports as TCP Ingress Rules in Oracle Cloud Security Lists.

## Grafana Setup

1. Access Grafana at `http://SERVER_IP:3030`
2. Go to **Connections** → **Data sources** → **Add data source** → **Loki**
3. Set URL to `http://loki:3100`
4. Click **Save & test**

## Security Dashboard

Created a `Security Overview` dashboard with 4 panels:

| Panel | Query | Type |
|-------|-------|------|
| Invalid Login Attempts | `{job="varlogs"} \|= "Invalid user"` | Logs |
| Root Login Attempts | `{job="varlogs"} \|= "user root"` | Logs |
| Successful SSH Logins | `{job="varlogs"} \|= "Accepted publickey"` | Logs |
| Failed Logins (last hour) | `sum(count_over_time({job="varlogs"} \|= "Invalid user" [1h]))` | Stat |

## What This Shows

Within hours of the server being online, automated bots were already attempting to access it — trying common usernames like `root`, `admin`, `dspace` from IPs across the world. All attempts were blocked thanks to SSH hardening and Fail2ban configured in [01-hardening.md](01-hardening.md).

## Access

`http://SERVER_IP:3030`
