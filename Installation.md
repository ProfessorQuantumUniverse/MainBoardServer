# HomeLab Installation Guide - Software und Konfiguration

## Übersicht
Dieser detaillierte Guide beschreibt die komplette Software-Installation und Konfiguration für dein HomeLab mit zwei Laptop-Mainboards (Lenovo & Dell) und einem Raspberry Pi.

## Architektur-Übersicht

```
┌─────────────────────────────────────────────────────┐
│                 Internet / Router                    │
└────────────────────┬────────────────────────────────┘
                     │
            ┌────────▼────────┐
            │   TP-Link 5P    │  (192.168.1.0/24)
            │     Switch      │
            └─┬──────┬──────┬─┘
              │      │      │
     ┌────────▼──┐ ┌▼──────▼────┐ ┌──────────────┐
     │ Raspberry │ │   Lenovo    │ │     Dell     │
     │    Pi     │ │  Mainboard  │ │  Mainboard   │
     │ (Gateway) │ │  (Docker)   │ │ (Proxmox/K8s)│
     └───────────┘ └─────────────┘ └──────────────┘
     192.168.1.1    192.168.1.10    192.168.1.20
```

## Hardware-Voraussetzungen

### Minimale Spezifikationen pro Mainboard:
- **CPU:** Dual-Core (Intel i3/i5 oder AMD äquivalent)
- **RAM:** Mindestens 8GB (16GB empfohlen)
- **Storage:** 128GB SSD minimum (256GB+ empfohlen)
- **Netzwerk:** Gigabit Ethernet

### Raspberry Pi Anforderungen:
- **Modell:** 2x Raspberry Pi 5 (1x 4GB + 1x 8GB)
- **Storage:** 32GB+ microSD Class 10 (A2 empfohlen)
- **Zusätzlich:** Optional USB-SSD für bessere Performance

## Phase 1: Betriebssystem Installation

### 1.1 Raspberry Pi Setup (Gateway & Management)

#### OS Installation: Raspberry Pi OS Lite (64-bit)

**Mit Raspberry Pi Imager:**
```bash
# Auf deinem Laptop/Desktop:
1. Raspberry Pi Imager herunterladen und installieren
2. Image auswählen: "Raspberry Pi OS Lite (64-bit)"
3. Erweiterte Optionen (Zahnrad-Icon):
   - Hostname: homelab-gateway
   - SSH aktivieren (Passwort oder Key)
   - Username: admin
   - WiFi optional konfigurieren (als Fallback)
   - Locale: de_DE.UTF-8, Timezone: Europe/Berlin
   - Keyboard: German
4. Auf microSD schreiben
```

#### Erste Schritte nach Boot:

```bash
# Per SSH verbinden (Default-IP vom Router erfragen)
ssh admin@192.168.1.xxx

# System updaten
sudo apt update && sudo apt upgrade -y

# Statische IP konfigurieren
sudo nano /etc/dhcpcd.conf

# Folgendes hinzufügen:
interface eth0
static ip_address=192.168.1.1/24
static routers=192.168.1.254
static domain_name_servers=1.1.1.1 8.8.8.8

# Reboot
sudo reboot

# Nach Reboot erneut verbinden mit neuer IP
ssh admin@192.168.1.1
```

#### Basis-Pakete installieren:

```bash
sudo apt install -y \
  vim \
  git \
  curl \
  wget \
  htop \
  tmux \
  python3-pip \
  python3-venv \
  docker.io \
  docker-compose \
  fail2ban \
  ufw \
  net-tools \
  dnsutils \
  iperf3
```

#### Docker für Pi optimieren:

```bash
# User zu docker-Gruppe hinzufügen
sudo usermod -aG docker $USER

# Docker beim Boot starten
sudo systemctl enable docker
sudo systemctl start docker

# Docker Compose Plugin
sudo apt install docker-compose-plugin

# Test
docker --version
docker compose version
```

### 1.2 Lenovo Mainboard Setup (Docker Host)

#### OS: Ubuntu Server 22.04 LTS

**Installation via USB-Stick:**
```bash
# Auf deinem Laptop:
1. Ubuntu Server 22.04 LTS ISO herunterladen
2. Mit Rufus/Etcher auf USB-Stick schreiben
3. USB an Lenovo Board, von USB booten

# Während Installation:
- Sprache: Deutsch
- Keyboard: Deutsch
- Netzwerk: DHCP (wird später statisch)
- Storage: gesamte SSD verwenden (LVM optional)
- Profil:
  * Name: admin
  * Server Name: homelab-docker
  * Username: admin
  * Password: [Sicheres Passwort!]
- SSH Server: Installieren
- Keine zusätzlichen Snaps
```

#### Nach der Installation:

```bash
# SSH-Verbindung vom Pi oder Laptop
ssh admin@[lenovo-ip]

# System update
sudo apt update && sudo apt full-upgrade -y

# Statische IP setzen
sudo nano /etc/netplan/00-installer-config.yaml

# Inhalt:
network:
  version: 2
  ethernets:
    [interface-name]:
      dhcp4: no
      addresses:
        - 192.168.1.10/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [192.168.1.1, 1.1.1.1]

# Anwenden
sudo netplan apply

# SSH neu verbinden
ssh admin@192.168.1.10
```

#### Docker Installation:

```bash
# Docker's offizielle Repository hinzufügen
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Docker installieren
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# User zu docker-Gruppe
sudo usermod -aG docker $USER

# Docker aktivieren
sudo systemctl enable docker
sudo systemctl start docker

# Logout/Login für Gruppenmitgliedschaft
exit
# Erneut einloggen
ssh admin@192.168.1.10

# Test
docker run hello-world
```

#### System-Optimierungen:

```bash
# Swap für bessere Performance (optional bei wenig RAM)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Kernel-Parameter optimieren
sudo nano /etc/sysctl.conf

# Hinzufügen:
vm.swappiness=10
vm.vfs_cache_pressure=50
net.core.rmem_max=134217728
net.core.wmem_max=134217728
net.ipv4.tcp_rmem=4096 87380 67108864
net.ipv4.tcp_wmem=4096 65536 67108864

# Anwenden
sudo sysctl -p
```

### 1.3 Dell Mainboard Setup (Proxmox oder Kubernetes)

#### Option A: Proxmox VE (Virtualisierung)

**Installation:**
```bash
# Proxmox VE ISO auf USB schreiben
# Von USB booten am Dell Board

# Installation:
- Sprache: Deutsch
- Target Disk: Komplette SSD
- Timezone: Europe/Berlin
- Password: [Sicheres Root-Passwort]
- Email: deine@email.de
- Hostname: homelab-proxmox.local
- IP: 192.168.1.20/24
- Gateway: 192.168.1.1
- DNS: 192.168.1.1

# Nach Installation: Reboot
# Web-UI erreichbar unter: https://192.168.1.20:8006
```

**Proxmox Post-Installation:**
```bash
# Via SSH verbinden
ssh root@192.168.1.20

# Enterprise Repository deaktivieren (keine Lizenz)
nano /etc/apt/sources.list.d/pve-enterprise.list
# Zeile auskommentieren mit #

# Community Repository aktivieren
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list

# Update
apt update && apt full-upgrade -y

# Nützliche Tools
apt install -y vim git curl htop
```

#### Option B: Kubernetes Node (K3s - leichtgewichtig)

```bash
# Ubuntu Server 22.04 LTS installieren (wie bei Lenovo)
# Statische IP: 192.168.1.20/24
# Hostname: homelab-k8s

# Nach OS-Installation:
ssh admin@192.168.1.20

# System vorbereiten
sudo apt update && sudo apt upgrade -y

# K3s installieren
curl -sfL https://get.k3s.io | sh -s - \
  --write-kubeconfig-mode 644 \
  --node-ip 192.168.1.20 \
  --node-external-ip 192.168.1.20

# Status prüfen
sudo systemctl status k3s

# Kubectl prüfen
kubectl get nodes
kubectl get pods -A

# Kubeconfig für externe Verwaltung
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
```

## Phase 2: Netzwerk und Sicherheit

### 2.1 Pi-hole DNS Installation (auf Raspberry Pi)

```bash
# Auf Raspberry Pi
ssh admin@192.168.1.1

# Pi-hole Installation
curl -sSL https://install.pi-hole.net | bash

# Während Installation:
- Upstream DNS: Cloudflare (1.1.1.1)
- Blocklists: Standard aktivieren
- Admin Interface: Ja
- Web Server: lighttpd
- Logging: An
- Privacy Mode: Show Everything (für HomeLab ok)

# Admin-Passwort setzen
pihole -a -p

# Web-Interface: http://192.168.1.1/admin
```

**Pi-hole konfigurieren:**
```bash
# Zusätzliche Blocklists hinzufügen (im Web-UI):
- https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
- https://v.firebog.net/hosts/static/w3kbl.txt
- https://dbl.oisd.nl/

# Lokale DNS-Einträge (im Web-UI unter Local DNS):
192.168.1.1     homelab-gateway.local
192.168.1.10    homelab-docker.local
192.168.1.20    homelab-proxmox.local (oder homelab-k8s.local)

# Gravity Update (Blocklists aktualisieren)
pihole -g
```

### 2.2 Firewall Konfiguration

#### Raspberry Pi (UFW):
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH
sudo ufw allow 22/tcp

# DNS
sudo ufw allow 53/tcp
sudo ufw allow 53/udp

# HTTP/HTTPS (Pi-hole Admin)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# WireGuard (falls installiert)
sudo ufw allow 51820/udp

# Aktivieren
sudo ufw enable

# Status
sudo ufw status verbose
```

#### Lenovo Docker Host:
```bash
ssh admin@192.168.1.10

sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH
sudo ufw allow from 192.168.1.0/24 to any port 22

# Docker Ports (je nach Services)
sudo ufw allow 8080/tcp  # Beispiel: Portainer
sudo ufw allow 9000/tcp  # Beispiel: Weitere Services

# Aktivieren
sudo ufw enable
```

#### Dell Proxmox/K8s:
```bash
ssh root@192.168.1.20  # oder admin bei K8s

# Proxmox Web-UI
ufw allow from 192.168.1.0/24 to any port 8006

# K8s API Server
ufw allow from 192.168.1.0/24 to any port 6443

# Standard Rules
ufw default deny incoming
ufw default allow outgoing
ufw allow from 192.168.1.0/24 to any port 22

ufw enable
```

### 2.3 VPN Setup (WireGuard auf Raspberry Pi)

```bash
# Auf Raspberry Pi
ssh admin@192.168.1.1

# WireGuard installieren
sudo apt install -y wireguard

# Keys generieren
wg genkey | sudo tee /etc/wireguard/private.key
sudo chmod 600 /etc/wireguard/private.key
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key

# Server-Konfiguration
sudo nano /etc/wireguard/wg0.conf

# Inhalt:
[Interface]
PrivateKey = <DEIN_PRIVATE_KEY_HIER_EINFÜGEN>
Address = 10.0.0.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

# Client-Konfiguration (Beispiel)
[Peer]
PublicKey = <CLIENT_PUBLIC_KEY_HIER_EINFÜGEN>
AllowedIPs = 10.0.0.2/32

# IP-Forwarding aktivieren
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# WireGuard starten
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0

# Status
sudo wg show
```

## Phase 3: Container und Services

### 3.1 Portainer (Docker GUI auf Lenovo)

```bash
ssh admin@192.168.1.10

# Portainer installieren
docker volume create portainer_data

docker run -d \
  --name portainer \
  --restart=always \
  -p 8000:8000 \
  -p 9443:9443 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest

# Web-UI: https://192.168.1.10:9443
# Beim ersten Start: Admin-Account erstellen
```

### 3.2 Monitoring Stack (Prometheus + Grafana auf Lenovo)

**Docker Compose erstellen:**
```bash
mkdir -p ~/monitoring
cd ~/monitoring
nano docker-compose.yml
```

```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    ports:
      - "9090:9090"
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=<SICHERES_PASSWORT_HIER>
      - GF_USERS_ALLOW_SIGN_UP=false
    ports:
      - "3000:3000"
    networks:
      - monitoring
    depends_on:
      - prometheus

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    ports:
      - "9100:9100"
    networks:
      - monitoring

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    restart: unless-stopped
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8080:8080"
    networks:
      - monitoring
    privileged: true

volumes:
  prometheus_data:
  grafana_data:

networks:
  monitoring:
    driver: bridge
```

**Prometheus Konfiguration:**
```bash
nano prometheus.yml
```

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
        labels:
          host: 'homelab-docker'

  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']

  # Raspberry Pi Node Exporter (separat zu installieren)
  - job_name: 'raspberry-pi'
    static_configs:
      - targets: ['192.168.1.1:9100']
        labels:
          host: 'homelab-gateway'

  # Proxmox Exporter (optional)
  - job_name: 'proxmox'
    static_configs:
      - targets: ['192.168.1.20:9221']
        labels:
          host: 'homelab-proxmox'
```

**Stack starten:**
```bash
docker compose up -d

# Logs prüfen
docker compose logs -f

# Services erreichbar:
# Prometheus: http://192.168.1.10:9090
# Grafana: http://192.168.1.10:3000 (admin/admin)
```

**Grafana konfigurieren:**
```
1. Login: http://192.168.1.10:3000 (admin/admin)
2. Data Source hinzufügen:
   - Type: Prometheus
   - URL: http://prometheus:9090
   - Save & Test
3. Dashboards importieren:
   - ID 1860: Node Exporter Full
   - ID 893: Docker & System Monitoring
   - ID 14282: cadvisor exporter
```

### 3.3 Weitere nützliche Services auf Lenovo

#### Nginx Proxy Manager (Reverse Proxy):
```bash
cd ~/nginx-proxy
nano docker-compose.yml
```

```yaml
version: '3.8'

services:
  nginx-proxy:
    image: jc21/nginx-proxy-manager:latest
    container_name: nginx-proxy-manager
    restart: unless-stopped
    ports:
      - "80:80"    # HTTP
      - "443:443"  # HTTPS
      - "81:81"    # Admin UI
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    environment:
      - DB_SQLITE_FILE=/data/database.sqlite
```

```bash
docker compose up -d

# Web-UI: http://192.168.1.10:81
# Default Login: admin@example.com / changeme
```

#### Uptime Kuma (Monitoring):
```bash
docker run -d \
  --name uptime-kuma \
  --restart=always \
  -p 3001:3001 \
  -v uptime-kuma:/app/data \
  louislam/uptime-kuma:1

# Web-UI: http://192.168.1.10:3001
```

#### Homer Dashboard (Startseite):
```bash
mkdir -p ~/homer
cd ~/homer

docker run -d \
  --name homer \
  --restart=always \
  -p 8888:8080 \
  -v $(pwd)/assets:/www/assets \
  b4bz/homer:latest

# Config erstellen
mkdir -p assets
nano assets/config.yml
```

```yaml
---
title: "HomeLab Dashboard"
subtitle: "Willkommen"
logo: "logo.png"

header: true
footer: false

columns: "3"

services:
  - name: "Management"
    icon: "fas fa-cog"
    items:
      - name: "Portainer"
        logo: "https://portainer.io/images/logo.png"
        subtitle: "Docker Management"
        url: "https://192.168.1.10:9443"
        target: "_blank"
      - name: "Proxmox"
        logo: "https://www.proxmox.com/images/proxmox/Proxmox_logo_standard_hex_400px.png"
        subtitle: "Virtualisierung"
        url: "https://192.168.1.20:8006"
        target: "_blank"

  - name: "Monitoring"
    icon: "fas fa-chart-line"
    items:
      - name: "Grafana"
        subtitle: "Metriken"
        url: "http://192.168.1.10:3000"
        target: "_blank"
      - name: "Uptime Kuma"
        subtitle: "Status"
        url: "http://192.168.1.10:3001"
        target: "_blank"

  - name: "Network"
    icon: "fas fa-network-wired"
    items:
      - name: "Pi-hole"
        subtitle: "DNS & Ad-Blocking"
        url: "http://192.168.1.1/admin"
        target: "_blank"
      - name: "Router"
        subtitle: "Netzwerk"
        url: "http://192.168.1.254"
        target: "_blank"
```

```bash
# Dashboard: http://192.168.1.10:8888
```

## Phase 4: Proxmox VMs und LXC Container

### 4.1 VMs erstellen (auf Dell/Proxmox)

**Beispiel: Ubuntu VM für Testing:**
```bash
# Via Web-UI (https://192.168.1.20:8006):

1. ISO-Images hochladen:
   - Datacenter → Storage → local → ISO Images → Upload
   - Ubuntu Server 22.04 LTS ISO hochladen

2. VM erstellen:
   - Rechtsklick auf Node → Create VM
   - General:
     * VM ID: 100
     * Name: test-vm-01
   - OS:
     * ISO: Ubuntu-22.04-live-server-amd64.iso
   - System:
     * QEMU Agent: Aktivieren
   - Disks:
     * Size: 32GB
   - CPU:
     * Cores: 2
   - Memory:
     * RAM: 4096MB
   - Network:
     * Bridge: vmbr0
     * Firewall: Aktivieren

3. VM starten und via Console installieren
4. Nach Installation: QEMU Guest Agent installieren
   sudo apt install qemu-guest-agent
   sudo systemctl enable qemu-guest-agent
   sudo systemctl start qemu-guest-agent
```

### 4.2 LXC Container (leichtgewichtiger)

**Beispiel: Container für Webserver:**
```bash
# Via Web-UI:

1. Container Template herunterladen:
   - Datacenter → Storage → local → CT Templates
   - Templates → Search: ubuntu
   - Download: ubuntu-22.04-standard

2. Container erstellen:
   - Rechtsklick auf Node → Create CT
   - General:
     * CT ID: 200
     * Hostname: webserver-01
     * Password: [Sicheres Passwort]
   - Template:
     * ubuntu-22.04-standard
   - Root Disk:
     * Size: 8GB
   - CPU:
     * Cores: 1
   - Memory:
     * RAM: 1024MB
     * Swap: 512MB
   - Network:
     * IPv4: DHCP oder Static (z.B. 192.168.1.30/24)
     * Gateway: 192.168.1.1

3. Container starten
4. Console öffnen und einloggen
```

## Phase 5: Kubernetes Deployments (auf Dell/K3s)

### 5.1 Erste Deployment: Nginx

```bash
ssh admin@192.168.1.20

# Namespace erstellen
kubectl create namespace web

# Deployment
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: web
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
EOF

# Status prüfen
kubectl get deployments -n web
kubectl get pods -n web
kubectl get services -n web

# Nginx erreichbar: http://192.168.1.20:30080
```

### 5.2 Persistent Storage

```bash
# PersistentVolume erstellen
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
  storageClassName: local-storage
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
  namespace: web
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: local-storage
EOF
```

## Phase 6: Backup und Recovery

### 6.1 Automatische Backups auf Raspberry Pi

```bash
ssh admin@192.168.1.1

# Backup-Verzeichnis
sudo mkdir -p /mnt/backup
sudo chown admin:admin /mnt/backup

# Backup-Script erstellen
nano ~/backup.sh
```

```bash
#!/bin/bash

BACKUP_DIR="/mnt/backup"
DATE=$(date +%Y%m%d_%H%M%S)

# Docker Container Backups vom Lenovo
ssh admin@192.168.1.10 "docker ps -a --format '{{.Names}}' | while read container; do \
  docker export \$container > /tmp/\${container}_${DATE}.tar; done"

scp admin@192.168.1.10:/tmp/*_${DATE}.tar ${BACKUP_DIR}/

# Proxmox VMs Backup (vzdump)
ssh root@192.168.1.20 "vzdump --all --mode snapshot --compress gzip --storage local"

# Pi-hole Config
sudo tar -czf ${BACKUP_DIR}/pihole_${DATE}.tar.gz /etc/pihole/ /etc/dnsmasq.d/

# Alte Backups löschen (älter als 30 Tage)
find ${BACKUP_DIR} -type f -mtime +30 -delete

echo "Backup completed: ${DATE}"
```

```bash
chmod +x ~/backup.sh

# Cron Job für wöchentliche Backups (Sonntags 2 Uhr)
crontab -e

# Hinzufügen:
0 2 * * 0 /home/admin/backup.sh >> /var/log/backup.log 2>&1
```

### 6.2 Restic für inkrementelle Backups (optional)

```bash
# Restic installieren (auf allen Hosts)
sudo apt install restic

# Repository initialisieren (auf Pi)
sudo restic init --repo /mnt/backup/restic

# Backup-Script mit Restic
nano ~/restic-backup.sh
```

```bash
#!/bin/bash

export RESTIC_REPOSITORY="/mnt/backup/restic"
export RESTIC_PASSWORD="[Sicheres Passwort]"

# Backup wichtiger Verzeichnisse
restic backup /home/admin
restic backup /etc

# Alte Snapshots bereinigen
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --prune

echo "Restic backup completed"
```

## Phase 7: Monitoring und Alerts

### 7.1 Node Exporter auf allen Hosts

**Auf Raspberry Pi:**
```bash
ssh admin@192.168.1.1

docker run -d \
  --name node-exporter \
  --restart=always \
  --net="host" \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  prom/node-exporter:latest \
  --path.rootfs=/host
```

**Auf Dell (wenn K8s):**
```bash
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/kube-prometheus/main/manifests/setup/prometheus-operator-0servicemonitorCustomResourceDefinition.yaml
```

### 7.2 Alerting mit Alertmanager

```bash
# Auf Lenovo (Monitoring Stack erweitern)
cd ~/monitoring

nano alertmanager.yml
```

```yaml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'email'

receivers:
  - name: 'email'
    email_configs:
      - to: 'deine@email.de'
        from: 'alertmanager@homelab.local'
        smarthost: 'smtp.gmail.com:587'
        auth_username: 'deine@email.de'
        auth_password: 'app-password'

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname']
```

**Docker Compose erweitern:**
```yaml
  alertmanager:
    image: prom/alertmanager:latest
    container_name: alertmanager
    restart: unless-stopped
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    ports:
      - "9093:9093"
    networks:
      - monitoring
```

## Troubleshooting

### Häufige Probleme:

**1. Netzwerk-Konnektivität:**
```bash
# Ping-Test
ping 192.168.1.1
ping 8.8.8.8

# DNS-Test
nslookup google.com 192.168.1.1

# Port-Scan
nc -zv 192.168.1.10 9443
```

**2. Docker-Container starten nicht:**
```bash
# Logs prüfen
docker logs [container-name]

# Container neu starten
docker restart [container-name]

# Volumes prüfen
docker volume ls
docker volume inspect [volume-name]
```

**3. Hohe Temperaturen:**
```bash
# Temperaturen auslesen (Raspberry Pi)
vcgencmd measure_temp

# Temperaturen auslesen (Linux)
sensors

# Lüfter-Geschwindigkeit prüfen
cat /sys/class/hwmon/hwmon*/fan*_input
```

**4. Proxmox VMs Performance:**
```bash
# Ballooning prüfen
qm config [vmid]

# IO-Scheduler optimieren
cat /sys/block/sda/queue/scheduler
echo "noop" > /sys/block/sda/queue/scheduler
```

## Nächste Schritte

Nach erfolgreicher Installation:
1. → Siehe `Weitere-Ideen.md` für Erweiterungen
2. → Siehe `Hardware-Spezifikationen.md` für Details
3. Regelmäßige Updates durchführen
4. Monitoring-Dashboards anpassen
5. Backups testen!

## Sicherheitshinweise

⚠️ **Best Practices:**
- Regelmäßige Updates aller Systeme
- Starke, einzigartige Passwörter
- SSH nur mit Key-Auth (Passwort deaktivieren)
- Firewall auf allen Hosts aktiv
- Backup-Strategie testen
- Monitoring und Alerting aktiv
- Nur notwendige Ports öffnen
- VPN für externen Zugriff nutzen
