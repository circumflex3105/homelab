# Homelab

A complete, private Homelab running on a Raspberry Pi 4 via Docker Compose.

This setup is designed for Zero Trust security using Cloudflare Tunnels with no open ports.

## Architecture

- Hardware: Raspberry Pi 4 (4GB RAM) + USB3.0 SSD (128 GB)
- OS: Raspberry Pi OS Lite (64 bit)
- Networking: Cloudflare Tunnel (Zero Trust), Docker Networks
- Storage: Local SSD (Data) + Nextcloud

## Project Structure

```
.
├── adguard/             # Network-wide Ad Blocking & DNS
│   ├── work/            # Persistent working directory
│   ├── conf/            # Configuration files
│   └── docker-compose.yml
├── home-assistant/       # Smart Home Logic
│   ├── config/          # HA Configuration (automations, scenes)
│   └── docker-compose.yml
├── nextcloud/           # Private Cloud Storage
│   ├── config/          # Nextcloud config (php, www)
│   ├── data/            # (Ignored) Actual user files
│   └── docker-compose.yml
├── npm/                 # Nginx Proxy Manager (Reverse Proxy)
│   ├── data/            # (Ignored) MySQL DB & Configs
│   ├── letsencrypt/     # (Ignored) SSL Certificates
│   └── docker-compose.yml
├── tunnel/              # Network Entry Point
│   └── docker-compose.yml
├── zigbee/              # IoT Network Layer
│   ├── zigbee2mqtt/     # Zigbee to MQTT bridge
│   ├── mosquitto/       # MQTT Broker
│   └── docker-compose.yml
└── .gitignore           # Ensures no secrets are uploaded
```

## Services Overview

### 1. Home Assistant Core
The central automation controller.
- Role: Automations, Dashboard, State Management.
- Access: Accessible locally via IP and remotely via Cloudflare Tunnel.
- Integrations: TP-Link Tapo, Zigbee2MQTT, Nextcloud.

### 2. Nextcloud
A complete self-hosted cloud storage solution.
- Role: Photo backup (mobile), File sync (desktop), Calendar, Contacts.
- Stack: Nextcloud + MariaDB + Redis (for caching).
- Storage: Direct mapping to NVMe SSD for high performance.

### 3. AdGuard Home
Network-wide ad and tracker blocker.
- Role: DNS Server. Filters traffic for all devices on the network.
- Integration: Can be set as the primary DNS in the Router (Fritz!Box) to block ads on smart TVs and phones.

### 4. Nginx Proxy Manager (NPM)
A graphical Reverse Proxy interface.
- Role: Manages local SSL certificates and routes internal traffic.
- Usage: Often used alongside Cloudflare Tunnel for local-only HTTPS access or specific routing needs.

### 5. Zigbee2MQTT + Mosquitto
The local IoT network layer.
- Hardware: Sonoff Zigbee 3.0 USB Dongle Plus (Model E).
- Role: Decouples devices from the automation software. If Home Assistant restarts, the Zigbee network remains active.
- Data Flow: Zigbee Device -> Zigbee2MQTT -> MQTT Broker -> Home Assistant.

### 6. Cloudflare Tunnel (cloudflared)
The secure ingress gateway.
- Role: Exposes services to the internet without opening router ports.
- Security: Encrypted outbound tunnel preventing direct IP exposure.
- Routes:
  - home-assistant.domain.com -> Home Assistant
  - files.domain.com -> Nextcloud

## Installation & Setup

### Prerequisites
- Docker & Docker Compose installed.
- A Cloudflare account & domain.
- A supported Zigbee USB stick.

### 1. Clone the Repository

```
git clone https://github.com/circumflex3105/homelab.git
cd homelab
```

### 2. Configure Secrets
This repository uses .env files to keep secrets safe. You must create them manually from the provided examples.

### 3. Launch Services
You can launch stacks individually based on your needs:

```
# Start Network Services (DNS & Proxy)
cd ~/docker/adguard
docker compose up -d
cd ~/docker/npm
docker compose up -d

# Start Zigbee Layer
cd ~/docker/zigbee
docker compose up -d

# Start Database & Cloud
cd ~/docker/nextcloud
docker compose up -d

# Start Home Automation
cd ~/docker/homeassistant
docker compose up -d
```

## Security Notes
- No Ports Opened: The router firewall blocks all incoming connections.
- Secrets Management: All API keys, passwords, and device IDs are stored in .env files which are excluded via .gitignore.
- Network Isolation: Services run on internal Docker networks where possible.

## License
This project is for educational purposes. Feel free to fork and adapt to your own hardware.
