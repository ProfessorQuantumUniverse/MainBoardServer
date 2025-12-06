# MainBoardServer - HomeLab Projekt

Willkommen zu meinem HomeLab-Projekt! Dieses Repository dokumentiert den Aufbau eines vollständigen HomeLabs mit zwei Laptop-Mainboards (Lenovo und Dell) und einem Raspberry Pi.

## 📚 Dokumentation

### 🔧 [Rack-Aufbau.md](Rack-Aufbau.md)
Detaillierte Anleitung zum physischen Aufbau des Racks:
- IKEA Ivar Regal / Schwerlastregal Setup
- 3D-gedruckte Halterungen und Komponenten
- TP-Link 5-Port Ethernet Switch Integration
- Kabelmanagement und Lüfter-Installation
- Schritt-für-Schritt Montage-Anleitung

### 💻 [Installation.md](Installation.md)
Vollständige Software-Installation und Konfiguration:
- Betriebssystem-Setup (Ubuntu Server, Proxmox, Raspberry Pi OS)
- Docker und Container-Orchestrierung
- Netzwerk-Konfiguration (VLANs, Firewall, VPN)
- Monitoring-Stack (Prometheus, Grafana)
- Service-Deployments (Portainer, Pi-hole, etc.)
- Backup und Recovery-Strategien

### 💡 [Weitere-Ideen.md](Weitere-Ideen.md)
Erweiterte Konzepte und Projektideen:
- Advanced Networking (VLANs, Bonding, SD-WAN)
- Storage-Lösungen (NAS, Ceph, Minio)
- Virtualisierung und Kubernetes
- CI/CD Pipeline (GitLab, Jenkins)
- Security und Privacy (Vault, Authentik, IDS)
- Media-Server (Jellyfin, Plex)
- Smart Home Integration (Home Assistant)
- Machine Learning und AI (Jupyter, LLMs)
- Monitoring und Logging (ELK Stack, Loki)

### 📋 [Hardware-Spezifikationen.md](Hardware-Spezifikationen.md)
Technische Details und Hardware-Informationen:
- Laptop-Mainboard Spezifikationen (Lenovo & Dell)
- Raspberry Pi technische Daten
- Netzwerk-Hardware (Switches, Kabel)
- Stromversorgung und USV
- Kühlung und Thermal Management
- Kompatibilitäts-Guide
- Einkaufsliste mit Preisen (Deutschland, 2024)
- Wartungsplan

## 🎯 Projekt-Übersicht

### Hardware-Komponenten:
- **2x Laptop Mainboards** (Lenovo & Dell)
  - Docker Host (Lenovo)
  - Proxmox/Kubernetes Node (Dell)
- **1x Raspberry Pi 4** (Gateway & Management)
- **TP-Link 5-Port Gigabit Switch**
- **IKEA Ivar Regal** als Rack-Basis
- **3D-gedruckte Halterungen** für alle Komponenten

### Software-Stack:
- **Operating Systems:** Ubuntu Server, Proxmox VE, Raspberry Pi OS
- **Container:** Docker, Docker Compose, Portainer
- **Networking:** Pi-hole (DNS), WireGuard (VPN), UFW (Firewall)
- **Monitoring:** Prometheus, Grafana, Node Exporter
- **Optional:** Kubernetes (K3s), Home Assistant, Media Server

### Architektur:
```
                    Internet
                       │
                   [Router]
                       │
              ┌────────┴────────┐
              │  TP-Link Switch │
              └─┬──────┬──────┬─┘
                │      │      │
       ┌────────┴──┐ ┌┴──────┴────┐
       │ Raspberry │ │   Lenovo    │
       │    Pi     │ │  Mainboard  │
       │ (Gateway) │ │   (Docker)  │
       └───────────┘ └─────────────┘
       192.168.1.1    192.168.1.10
                       
                      ┌──────────────┐
                      │     Dell     │
                      │  Mainboard   │
                      │(Proxmox/K8s) │
                      └──────────────┘
                      192.168.1.20
```

## 🚀 Quick Start

1. **Hardware aufbauen:** Folge der [Rack-Aufbau.md](Rack-Aufbau.md) Anleitung
2. **Software installieren:** Siehe [Installation.md](Installation.md) für detaillierte Schritte
3. **Services deployen:** Wähle Services aus [Weitere-Ideen.md](Weitere-Ideen.md)
4. **Erweitern:** Nutze die [Hardware-Spezifikationen.md](Hardware-Spezifikationen.md) für Upgrades

## 📊 Geschätzte Kosten

- **Basis-Setup:** ~500 EUR (ohne bereits vorhandene Mainboards)
- **Mit Extras:** ~865 EUR
- **Stromkosten:** ~22-29 EUR/Monat (bei 24/7 Betrieb in DE)

Siehe [Hardware-Spezifikationen.md](Hardware-Spezifikationen.md) für detaillierte Preisliste.

## 🔒 Sicherheitshinweise

⚠️ **Wichtig:**
- Niemals an stromführenden Teilen arbeiten
- Überspannungsschutz ist Pflicht
- Ausreichende Belüftung sicherstellen
- Regelmäßige Backups durchführen
- Firewall auf allen Hosts aktivieren

## 📝 Lizenz

Dieses Projekt ist unter der MIT-Lizenz veröffentlicht. Siehe [LICENSE](LICENSE) für Details.

## 🤝 Beitragen

Verbesserungsvorschläge und Pull Requests sind willkommen! 

## 📧 Kontakt

Bei Fragen oder Anregungen, öffne gerne ein Issue in diesem Repository.

---

**Status:** 🚧 In Arbeit | **Sprache:** 🇩🇪 Deutsch | **Jahr:** 2024
