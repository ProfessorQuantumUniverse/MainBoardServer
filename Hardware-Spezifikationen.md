# Hardware-Spezifikationen und technische Details

## Übersicht
Dieses Dokument enthält detaillierte technische Spezifikationen, Kompatibilitätsinformationen und Hardware-Empfehlungen für dein HomeLab-Setup.

## 1. Mainboard-Spezifikationen

### 1.1 Lenovo Laptop Mainboard

**Typische Spezifikationen (Beispiel: ThinkPad T-Serie):**

#### CPU:
- **Intel Core i5/i7** (6th-10th Generation)
- Kerne: 2-4 physische Cores (4-8 Threads mit Hyperthreading)
- Basis-Takt: 2.0-2.6 GHz
- Turbo: bis 4.5 GHz
- TDP: 15W-45W (U-Series: 15W, H-Series: 45W)
- **Wichtig:** CPU ist fest verlötet, nicht austauschbar

#### RAM:
- **DDR4 SO-DIMM** Slots (meist 2x)
- Maximale Kapazität: 32GB-64GB (abhängig vom Modell)
- Unterstützte Geschwindigkeit: 2400-3200 MHz
- Dual-Channel fähig
- **Empfehlung:** 2x 16GB (32GB total) für Docker-Workloads

#### Storage:
- **M.2 2280 Slot** (NVMe PCIe Gen3 x4)
  - Alternativ: M.2 2242 (kürzere Version)
  - Geschwindigkeit: bis 3500 MB/s read, 3000 MB/s write
- **SATA 2.5" Anschluss** (bei einigen Modellen zusätzlich)
  - Geschwindigkeit: bis 550 MB/s
- **Empfehlung:** 512GB NVMe SSD (Samsung 970 EVO Plus oder Crucial P3)

#### Netzwerk:
- **Intel Ethernet** (typisch I219-V oder I219-LM)
  - Geschwindigkeit: Gigabit (1000 Mbps)
  - Wake-on-LAN Support
  - VLAN-Tagging fähig
- **WiFi/Bluetooth** (optional, meist M.2 2230 Modul)
  - Intel AX200/AX201 (WiFi 6)
  - Kann für zusätzlichen Storage-Slot entfernt werden

#### USB-Ports:
- 2-4x USB 3.0/3.1 (Typ-A)
- 1-2x USB-C (mit DisplayPort Alt-Mode)
- Geeignet für externe Storage, USB-Ethernet-Adapter

#### Video-Ausgänge:
- HDMI 1.4/2.0 oder Mini DisplayPort
- USB-C DisplayPort
- Integrierte Intel UHD Graphics 620/630

#### Audio:
- Realtek HD Audio Codec
- 3.5mm Combo-Jack
- Nicht kritisch für Server-Betrieb

#### Power:
- **Input:** 19-20V DC, 3.25A-4.5A (65W-90W)
- **Original Lenovo Netzteil erforderlich**
- Barrel Jack Durchmesser: 4.5mm outer, 3.0mm inner
- **Wichtig:** Pinbelegung beachten (Center: +)

#### BIOS/UEFI:
- Lenovo proprietäres BIOS
- UEFI Boot-Support
- Secure Boot (kann deaktiviert werden)
- Boot-Optionen: USB, M.2, Network (PXE)
- **Whitelist für WiFi-Karten** (bei älteren Modellen)

#### Besonderheiten:
- **Embedded Controller (EC):** Verwaltet Power, Lüfter, Tastatur
- **TPM 2.0 Chip:** für Verschlüsselung
- **Kensington Lock:** physische Sicherheit
- **CMOS Batterie:** CR2032 für BIOS-Settings

#### Kühlung:
- Single Heatpipe System mit Lüfter
- Lüftersteuerung: BIOS-kontrolliert
- **Empfehlung:** Wärmeleitpaste erneuern (Arctic MX-4 oder Noctua NT-H1)
- Externe Kühlung ergänzen (siehe Rack-Aufbau)

### 1.2 Dell Laptop Mainboard

**Typische Spezifikationen (Beispiel: Latitude E-Serie oder XPS):**

#### CPU:
- **Intel Core i5/i7** (ähnlich Lenovo)
- Architektur: Skylake bis Comet Lake
- TDP: 15W-45W
- **Achtung:** Einige Dell-Modelle haben proprietäre CPU-Sockel

#### RAM:
- **DDR4 SO-DIMM** (2x Slots)
- Maximum: 32GB-64GB
- Geschwindigkeit: 2400-3200 MHz
- **Dell-spezifisch:** Manchmal empfindlich bei RAM-Kompatibilität
- **Empfehlung:** Crucial oder Kingston, getestet mit Dell

#### Storage:
- **M.2 2280 NVMe** (PCIe Gen3 x4)
- **M.2 2230 Slot** (manchmal für WiFi/Storage)
- **SATA 2.5"** (je nach Modell)
- **Dell-spezifisch:** Einige Modelle haben proprietäre Connector

#### Netzwerk:
- **Intel oder Qualcomm Atheros Ethernet**
  - Gigabit-fähig
- **Dell Wireless** (Intel oder Killer)
  - WiFi 5/6
  - Manchmal in Whitelist gesperrt

#### USB-Ports:
- 2-4x USB 3.0/3.1
- 1-2x USB-C (Thunderbolt 3/4 bei XPS)
- SD-Card Reader (bei einigen Modellen)

#### Video-Ausgänge:
- HDMI 1.4/2.0
- Mini DisplayPort oder USB-C DP
- Dedizierte NVIDIA GPU (bei einigen Modellen)
  - GeForce MX150/250 oder GTX 1050
  - **Vorteil:** GPU-Computing für ML-Workloads

#### Power:
- **Input:** 19.5V DC, 3.34A-4.62A (65W-90W)
- **Dell-spezifisch:** Oft proprietärer Barrel Jack (7.4mm outer, 5.0mm inner mit Center-Pin)
- **Wichtig:** Dell-Netzteile haben ID-Chip (OEM erforderlich)

#### BIOS/UEFI:
- Dell BIOS mit umfangreichen Optionen
- Hardware-Monitoring
- RAID-Konfiguration (bei einigen Modellen)
- Intel ME (Management Engine)

#### Besonderheiten:
- **Dell ControlVault:** Fingerprint/SmartCard Security
- **TPM 2.0**
- **Express Charge:** Schnellladefunktion (für uns irrelevant)

#### Kühlung:
- Copper Heatpipe System
- Variabel-Speed Lüfter
- **Dell-spezifisch:** Lüftersteuerung oft aggressiv
  - Kann via i8kutils gesteuert werden (Linux)

### 1.3 Mainboard Identifikation und Dokumentation

#### Lenovo identifizieren:
```bash
# Mainboard-Info auslesen
sudo dmidecode -t baseboard

# Spezifische Modellnummer
sudo dmidecode -s baseboard-product-name

# FRU (Field Replaceable Unit) Nummer
# Steht auf Aufkleber auf dem Board
# Format: FRU P/N: 01AV123 (Beispiel)
```

#### Dell identifizieren:
```bash
# Service Tag auslesen
sudo dmidecode -s system-serial-number

# Dell Support-Website: support.dell.com
# Service Tag eingeben → Technische Daten abrufen
```

#### Dokumentieren:
```
Lenovo Board:
- Modell: ThinkPad T480
- FRU P/N: 01AV489
- CPU: Intel Core i5-8250U
- RAM Slots: 2x SO-DIMM (Max 32GB)
- M.2: 2280 NVMe PCIe x4
- Ethernet: Intel I219-V

Dell Board:
- Modell: Latitude E7470
- Service Tag: ABCD123
- CPU: Intel Core i5-6300U
- RAM Slots: 2x SO-DIMM (Max 32GB)
- M.2: 2280 SATA
- Ethernet: Intel I219-LM
```

## 2. Raspberry Pi Spezifikationen

### 2.1 Raspberry Pi 4 Model B

#### Technische Daten:

**Prozessor:**
- Broadcom BCM2711 (ARM Cortex-A72)
- Quad-Core @ 1.5 GHz
- ARMv8 64-bit
- **Performance:** ~4x schneller als Pi 3B+

**RAM:**
- Verfügbare Varianten: 2GB, 4GB, 8GB LPDDR4-3200
- **Empfehlung für HomeLab:** 4GB minimum, 8GB ideal
- Shared Memory mit GPU (512MB default)

**Storage:**
- microSD Slot (bis zu 512GB)
  - **Empfehlung:** Samsung EVO Plus 64GB+ (A2 Class)
- USB 3.0 Boot Support (SSD empfohlen!)
  - Deutlich schneller und zuverlässiger als SD

**Netzwerk:**
- **Gigabit Ethernet** (über USB 3.0 Bus)
  - Theoretisch: 1000 Mbps
  - Praktisch: ~950 Mbps (Gen2 USB-Limitierung)
- **WiFi:** 2.4GHz & 5GHz (802.11ac/WiFi 5)
  - Dual-Band
  - Bluetooth 5.0 / BLE
- **MAC-Adresse:** Unique, fest programmiert

**USB-Ports:**
- 2x USB 3.0 (5 Gbps)
- 2x USB 2.0 (480 Mbps)
- **Stromversorgung pro Port:** 1.2A max

**Video/Display:**
- 2x micro-HDMI (bis zu 4K@60Hz)
- Dual-Display-Support
- H.265 (4K), H.264 (1080p) Hardware-Decoding
- OpenGL ES 3.0, Vulkan Support

**GPIO:**
- 40-Pin Header
- SPI, I2C, UART
- PWM-Kanäle
- 3.3V Logic Level
- **Geeignet für:** Sensor-Integration, Lüftersteuerung

**Power:**
- **Eingang:** 5V via USB-C
- **Mindest-Strom:** 3A (15W)
- **Empfohlenes Netzteil:** Official Raspberry Pi PSU 5.1V/3A
- **Unter Last:** bis zu 12.5W (mit Peripherie)
- **PoE Support:** über PoE HAT (optional)

**Abmessungen:**
- 88 x 58 x 19.5 mm
- Gewicht: 46g
- Standard-Montagelöcher (M2.5, 58×49mm)

**Betriebstemperatur:**
- Spezifiziert: 0°C bis 50°C
- Throttling ab: ~80°C
- **Empfehlung:** Aktive Kühlung ab 60°C
- Passiver Kühlkörper: Aluminium oder Kupfer

**Besonderheiten:**
- Real-Time Clock Support (mit RTC HAT)
- EEPROM für Boot-Konfiguration
- USB-Boot möglich (kein SD-Karte nötig)
- Wake-on-LAN fähig (mit Konfiguration)

### 2.2 Alternative: Raspberry Pi 5 (wenn verfügbar)

**Verbesserungen über Pi 4:**
- CPU: Broadcom BCM2712 (Cortex-A76 @ 2.4GHz)
- Performance: ~2-3x schneller
- PCIe 2.0 x1 Interface (M.2 HAT Support!)
- Bessere I/O-Performance
- Dedizierter Ethernet-Controller
- Power-Button
- RTC onboard

**Nachteil:**
- Höherer Stromverbrauch (bis 5A unter Last)
- Teurer
- Weniger verfügbar

## 3. Netzwerk-Hardware

### 3.1 TP-Link 5-Port Switch Modelle

#### TL-SG105 (Unmanaged, Metal)
**Spezifikationen:**
- Ports: 5x Gigabit (10/100/1000 Mbps)
- Switching Capacity: 10 Gbps
- MAC Address Table: 2K
- Jumbo Frames: 10KB
- Auto MDI/MDIX
- IEEE 802.3x Flow Control
- Green Ethernet (Energiesparend)

**Abmessungen:** 99 × 98 × 25 mm
**Power:** 100-240V AC, 50/60Hz, max 4.2W
**Montage:** Desktop (Metallgehäuse)
**Kühlung:** Lüfterlos (passiv)

**Preis:** ~15-20 EUR

**Vorteile:**
- Klein und kompakt
- Lüfterlos (geräuschlos)
- Robust (Metallgehäuse)
- Plug-and-Play

**Nachteile:**
- Kein Management (kein VLAN, QoS)
- Keine Port-Statistiken
- Keine PoE

#### TL-SG105E (Easy Smart, Managed)
**Zusätzliche Features:**
- Web-basiertes Management
- VLAN Support (Port-based, 802.1Q)
- QoS (Quality of Service)
- IGMP Snooping
- Port Mirroring
- Loop Prevention
- Cable Diagnostics

**Preis:** ~25-30 EUR

**Empfehlung für HomeLab:** Besser als unmanaged für VLANs!

#### TL-SG1005D (Plastic, Budget)
- Ähnlich wie SG105
- Plastikgehäuse
- Günstiger (~12 EUR)
- Weniger robust

#### Alternativen:

**Netgear GS105 / GS305:**
- Vergleichbar mit TP-Link SG105
- Etwas teurer aber zuverlässig
- Lifetime Warranty (bei einigen Modellen)

**Ubiquiti UniFi Switch Flex Mini:**
- 5 Ports Gigabit
- Managed (UniFi Controller)
- PoE auf einem Port
- ~30 EUR
- **Vorteil:** Professionelles Management

### 3.2 Verkabelung

#### Ethernet-Kabel Spezifikationen:

**Cat5e:**
- Geschwindigkeit: bis 1 Gbps (Gigabit)
- Frequenz: 100 MHz
- Max. Länge: 100m
- Preis: ~0.50 EUR/m
- **Ausreichend für HomeLab**

**Cat6:**
- Geschwindigkeit: bis 10 Gbps (bei <55m)
- Frequenz: 250 MHz
- Bessere Abschirmung
- Preis: ~0.70 EUR/m
- **Empfohlen für Zukunftssicherheit**

**Cat6a:**
- Geschwindigkeit: 10 Gbps bei voller Länge (100m)
- Frequenz: 500 MHz
- Dickeres Kabel
- Preis: ~1.00 EUR/m
- **Overkill für aktuelles Setup**

**Cat7/7a:**
- Bis 40 Gbps (Cat7a)
- Immer geschirmt (S/FTP)
- Benötigt spezielle GG45/TERA Connectors für volle Performance
- Mit RJ45: gleich wie Cat6a
- **Nicht nötig**

**Empfehlung:**
- Kurze Patch-Kabel (0.3m-1m): Cat6, gekauft
- Längere Kabel (2m+): Cat6, selbst gecrimpt oder gekauft

**Kabel-Typen:**
- **UTP (Unshielded):** Standard, ausreichend
- **STP/FTP (Shielded):** Bei EMI-Problemen
  - Benötigt Erdung!
- **SSTP (Screened Shielded):** Maximale Abschirmung

**Stecker:**
- **RJ45 Standard:** für alle Ethernet
- **Belegung:** T568A oder T568B (konsistent bleiben!)
  - T568B ist gängiger

**Werkzeug für DIY:**
- Crimp-Tool: ~10-20 EUR
- Kabel-Tester: ~5-10 EUR
- RJ45-Stecker: ~0.10 EUR/Stück (100er Pack)
- Knickschutz: optional

## 4. Stromversorgung

### 4.1 Netzteile

#### Lenovo Netzteil:
**Original Lenovo Slim Tip (rechteckig):**
- Output: 20V, 3.25A (65W) oder 4.5A (90W)
- **Teil-Nummer:** ADLX65NLC3A oder ADLX90NLC3A
- Preis (Original): 40-60 EUR
- Preis (Kompatibel): 15-25 EUR
- **Wichtig:** Tip-Größe beachten!

**Alternative: USB-C PD (bei neueren Models):**
- USB Power Delivery
- 65W oder 100W
- **Vorteil:** Universal
- z.B. Anker PowerPort Atom PD 4

#### Dell Netzteil:
**Original Dell Barrel Jack:**
- Output: 19.5V, 3.34A (65W) oder 4.62A (90W)
- **Stecker:** 7.4mm/5.0mm mit Center-Pin
- **Teil-Nummer:** LA65NS2-01, HA65NS5-00
- Preis (Original): 30-50 EUR
- **Problem:** Center-Pin für Identifikation
  - Nachbau-Netzteile ohne ID-Chip funktionieren evtl. nicht
  - BIOS-Warnung möglich

**Dell USB-C PD (XPS 13/15):**
- 45W, 65W, 90W, 130W
- Standard USB-PD kompatibel

#### Raspberry Pi Netzteil:
**Official Raspberry Pi PSU:**
- Output: 5.1V, 3A (15W)
- USB-C Connector
- Preis: 10 EUR
- **Vorteil:** Getestet, zuverlässig

**Alternative:**
- Anker PowerPort
- Aukey USB-C Charger
- **Wichtig:** Mindestens 3A!
- Keine Quick-Charge-Profile (macht Probleme)

#### TP-Link Switch:
- Integriertes Netzteil (100-240V AC)
- Kaltgerätekabel (C13)
- Max. Verbrauch: 5W

### 4.2 Stromverbrauch und Kosten

**Geschätzter Verbrauch (24/7 Betrieb):**

```
Lenovo Mainboard (Docker):
- Idle: 15-25W
- Load: 45-65W
- Durchschnitt: ~30W

Dell Mainboard (Proxmox):
- Idle: 20-30W
- Load: 50-80W
- Durchschnitt: ~40W

Raspberry Pi:
- Idle: 3-4W
- Load: 6-8W
- Durchschnitt: ~5W

TP-Link Switch:
- Konstant: ~4W

5x Lüfter (12V, je 0.15A):
- Gesamt: ~9W (bei 50% Speed: ~5W)

Gesamt-Durchschnitt: ~84W (100W mit Overhead)
```

**Stromkosten-Rechnung:**
```
Verbrauch: 100W = 0.1 kW
24/7 Betrieb: 0.1 kW × 24h × 365 Tage = 876 kWh/Jahr

Strompreis (Deutschland): ~0.40 EUR/kWh (2024)
Jährliche Kosten: 876 × 0.40 = ~350 EUR/Jahr
Monatlich: ~29 EUR/Monat

Bei optimierter Last (75W):
24/7 Betrieb: 657 kWh/Jahr
Jährliche Kosten: ~263 EUR/Jahr (~22 EUR/Monat)
```

**Optimierungen:**
- Idle-Zeit reduzieren (Sleep-Modus wenn möglich)
- Nicht benötigte Services abschalten
- Lüfter bei niedriger Temp. langsam laufen lassen
- CPU-Frequency-Scaling aktivieren

### 4.3 USV (Unterbrechungsfreie Stromversorgung)

**Empfohlene Modelle:**

#### APC Back-UPS BX700U-GR:
- Kapazität: 700VA / 390W
- Laufzeit: ~10-15 Min bei 100W Last
- Batterie: Sealed Lead-Acid (12V)
- Anschlüsse: 4x Schuko
- USB-Monitoring
- Preis: ~80-100 EUR
- **Ausreichend für HomeLab**

#### CyberPower CP1500EPFCLCD:
- Kapazität: 1500VA / 900W
- Pure Sine Wave
- LCD-Display
- Laufzeit: ~25-30 Min bei 100W
- Preis: ~200 EUR
- **Wenn mehr Laufzeit benötigt**

**Berechnungen:**
```
HomeLab: 100W
UPS: 700VA / 390W

Effektive Nutzung: ~85% (Umwandlungsverluste)
Nutzbare Kapazität: 390W × 0.85 = 330W

Bei 100W Last:
Laufzeit = (Batterie-Kapazität / Last) × Effizienz
Laufzeit ≈ 12-15 Minuten

Genug Zeit für:
- Notifications versenden
- VMs herunterfahren
- Container stoppen
- Clean Shutdown
```

**NUT Integration (Network UPS Tools):**
```bash
# Auf Raspberry Pi
sudo apt install nut

# UPS via USB verbinden
# Konfiguration erstellen
sudo nano /etc/nut/ups.conf

[apc]
  driver = usbhid-ups
  port = auto
  desc = "HomeLab UPS"

# Monitoring und Auto-Shutdown konfigurieren
```

## 5. Kühlung und Thermal Management

### 5.1 Lüfter-Spezifikationen

#### Standard 120mm PC-Lüfter:

**Noctua NF-P12 redux-1700 PWM:**
- Größe: 120x120x25mm
- Drehzahl: 450-1700 RPM (PWM)
- Airflow: 70.75 m³/h (bei 1700 RPM)
- Lautstärke: 22.6 dB(A) (bei 1700 RPM)
- Lebensdauer: 150,000 Stunden
- Preis: ~15 EUR
- **Premium-Wahl**

**Arctic P12 PWM PST:**
- Größe: 120x120x25mm
- Drehzahl: 200-1800 RPM
- Airflow: 56.3 CFM
- Lautstärke: 0.3 Sone (leise)
- Preis: ~6 EUR (5er-Pack: ~25 EUR)
- **Budget-Empfehlung**

**Be Quiet! Pure Wings 2:**
- Größe: 120x120x25mm
- Drehzahl: 1500 RPM
- Airflow: 51.4 CFM
- Lautstärke: 19.2 dB(A)
- Preis: ~10 EUR
- **Sehr leise**

#### Alternative: 80mm oder 92mm Lüfter
- Für engere Räume
- Höhere Drehzahl für gleichen Airflow
- Lauter als 120mm

#### Raspberry Pi Lüfter (40mm):
**Noctua NF-A4x10 FLX:**
- Größe: 40x40x10mm
- Drehzahl: 4500 RPM
- Airflow: 4.8 m³/h
- Lautstärke: 12.9 dB(A)
- Anschluss: 5V über GPIO
- Preis: ~14 EUR
- **Beste Wahl für Pi**

**GeeekPi Ice Tower:**
- RGB-Kühler mit Lüfter
- Deutliche Kühlung (-20°C)
- Preis: ~15 EUR

### 5.2 Temperatursensoren

**DS18B20 Digital Temperature Sensor:**
- Interface: 1-Wire (über GPIO)
- Bereich: -55°C bis +125°C
- Genauigkeit: ±0.5°C
- Preis: ~2-3 EUR pro Sensor
- **Wasserdicht-Version verfügbar**

**DHT22 (Alternative):**
- Temperatur + Luftfeuchtigkeit
- Bereich: -40°C bis +80°C
- Genauigkeit: ±0.5°C
- Preis: ~5 EUR

**Software-Monitoring:**
```bash
# lm-sensors für CPU-Temp
sudo apt install lm-sensors
sudo sensors-detect
sensors

# Raspberry Pi
vcgencmd measure_temp

# Integration in Prometheus/Grafana
```

### 5.3 Wärmeleitpaste

**Arctic MX-4:**
- Wärmeleitfähigkeit: 8.5 W/(m·K)
- Anwendung: Sehr einfach
- Haltbarkeit: 8+ Jahre
- Preis: 4g für ~8 EUR
- **Standard-Empfehlung**

**Noctua NT-H1:**
- Wärmeleitfähigkeit: >3.5 W/(m·K)
- Premium-Qualität
- Preis: 3.5g für ~10 EUR

**Thermal Grizzly Kryonaut:**
- Wärmeleitfähigkeit: 12.5 W/(m·K)
- Beste Performance
- Preis: 1g für ~8 EUR
- **Für Enthusiasten**

## 6. Kompatibilität und Troubleshooting

### 6.1 RAM-Kompatibilität

**Empfohlene Hersteller:**
- **Crucial:** Beste Kompatibilität (Micron-Chips)
- **Kingston:** Zuverlässig
- **Samsung:** Premium
- **Corsair:** Meist kompatibel
- **G.Skill:** High-Performance

**Zu vermeiden:**
- No-Name China-RAM
- Niedrigste Qualität (CL22+)

**Testen:**
```bash
# memtest86+ beim Boot
# oder
sudo apt install memtester
sudo memtester 1000M 5

# Für 1GB Test, 5 Durchläufe
```

### 6.2 NVMe SSD Kompatibilität

**Empfohlene SSDs:**

**Samsung 970 EVO Plus (500GB/1TB):**
- Interface: PCIe 3.0 x4
- Read: bis 3500 MB/s
- Write: bis 3300 MB/s
- Preis: 500GB ~50 EUR, 1TB ~80 EUR
- **Sehr zuverlässig**

**Crucial P3 (500GB/1TB):**
- Interface: PCIe 3.0 x4
- Read: bis 3500 MB/s
- Write: bis 3000 MB/s
- Preis: 500GB ~35 EUR, 1TB ~60 EUR
- **Budget-Empfehlung**

**WD Blue SN570:**
- Interface: PCIe 3.0 x4
- Read: bis 3500 MB/s
- Write: bis 3000 MB/s
- Preis: ähnlich Crucial

**Zu vermeiden:**
- QLC-NAND bei intensivem Schreiben
  - Kurze Lebensdauer
- DRAMless-SSDs für OS-Drive
  - Schlechtere Latenz

### 6.3 Bootloader und BIOS-Probleme

#### Lenovo BIOS-Zugang:
- **Boot-Taste:** F1 (BIOS), F12 (Boot-Menu)
- **Passwort vergessen:** CMOS-Reset (Batterie entfernen)
- **Whitelist-Problem:**
  - Ältere ThinkPads haben WiFi-Whitelist
  - Lösung: Modifiziertes BIOS (Vorsicht!) oder akzeptierte Karten

#### Dell BIOS-Zugang:
- **Boot-Taste:** F2 (BIOS), F12 (Boot-Menu)
- **Diagnostics:** F12 → Diagnostics
- **BIOS-Update:** Von USB möglich

#### USB-Boot-Probleme:
```bash
# Secure Boot deaktivieren
# Legacy/UEFI Boot Mode prüfen
# Fast Boot deaktivieren

# USB-Stick vorbereiten (Linux):
sudo dd if=ubuntu-22.04-server.iso of=/dev/sdX bs=4M status=progress
sudo sync
```

### 6.4 Netzwerk-Troubleshooting

**Ethernet funktioniert nicht:**
```bash
# Interface prüfen
ip link show

# Treiber prüfen
lspci -k | grep -A 3 Ethernet

# MAC-Adresse
ip link show eth0 | grep link/ether

# Kabel-Test
ethtool eth0
```

**Switch-Probleme:**
- LED-Status prüfen:
  - Grün: Link OK
  - Orange: 100Mbps (sollte grün sein für Gigabit)
  - Blinken: Daten-Transfer
  - Aus: Kein Link

## 7. Einkaufsliste und Preise (Deutschland, 2024)

### Must-Have:

| Komponente | Produkt | Preis |
|-----------|---------|-------|
| **Regal** | IKEA Ivar 50x30 | 60 EUR |
| **Switch** | TP-Link TL-SG105E | 25 EUR |
| **Raspberry Pi** | RPi 4B 4GB | 60 EUR |
| **Pi Netzteil** | Official PSU 5.1V/3A | 10 EUR |
| **Pi Gehäuse** | Flirc Case (passiv) | 25 EUR |
| **RAM (Lenovo)** | Crucial 2x16GB DDR4 | 70 EUR |
| **RAM (Dell)** | Crucial 2x16GB DDR4 | 70 EUR |
| **SSD (Lenovo)** | Crucial P3 500GB | 35 EUR |
| **SSD (Dell)** | Crucial P3 500GB | 35 EUR |
| **SD-Karte** | Samsung EVO Plus 64GB | 12 EUR |
| **Lüfter** | Arctic P12 (5er Pack) | 25 EUR |
| **Ethernet-Kabel** | Cat6, 5x versch. Längen | 20 EUR |
| **Wärmeleitpaste** | Arctic MX-4 4g | 8 EUR |
| **Kabelmanagement** | Kabelbinder, Clips | 10 EUR |
| **3D-Druck Material** | PETG 1kg | 20 EUR |
| **Kleinteile** | Schrauben, Standoffs | 15 EUR |
| **Gesamt** | | **500 EUR** |

### Nice-to-Have:

| Komponente | Produkt | Preis |
|-----------|---------|-------|
| **USV** | APC BX700U | 90 EUR |
| **Temp-Sensoren** | DS18B20 (5 Stück) | 10 EUR |
| **Lüftersteuerung** | Arduino Nano | 15 EUR |
| **USB-SSD (Pi)** | Crucial X8 500GB | 60 EUR |
| **LED-Strips** | WS2812B 5m | 20 EUR |
| **zusätzliche NVMe** | Samsung 970 EVO 1TB | 80 EUR |
| **Coral TPU** | Google Coral USB | 60 EUR |
| **Managed Switch** | UniFi Flex Mini | 30 EUR |
| **Gesamt** | | **365 EUR** |

**Gesamt-Investment:**
- **Minimal:** 500 EUR
- **Mit Extras:** 865 EUR
- **+ Mainboards falls gekauft:** 100-200 EUR (gebraucht)

## 8. Wartungsplan

### Täglich (automatisiert):
- [ ] Temperatur-Check (Alerting)
- [ ] Verfügbarkeit-Check (Uptime)
- [ ] Backup-Job-Status

### Wöchentlich:
- [ ] Logs durchsehen
- [ ] Speicherplatz prüfen
- [ ] Security-Updates

### Monatlich:
- [ ] Staub entfernen (Druckluft)
- [ ] Kabel-Check
- [ ] Performance-Review

### Quartalsweise:
- [ ] Lüfter reinigen
- [ ] Schrauben nachziehen
- [ ] Wärmeleitpaste prüfen
- [ ] Backup-Tests

### Jährlich:
- [ ] Wärmeleitpaste erneuern
- [ ] BIOS-Updates
- [ ] Hardware-Upgrade-Planung
- [ ] Dokumentation aktualisieren

## Zusammenfassung

Dieses Dokument bietet eine umfassende technische Grundlage für dein HomeLab. Ergänze es mit spezifischen Informationen deiner tatsächlichen Hardware-Komponenten, sobald du sie identifiziert hast.

**Wichtigste Punkte:**
- Dokumentiere deine spezifischen Mainboard-Modelle
- Teste RAM- und SSD-Kompatibilität vor Kauf
- Plane ausreichend Kühlung ein
- Berücksichtige Stromkosten
- Investiere in gute Grundkomponenten

→ Zurück zu: `Rack-Aufbau.md`, `Installation.md`, `Weitere-Ideen.md`
