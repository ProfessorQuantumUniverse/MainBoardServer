# Weitere Ideen und Erweiterungen für dein HomeLab

## Übersicht
Dieses Dokument enthält erweiterte Konzepte, Projektideen und Optimierungen für dein HomeLab, die über die Basis-Installation hinausgehen.

## 1. Advanced Networking

### 1.1 VLAN Segmentierung

**Konzept:** Netzwerk in logische Segmente aufteilen für bessere Sicherheit und Performance.

```
VLAN 10 (Management): 192.168.10.0/24
├── Raspberry Pi: .1
├── Proxmox Web-UI: .20
└── Switches Management: .254

VLAN 20 (Services): 192.168.20.0/24
├── Docker Services: .10
├── VMs: .30-50
└── Kubernetes: .100-150

VLAN 30 (IoT): 192.168.30.0/24
├── Smart Home Devices
├── Kameras
└── Sensoren (kein Internet-Zugriff!)

VLAN 99 (Guest): 192.168.99.0/24
└── Gast-WLAN (isoliert vom Rest)
```

**Umsetzung:**
```bash
# Managed Switch erforderlich (z.B. TP-Link TL-SG108E)
# VLANs auf Switch konfigurieren
# Raspberry Pi als Router zwischen VLANs

# VLAN Interfaces erstellen (auf Pi)
sudo apt install vlan

# VLAN 10 (Management)
sudo ip link add link eth0 name eth0.10 type vlan id 10
sudo ip addr add 192.168.10.1/24 dev eth0.10
sudo ip link set eth0.10 up

# VLAN 20 (Services)
sudo ip link add link eth0 name eth0.20 type vlan id 20
sudo ip addr add 192.168.20.1/24 dev eth0.20
sudo ip link set eth0.20 up

# Persistent machen in /etc/network/interfaces
```

**Firewall-Regeln zwischen VLANs:**
```bash
# IoT-Geräte dürfen nicht ins Management VLAN
sudo iptables -A FORWARD -s 192.168.30.0/24 -d 192.168.10.0/24 -j DROP

# Services dürfen zu Management
sudo iptables -A FORWARD -s 192.168.20.0/24 -d 192.168.10.0/24 -m state --state NEW,ESTABLISHED -j ACCEPT

# Guest-VLAN ist komplett isoliert
sudo iptables -A FORWARD -s 192.168.99.0/24 -d 192.168.0.0/16 -j DROP
sudo iptables -A FORWARD -s 192.168.99.0/24 -d 0.0.0.0/0 -j ACCEPT
```

### 1.2 Bonding/Link Aggregation

**Wenn Mainboards mehrere Ethernet-Ports haben:**

```bash
# Bonding Module laden
sudo modprobe bonding

# Bond Interface erstellen (Mode 4 = LACP)
sudo ip link add bond0 type bond mode 802.3ad
sudo ip link set eth0 master bond0
sudo ip link set eth1 master bond0
sudo ip link set bond0 up

# Verdoppelt theoretisch die Bandbreite zu 2 Gbit/s
```

**Benötigt:** Managed Switch mit LACP/Link Aggregation Support

### 1.3 IPv6 Support

```bash
# IPv6 auf allen Hosts aktivieren
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=0

# ULA (Unique Local Address) vergeben
# Prefix: fd00::/8
# Beispiel: fd00:1234:5678::/48

# Pi-hole IPv6 aktivieren
# Web-UI → Settings → System → Enable IPv6
```

**Vorteile:**
- Vorbereitung auf IPv6-Only Netzwerke
- Mehr Adressen (keine NAT nötig)
- Besseres Routing

### 1.4 SD-WAN / Multi-WAN

**Wenn du mehrere Internet-Verbindungen hast:**

```bash
# OpenWrt/pfSense VM auf Proxmox
# oder
# mwan3 auf dem Raspberry Pi

sudo apt install mwan3

# Load-Balancing über 2+ Internet-Verbindungen
# Failover bei Ausfall einer Verbindung
# Traffic-Shaping pro Service
```

## 2. Storage und Datenmanagement

### 2.1 Network Attached Storage (NAS)

**Option A: OpenMediaVault auf Raspberry Pi**
```bash
# USB-HDD an Pi anschließen
# OpenMediaVault installieren
wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash

# Nach Installation: Web-UI auf Port 80
# Features:
# - SMB/CIFS Shares
# - NFS Shares
# - FTP/SFTP
# - RAID Management
# - Snapshots
```

**Option B: TrueNAS SCALE VM auf Proxmox**
```bash
# TrueNAS SCALE ISO herunterladen
# VM mit mindestens:
# - 8GB RAM
# - 2 vCPUs
# - Mehrere virtuelle Disks für ZFS Pool

# ZFS Features:
# - Snapshots
# - Kompression
# - Deduplizierung
# - RAID-Z
```

**Storage-Architektur:**
```
┌─────────────────────────────────────┐
│    TrueNAS VM (Proxmox)             │
│    - 4x 500GB Disks (RAID-Z1)       │
│    - Shares: /media, /backup, /data │
└────────────────┬────────────────────┘
                 │ NFS/SMB
        ┌────────┴────────┐
        │                 │
   ┌────▼────┐      ┌────▼────┐
   │ Lenovo  │      │  Dell   │
   │ Docker  │      │ Proxmox │
   └─────────┘      └─────────┘
```

### 2.2 Distributed Storage mit Ceph

**Für Multi-Node Setup (wenn du ausbaust):**
```bash
# Ceph auf allen 3 Nodes (Pi, Lenovo, Dell)
# Bietet:
# - Objekt Storage (wie S3)
# - Block Storage (für VMs)
# - File System (CephFS)
# - Replikation und Hochverfügbarkeit

# Installation auf Proxmox integriert
pveceph install
```

### 2.3 Minio Object Storage

**S3-kompatibles Storage auf Docker:**
```yaml
version: '3.8'

services:
  minio:
    image: minio/minio:latest
    container_name: minio
    restart: unless-stopped
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      MINIO_ROOT_USER: admin
      MINIO_ROOT_PASSWORD: strongpassword
    volumes:
      - minio_data:/data
    command: server /data --console-address ":9001"

volumes:
  minio_data:
```

**Use Cases:**
- Backup-Target für Restic
- Media-Storage für Apps
- Object Storage für Kubernetes
- S3-kompatible API für Development

### 2.4 Automatische Datensynchronisation

**Syncthing für Peer-to-Peer Sync:**
```bash
docker run -d \
  --name syncthing \
  --restart=always \
  -p 8384:8384 \
  -p 22000:22000/tcp \
  -p 22000:22000/udp \
  -p 21027:21027/udp \
  -v /path/to/data:/var/syncthing \
  syncthing/syncthing:latest

# Dateien zwischen allen Hosts synchronisieren
# Keine Cloud nötig
# Verschlüsselt
```

## 3. Virtualisierung und Container

### 3.1 LXD/LXC auf allen Hosts

**System Container (wie VMs aber leichtgewichtiger):**
```bash
# Auf Ubuntu-Hosts (Lenovo, Dell)
sudo snap install lxd
sudo lxd init

# Container erstellen
lxc launch ubuntu:22.04 webserver-01
lxc launch images:debian/11 database-01

# Liste Container
lxc list

# Shell in Container
lxc exec webserver-01 -- /bin/bash
```

**Vorteile gegenüber Docker:**
- Vollständiges OS
- Systemd Support
- Bessere Isolation
- Einfaches Management

### 3.2 Kubernetes Multi-Node Cluster

**Cluster-Erweiterung:**
```
Master Node: Dell (K3s Server)
Worker Node 1: Lenovo (K3s Agent)
Worker Node 2: Optional zusätzlicher Pi

# Auf Lenovo (Worker):
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1.20:6443 \
  K3S_TOKEN=[token-vom-master] sh -

# High-Availability Features:
# - Pod-Distribution über Nodes
# - Automatisches Failover
# - Load-Balancing
```

### 3.3 Docker Swarm als Alternative

**Wenn K8s zu komplex ist:**
```bash
# Auf Lenovo (Manager):
docker swarm init --advertise-addr 192.168.1.10

# Auf Dell (Worker):
docker swarm join --token [token] 192.168.1.10:2377

# Service deployen:
docker service create \
  --name nginx \
  --replicas 3 \
  --publish 80:80 \
  nginx:latest

# Einfacher als K8s, weniger Features
```

### 3.4 Nested Virtualization

**VMs in VMs (auf Proxmox):**
```bash
# CPU Flags aktivieren
qm set [vmid] --cpu host

# Ermöglicht:
# - Development von Virtualisierungs-Software
# - Testing von Hypervisors
# - Multi-Layer Isolation
```

## 4. Development und CI/CD

### 4.1 GitLab Self-Hosted

**Komplette DevOps-Platform:**
```yaml
version: '3.8'

services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    container_name: gitlab
    restart: unless-stopped
    hostname: gitlab.homelab.local
    ports:
      - "8090:80"
      - "8443:443"
      - "2222:22"
    volumes:
      - gitlab_config:/etc/gitlab
      - gitlab_logs:/var/log/gitlab
      - gitlab_data:/var/opt/gitlab
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.homelab.local:8090'
        gitlab_rails['gitlab_shell_ssh_port'] = 2222

volumes:
  gitlab_config:
  gitlab_logs:
  gitlab_data:
```

**Features:**
- Git Repository Hosting
- CI/CD Pipelines
- Container Registry
- Issue Tracking
- Wiki

### 4.2 Jenkins für CI/CD

```bash
docker run -d \
  --name jenkins \
  --restart=always \
  -p 8081:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts

# GitLab Runner für CI
docker run -d \
  --name gitlab-runner \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v gitlab-runner-config:/etc/gitlab-runner \
  gitlab/gitlab-runner:latest
```

### 4.3 Nexus Repository Manager

**Artifacts und Package Management:**
```bash
docker run -d \
  --name nexus \
  --restart=always \
  -p 8081:8081 \
  -v nexus-data:/nexus-data \
  sonatype/nexus3

# Hosted Repositories für:
# - Docker Images
# - Maven Artifacts
# - npm Packages
# - Python Packages (PyPI)
```

### 4.4 Development Environments

**Code-Server (VSCode im Browser):**
```bash
docker run -d \
  --name code-server \
  --restart=always \
  -p 8443:8080 \
  -v "$HOME/.config:/home/coder/.config" \
  -v "$HOME/projects:/home/coder/projects" \
  -e "PASSWORD=yourpassword" \
  codercom/code-server:latest

# Von überall entwickeln
# Einheitliche Umgebung
```

## 5. Security und Privacy

### 5.1 Intrusion Detection (Suricata)

```bash
# Auf Raspberry Pi (Gateway)
sudo apt install suricata

# Konfiguration
sudo nano /etc/suricata/suricata.yaml

# Aktive Monitoring aller Netzwerk-Traffic
# Alerts bei verdächtigen Aktivitäten

# Kibana/Elasticsearch für Log-Visualisierung
```

### 5.2 Vault für Secrets Management

```bash
docker run -d \
  --name vault \
  --restart=always \
  --cap-add=IPC_LOCK \
  -p 8200:8200 \
  -v vault-data:/vault/data \
  -e 'VAULT_DEV_ROOT_TOKEN_ID=myroot' \
  vault:latest

⚠️ **ACHTUNG:** Dies ist DEV-Mode! Nur für Test/Development!
Für Produktion: https://learn.hashicorp.com/tutorials/vault/getting-started-deploy

# Zentrale Passwort- und Secret-Verwaltung
# API-Keys
# Zertifikate
# Datenbank-Credentials
```

### 5.3 2FA/MFA für alle Services

**Authentik (SSO):**
```bash
# Single Sign-On für alle Services
docker run -d \
  --name authentik \
  --restart=always \
  -p 9000:9000 \
  -v authentik-data:/data \
  -e AUTHENTIK_SECRET_KEY=changeme \
  -e AUTHENTIK_ERROR_REPORTING__ENABLED=false \
  ghcr.io/goauthentik/server:latest

# Zentrales Login
# 2FA Support
# LDAP Integration
```

### 5.4 Certificate Authority (CA)

**Step-CA für eigene Zertifikate:**
```bash
# Eigene Certificate Authority
docker run -d \
  --name step-ca \
  --restart=always \
  -p 9000:9000 \
  -v step:/home/step \
  smallstep/step-ca

# Automatische SSL-Zertifikate für alle Services
# Kein Self-Signed Warning mehr
# ACME Protocol Support
```

### 5.5 Network Monitoring

**Zeek Network Security Monitor:**
```bash
# Auf Raspberry Pi
sudo apt install zeek

# Analysiert Netzwerk-Traffic
# Protokoll-Logs
# Anomalie-Erkennung
# Integration mit ELK Stack
```

## 6. Media und Entertainment

### 6.1 Plex/Jellyfin Media Server

```bash
# Jellyfin (Open Source)
docker run -d \
  --name jellyfin \
  --restart=unless-stopped \
  -p 8096:8096 \
  -v jellyfin-config:/config \
  -v /mnt/media:/media \
  jellyfin/jellyfin:latest

# Oder Plex
docker run -d \
  --name plex \
  --restart=unless-stopped \
  --network=host \
  -e TZ="Europe/Berlin" \
  -e PLEX_CLAIM="claim-token" \
  -v plex-config:/config \
  -v /mnt/media:/media \
  plexinc/pms-docker

# Media-Streaming zu allen Geräten
# Transcoding
# Mobile Apps
```

### 6.2 Sonarr/Radarr/Lidarr

**Automatisierte Media-Verwaltung:**
```yaml
version: '3.8'

services:
  sonarr:
    image: linuxserver/sonarr:latest
    container_name: sonarr
    ports:
      - "8989:8989"
    volumes:
      - sonarr-config:/config
      - /mnt/media/tv:/tv
    restart: unless-stopped

  radarr:
    image: linuxserver/radarr:latest
    container_name: radarr
    ports:
      - "7878:7878"
    volumes:
      - radarr-config:/config
      - /mnt/media/movies:/movies
    restart: unless-stopped

  lidarr:
    image: linuxserver/lidarr:latest
    container_name: lidarr
    ports:
      - "8686:8686"
    volumes:
      - lidarr-config:/config
      - /mnt/media/music:/music
    restart: unless-stopped
```

### 6.3 Calibre-Web E-Book Library

```bash
docker run -d \
  --name calibre-web \
  --restart=always \
  -p 8083:8083 \
  -v calibre-web-config:/config \
  -v /mnt/media/books:/books \
  linuxserver/calibre-web:latest

# E-Book Management
# Web-Reader
# OPDS Support (für E-Reader)
```

### 6.4 Airsonic/Navidrome Music Server

```bash
# Navidrome (leichtgewichtiger)
docker run -d \
  --name navidrome \
  --restart=unless-stopped \
  -p 4533:4533 \
  -v navidrome-data:/data \
  -v /mnt/media/music:/music:ro \
  -e ND_LOGLEVEL=info \
  deluan/navidrome:latest

# Subsonic-kompatibel
# Mobile Apps verfügbar
# Transcoding
```

## 7. Smart Home Integration

### 7.1 Home Assistant

```bash
docker run -d \
  --name homeassistant \
  --restart=unless-stopped \
  --privileged \
  --network=host \
  -v homeassistant-config:/config \
  -e TZ=Europe/Berlin \
  ghcr.io/home-assistant/home-assistant:stable

# Integration von:
# - Smart Lights (Philips Hue, IKEA)
# - Thermostate
# - Kameras
# - Sensoren
# - Automatisierungen
```

### 7.2 Node-RED für Automatisierung

```bash
docker run -d \
  --name nodered \
  --restart=unless-stopped \
  -p 1880:1880 \
  -v nodered-data:/data \
  nodered/node-red:latest

# Visual Programming
# IoT-Workflows
# Integration mit Home Assistant
# MQTT Support
```

### 7.3 Zigbee/Z-Wave Gateway

**Zigbee2MQTT:**
```bash
# USB-Dongle (CC2531 oder ConBee II) an Raspberry Pi

docker run -d \
  --name zigbee2mqtt \
  --restart=unless-stopped \
  --device=/dev/ttyACM0 \
  -p 8080:8080 \
  -v zigbee2mqtt-data:/app/data \
  -v /run/udev:/run/udev:ro \
  -e TZ=Europe/Berlin \
  koenkk/zigbee2mqtt

# Zigbee-Geräte ohne Cloud
# Lokale Kontrolle
# Integration mit Home Assistant
```

### 7.4 Frigate NVR

**AI-basierte Kamera-Überwachung:**
```yaml
version: '3.8'

services:
  frigate:
    image: ghcr.io/blakeblackshear/frigate:stable
    container_name: frigate
    restart: unless-stopped
    privileged: true
    devices:
      - /dev/bus/usb:/dev/bus/usb  # Coral TPU (optional)
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - frigate-config:/config
      - frigate-media:/media/frigate
    ports:
      - "5000:5000"
      - "1935:1935"
    environment:
      - FRIGATE_RTSP_PASSWORD=password

# Objekt-Erkennung (Personen, Autos, Tiere)
# 24/7 Recording
# Push-Benachrichtigungen
```

## 8. Machine Learning und AI

### 8.1 Jupyter Lab

```bash
docker run -d \
  --name jupyter \
  --restart=unless-stopped \
  -p 8888:8888 \
  -v jupyter-work:/home/jovyan/work \
  -e JUPYTER_ENABLE_LAB=yes \
  jupyter/datascience-notebook:latest

# Data Science Umgebung
# Python, R, Julia
# GPU-Support (mit nvidia-docker)
```

### 8.2 TensorFlow Serving

```bash
# Auf Lenovo (mehr RAM)
docker run -d \
  --name tf-serving \
  -p 8501:8501 \
  -v /models:/models \
  -e MODEL_NAME=my_model \
  tensorflow/serving

# ML-Modelle als API bereitstellen
# REST und gRPC
# Model-Versioning
```

### 8.3 Stable Diffusion WebUI

**Wenn du eine dedizierte GPU hast:**
```bash
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
cd stable-diffusion-webui

# In Docker oder direkt
./webui.sh --listen --port 7860

# AI Image Generation
# Lokal, keine Cloud
# Verschiedene Modelle
```

### 8.4 LLM Self-Hosting

**Ollama für lokale LLMs:**
```bash
docker run -d \
  --name ollama \
  --restart=unless-stopped \
  -p 11434:11434 \
  -v ollama-data:/root/.ollama \
  ollama/ollama

# Modelle herunterladen:
docker exec -it ollama ollama pull llama2
docker exec -it ollama ollama pull codellama

# Web-UI (Open WebUI):
docker run -d \
  --name open-webui \
  --restart=unless-stopped \
  -p 3000:8080 \
  -v open-webui-data:/app/backend/data \
  ghcr.io/open-webui/open-webui:main

# ChatGPT-Alternative
# Komplett privat
# Keine API-Kosten
```

## 9. Monitoring und Logging

### 9.1 ELK Stack (Elasticsearch, Logstash, Kibana)

```yaml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    container_name: logstash
    ports:
      - "5000:5000"
      - "9600:9600"
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

volumes:
  elasticsearch-data:
```

**Log-Aggregation von allen Hosts:**
```bash
# Filebeat auf jedem Host
docker run -d \
  --name filebeat \
  --restart=unless-stopped \
  --user=root \
  -v filebeat-data:/usr/share/filebeat/data \
  -v /var/log:/var/log:ro \
  -v /var/lib/docker/containers:/var/lib/docker/containers:ro \
  -e ELASTICSEARCH_HOST=192.168.1.10:9200 \
  docker.elastic.co/beats/filebeat:8.11.0
```

### 9.2 Loki + Promtail

**Alternative zu ELK (leichtgewichtiger):**
```yaml
version: '3.8'

services:
  loki:
    image: grafana/loki:latest
    container_name: loki
    ports:
      - "3100:3100"
    volumes:
      - loki-data:/loki
    command: -config.file=/etc/loki/local-config.yaml

  promtail:
    image: grafana/promtail:latest
    container_name: promtail
    volumes:
      - /var/log:/var/log:ro
      - ./promtail-config.yml:/etc/promtail/config.yml
    command: -config.file=/etc/promtail/config.yml

volumes:
  loki-data:
```

**In Grafana als Data Source:**
- URL: http://loki:3100
- Log-Queries direkt in Grafana

### 9.3 Netdata Real-Time Monitoring

```bash
# Auf jedem Host
docker run -d \
  --name netdata \
  --restart=unless-stopped \
  --pid=host \
  --network=host \
  --cap-add SYS_PTRACE \
  --security-opt apparmor=unconfined \
  -v netdataconfig:/etc/netdata \
  -v netdatalib:/var/lib/netdata \
  -v netdatacache:/var/cache/netdata \
  -v /etc/passwd:/host/etc/passwd:ro \
  -v /etc/group:/host/etc/group:ro \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  netdata/netdata

# Real-Time Metriken
# Schöne Dashboards
# Alerts
```

### 9.4 Uptime Robot Alternative (Statping)

```bash
docker run -d \
  --name statping \
  --restart=unless-stopped \
  -p 8080:8080 \
  -v statping-data:/app \
  statping/statping:latest

# Uptime-Monitoring für alle Services
# Status-Page (öffentlich oder privat)
# Notifications (Email, Discord, Slack)
```

## 10. Backup und Disaster Recovery

### 10.1 Borg Backup

```bash
# Borg installieren
sudo apt install borgbackup

# Repository initialisieren
borg init --encryption=repokey /mnt/backup/borg

# Backup-Script
cat > ~/borg-backup.sh << 'EOF'
#!/bin/bash

export BORG_REPO=/mnt/backup/borg
export BORG_PASSPHRASE='strong-passphrase'

# Backup erstellen
borg create \
  --stats \
  --compression lz4 \
  ::'{hostname}-{now}' \
  /home \
  /etc \
  /var/lib/docker/volumes

# Alte Backups löschen
borg prune \
  --keep-daily 7 \
  --keep-weekly 4 \
  --keep-monthly 6

EOF

chmod +x ~/borg-backup.sh

# Täglich um 3 Uhr
echo "0 3 * * * /home/admin/borg-backup.sh >> /var/log/borg-backup.log 2>&1" | crontab -
```

### 10.2 Offsite Backups

**Rclone zu Cloud-Provider:**
```bash
# Rclone installieren
curl https://rclone.org/install.sh | sudo bash

# Cloud-Provider konfigurieren
rclone config

# Backup verschlüsselt zu Cloud
rclone copy /mnt/backup/borg remote:homelab-backup \
  --crypt-password='encryption-password' \
  --progress

# Automatisieren mit Cron
```

**Backup zu Remote-Standort (Freund/Familie):**
```bash
# VPN-Tunnel zu Remote-Location
# Rsync über VPN:
rsync -avz --delete \
  -e "ssh -p 22" \
  /mnt/backup/ \
  user@remote-location:/backup/homelab/

# Wöchentlich ausführen
```

### 10.3 Disaster Recovery Plan

**Dokumentieren:**
```markdown
# DR-Plan

## Komplettausfall:
1. Neuen Pi besorgen
2. SD-Karte mit Backup-Image flashen
3. Backup-Recovery-Script ausführen
4. VMs aus Proxmox Backups wiederherstellen
5. Docker Volumes aus Backup mounten
6. Services neu starten

## Teilausfall (ein Host):
1. Fehler identifizieren
2. Host herunterfahren
3. Backup auf Ersatz-Hardware einspielen
4. oder temporär auf verbleibende Hosts verteilen

## Datenverlust:
1. Backup-Integrität prüfen
2. Letzten guten Backup-Punkt identifizieren
3. Restore durchführen
4. Datenverlust-Fenster akzeptieren
```

### 10.4 Hochverfügbarkeit (HA)

**Für kritische Services:**
```bash
# Keepalived für VIP (Virtual IP)
# Automatisches Failover zwischen Hosts

# Oder: K8s mit Multi-Master Setup
# Redundante Services auf mehreren Nodes
```

## 11. Performance Optimierung

### 11.1 Caching Layer

**Redis für Application Caching:**
```bash
docker run -d \
  --name redis \
  --restart=unless-stopped \
  -p 6379:6379 \
  -v redis-data:/data \
  redis:latest redis-server --appendonly yes

# Für:
# - Session Storage
# - Database Query Caching
# - Rate Limiting
```

**Varnish HTTP Cache:**
```bash
# Vor Webservern als Reverse Proxy
docker run -d \
  --name varnish \
  --restart=unless-stopped \
  -p 80:80 \
  -v ./default.vcl:/etc/varnish/default.vcl \
  varnish:latest

# Beschleunigt statische und dynamische Inhalte
```

### 11.2 Load Balancing

**HAProxy:**
```bash
docker run -d \
  --name haproxy \
  --restart=unless-stopped \
  -p 80:80 \
  -p 443:443 \
  -p 9999:9999 \
  -v ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg \
  haproxy:latest

# Verteilt Traffic auf mehrere Backends
# Health Checks
# SSL Termination
```

### 11.3 CDN mit Cloudflare (für externe Services)

```bash
# Cloudflare Tunnel (vormals Argo)
docker run -d \
  --name cloudflared \
  --restart=unless-stopped \
  cloudflare/cloudflared:latest \
  tunnel --no-autoupdate run --token [your-token]

# Vorteile:
# - Kein Port-Forwarding nötig
# - DDoS-Protection
# - CDN
# - Free SSL
```

### 11.4 Compression und Optimization

```bash
# Alle Webserver mit gzip/brotli konfigurieren
# Images automatisch komprimieren (ImageMagick)
# Logs rotieren und komprimieren

# Logrotate konfigurieren
sudo nano /etc/logrotate.d/homelab
```

```
/var/log/homelab/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 640 root adm
}
```

## 12. Dokumentation und Knowledge Base

### 12.1 WikiJS

```bash
docker run -d \
  --name wikijs \
  --restart=unless-stopped \
  -p 3000:3000 \
  -e DB_TYPE=sqlite \
  -e DB_FILEPATH=/wiki/database.sqlite \
  -v wikijs-data:/wiki/data \
  -v wikijs-db:/wiki \
  ghcr.io/requarks/wiki:2

# Technische Dokumentation
# Runbooks
# Troubleshooting-Guides
# Netzwerk-Diagramme
```

### 12.2 Bookstack

```yaml
version: '3.8'

services:
  bookstack:
    image: lscr.io/linuxserver/bookstack:latest
    container_name: bookstack
    environment:
      - PUID=1000
      - PGID=1000
      - APP_URL=http://bookstack.homelab.local
      - DB_HOST=bookstack_db
      - DB_USER=bookstack
      - DB_PASS=password
      - DB_DATABASE=bookstackapp
    ports:
      - "6875:80"
    volumes:
      - bookstack-data:/config
    restart: unless-stopped
    depends_on:
      - bookstack_db

  bookstack_db:
    image: mysql:8
    container_name: bookstack_db
    environment:
      - MYSQL_ROOT_PASSWORD=rootpass
      - MYSQL_DATABASE=bookstackapp
      - MYSQL_USER=bookstack
      - MYSQL_PASSWORD=password
    volumes:
      - bookstack-db:/var/lib/mysql
    restart: unless-stopped

volumes:
  bookstack-data:
  bookstack-db:
```

### 12.3 Network Diagramme mit draw.io

```bash
docker run -d \
  --name drawio \
  --restart=unless-stopped \
  -p 8080:8080 \
  jgraph/drawio

# Oder Lokal: https://app.diagrams.net
# Netzwerk-Topologie visualisieren
# Rack-Layout zeichnen
```

## 13. Testing und Experimenting

### 13.1 Chaos Engineering

**Testen der Ausfallsicherheit:**
```bash
# Chaoskube (für K8s)
# Tötet zufällig Pods
kubectl apply -f https://raw.githubusercontent.com/linki/chaoskube/master/deploy/chaoskube.yaml

# Manuell:
# - Services stoppen
# - Netzwerk trennen
# - Ressourcen limitieren
# - Last-Tests durchführen
```

### 13.2 Performance Testing

**Apache Bench:**
```bash
# Web-Service testen
ab -n 10000 -c 100 http://192.168.1.10/

# iperf3 für Netzwerk
iperf3 -s  # Server auf einem Host
iperf3 -c 192.168.1.10  # Client auf anderem Host
```

**Stress Testing:**
```bash
# CPU/RAM/IO belasten
sudo apt install stress-ng

stress-ng --cpu 4 --io 2 --vm 1 --vm-bytes 1G --timeout 60s
```

### 13.3 Vulnerability Scanning

**Trivy für Container:**
```bash
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image nginx:latest

# Scannt Container-Images auf Schwachstellen
```

**OpenVAS für Netzwerk:**
```bash
docker run -d \
  --name openvas \
  -p 443:443 \
  -v openvas-data:/data \
  immauss/openvas

# Vulnerability Scanner für gesamtes Netzwerk
```

## 14. Erweiterungshardware

### 14.1 Coral TPU für AI

**Google Coral USB Accelerator:**
- Anschluss an Raspberry Pi oder Lenovo
- 4 TOPS Performance
- Für Frigate, TensorFlow Lite
- ~60 EUR

### 14.2 UPS (Unterbrechungsfreie Stromversorgung)

**APC Back-UPS oder ähnliches:**
```bash
# NUT (Network UPS Tools) auf Pi
sudo apt install nut

# Konfiguration
sudo nano /etc/nut/ups.conf

[ups]
  driver = usbhid-ups
  port = auto
  desc = "HomeLab UPS"

# Services konfigurieren bei niedrigem Akku:
# - VMs herunterfahren
# - Container stoppen
# - Sauberer Shutdown
```

### 14.3 Zusätzliche Netzwerkkarten

**USB-Ethernet-Adapter für mehr Ports:**
- VLAN Interfaces
- Dedizierte Management-Ports
- 2.5G oder 10G Upgrade

### 14.4 M.2 NVMe über USB

**Externe NVMe-Gehäuse für schnellen Storage:**
- An Pi für Root-FS (statt SD-Karte)
- An Mainboards für zusätzlichen Storage
- 500 MB/s+ statt 50 MB/s (SD)

## Zusammenfassung und Prioritäten

### Immediate Next Steps:
1. ✅ Hardware aufbauen
2. ✅ Basis-OS installieren
3. ⚡ Monitoring (Grafana + Prometheus)
4. ⚡ Backup-System (Borg oder Restic)
5. 🔒 Security Hardening (Firewall, 2FA)

### Short Term (1-3 Monate):
- Media-Server (Jellyfin/Plex)
- NAS-Lösung (OMV oder TrueNAS)
- CI/CD Pipeline (GitLab oder Jenkins)
- VPN für Remote-Access (WireGuard)

### Medium Term (3-6 Monate):
- Kubernetes Cluster erweitern
- Smart Home Integration
- Log-Aggregation (ELK oder Loki)
- Advanced Networking (VLANs)

### Long Term (6-12 Monate):
- Machine Learning Projekte
- Distributed Storage (Ceph)
- High Availability Setup
- Community/Learning Projects

## Lern-Ressourcen

### Online-Kurse:
- **Udemy:** Docker, Kubernetes, Linux
- **Plural Sight:** DevOps, Networking
- **YouTube:** TechnoTim, NetworkChuck, Jeff Geerling

### Communities:
- **Reddit:** r/homelab, r/selfhosted
- **Discord:** HomelaHub Discord Server
- **Forum:** ServeTheHome Forum

### Dokumentation:
- Docker Docs
- Kubernetes Docs
- Proxmox Wiki
- Raspberry Pi Forums

---

**Viel Erfolg mit deinem HomeLab! 🚀**

*Dokumentiere alles, experimentiere viel, und hab Spaß beim Lernen!*
