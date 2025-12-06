# HomeLab Rack-Aufbau Guide

## Übersicht
Dieser Guide beschreibt detailliert, wie du ein funktionales HomeLab-Rack aus IKEA Ivar Regalen, 3D-gedruckten Komponenten und zwei Laptop-Mainboards (Lenovo und Dell) sowie einem Raspberry Pi aufbauen kannst.

## Benötigte Materialien

### IKEA Ivar Komponenten
- **1x IKEA Ivar Regalböden-Set** (50x30 cm oder 50x50 cm je nach Platzbedarf)
- **4x Ivar Seitenteile** (50 cm Höhe minimum)
- **6-8x Regalböden** für flexible Höhenanpassung
- **Schrauben und Beschläge** (im Ivar-Set enthalten)

**Alternative:** Schwerlastregal aus dem Baumarkt
- Tragkraft: mindestens 75 kg pro Boden
- Abmessungen: ca. 90x40x180 cm (B×T×H)
- Vorteil: Höhere Stabilität, Metallkonstruktion

### Elektronik-Komponenten
- **2x Laptop-Mainboards:**
  - Lenovo Mainboard (mit Modellbezeichnung notieren)
  - Dell Mainboard (mit Modellbezeichnung notieren)
- **1x Raspberry Pi** (empfohlen: Pi 4 Model B mit 4-8GB RAM)
- **TP-Link 5-Port Ethernet Switch** (z.B. TL-SG105 oder TL-SG1005D)
- **Netzteile:**
  - 2x Laptop-Netzteile (passend zu den Mainboards)
  - 1x USB-C Netzteil für Raspberry Pi (mindestens 3A)
- **Externe Lüfter:** 3-5x 120mm PC-Lüfter mit 12V Versorgung
- **Kabel:**
  - 5x Ethernet-Kabel (Cat6 oder Cat6a, verschiedene Längen)
  - Stromkabel und Verlängerungen
  - HDMI/DisplayPort-Kabel für Setup
- **Speicher:**
  - 2x M.2 SSD oder 2.5" SATA SSDs für die Mainboards
  - 1x microSD-Karte (mindestens 32GB Class 10) für Raspberry Pi

### 3D-Druck Komponenten

#### Liste der zu druckenden Teile:

1. **Mainboard-Halterungen (2 Stück)**
   - Abmessungen: Angepasst an die spezifischen Mainboard-Größen
   - Material: PETG oder ABS (hitzebeständiger als PLA)
   - Druckzeit: ca. 6-8 Stunden pro Stück
   - Infill: 30-40%
   - Wandstärke: 3-4 Perimeter
   - **Features:**
     - Montage-Löcher kompatibel mit Laptop-Standoffs (M3)
     - Abstandshalter für Luftzirkulation (min. 15mm)
     - Kabelmanagement-Clips integriert

2. **Raspberry Pi Halterung (1 Stück)**
   - Standard Raspberry Pi 4 Mounting-Holes
   - Material: PLA ausreichend
   - Druckzeit: ca. 2-3 Stunden
   - **Features:**
     - Kühlrippen oder Lüfterhalterung für aktive Kühlung
     - GPIO-Zugang von oben
     - SD-Karten-Zugang von der Seite

3. **Switch-Halterung für TP-Link (1 Stück)**
   - Dimensionen: 100x100x30mm (anpassbar)
   - Material: PLA oder PETG
   - Druckzeit: ca. 3-4 Stunden
   - **Features:**
     - Wandmontage oder Regalbefestigung
     - Kabelführung für 5 Ethernet-Kabel
     - Kühlschlitze für Wärmeabfuhr

4. **Kabelmanagement-Clips (10-15 Stück)**
   - Verschiedene Größen für unterschiedliche Kabeldicken
   - Material: PLA
   - Druckzeit: ca. 15-30 Minuten pro Clip
   - **Design:** Clip-On Design für Regalböden

5. **Lüfterhalterungen (3-5 Stück)**
   - Standard 120mm Lüfter-Montage
   - Material: PETG
   - Druckzeit: ca. 2 Stunden pro Stück
   - **Features:**
     - Vibrationsdämpfende Gummipuffer-Aufnahmen
     - Verstellbarer Winkel (0-45°)

6. **Netzteil-Organizer (2-3 Stück)**
   - Größe anpassbar an die Netzteile
   - Material: PETG
   - Druckzeit: ca. 4-5 Stunden pro Stück
   - **Features:**
     - Sichere Befestigung mit Kabelbindern
     - Luftzirkulation durch Gitterstruktur

### Werkzeug und Zubehör
- Kreuzschlitz-Schraubendreher
- Inbusschlüssel-Set
- Bohrmaschine mit Holz/Metallbohrern
- Kabelbinder (verschiedene Größen)
- Doppelseitiges Klebeband oder Klettband
- Wärmeleitpaste
- Multimeter für elektrische Tests
- Antistatik-Armband

## Schritt-für-Schritt Aufbau

### Phase 1: Regal-Montage und Vorbereitung

#### Schritt 1: Ivar-Regal Zusammenbau
```
1. Seitenteile aufstellen und mit Regalböden verbinden
2. Erste Ebene: 10cm vom Boden (für Kabelmanagement)
3. Zweite Ebene: 35cm (für Switch und Raspberry Pi)
4. Dritte Ebene: 60cm (für erstes Mainboard - Lenovo)
5. Vierte Ebene: 85cm (für zweites Mainboard - Dell)
6. Fünfte Ebene: 110cm (optional für Monitoring-Display)
```

**Stabilisierung:**
- Regal an der Wand befestigen (Kippschutz!)
- Eventuell Querstreben zwischen den Seitenteilen anbringen
- Gummifüße unter das Regal für Vibrationsdämpfung

#### Schritt 2: 3D-Druck Vorbereitung
1. Alle STL-Dateien in PrusaSlicer/Cura importieren
2. Druckeinstellungen optimieren:
   - Layer-Höhe: 0.2mm (guter Kompromiss)
   - Infill: 30-40% (strukturelle Teile)
   - Supports: Nur wo nötig (z.B. bei Lüfterhalterungen)
3. Druckreihenfolge priorisieren:
   - Zuerst: Mainboard-Halterungen (längste Druckzeit)
   - Dann: Lüfter- und Switch-Halterungen
   - Zuletzt: Kabelmanagement-Clips

**Tipp:** Drucke 1-2 Ersatzteile für kritische Komponenten!

### Phase 2: Montage der Komponenten

#### Schritt 3: Mainboard-Halterungen montieren

**Lenovo Mainboard (Ebene 3):**
```
1. 3D-gedruckte Halterung mit Schrauben am Regalboden befestigen
2. Mainboard-Standoffs (M3 Abstandshalter) in die Halterung einschrauben
3. Mainboard vorsichtig auflegen - auf korrekte Ausrichtung achten!
4. Mit M3-Schrauben das Mainboard fixieren (nicht zu fest!)
5. Kühlkörper auf CPU/GPU mit frischer Wärmeleitpaste
```

**Dell Mainboard (Ebene 4):**
- Gleiche Vorgehensweise wie beim Lenovo Board
- Darauf achten, dass genug Abstand zur unteren Ebene bleibt (min. 25cm)

**Wichtig:** 
- Antistatik-Armband tragen!
- Mainboards nie auf leitenden Oberflächen ablegen
- Vor dem Einbau auf Beschädigungen prüfen

#### Schritt 4: Lüfter-Installation

**Strategische Lüfterplatzierung:**
```
Position 1: Seitlich unten - Frischluft-Einzug
Position 2: Unter Lenovo Board - direkter Luftstrom
Position 3: Unter Dell Board - direkter Luftstrom
Position 4: Seitlich oben - Warmlufft-Abzug
Position 5: Hinten mittig - zusätzlicher Abzug (optional)
```

**Montage:**
1. Lüfterhalterungen mit 3D-gedruckten Brackets am Regal befestigen
2. Lüfter mit Gummidämpfern montieren (Vibration reduzieren)
3. Alle Lüfter an ein 12V-Netzteil anschließen
4. Lüftersteuerung optional: PWM-Controller oder per Software

**Luftstrom-Konzept:**
- Kaltluft von unten/vorne ansaugen
- Warmluft nach oben/hinten abführen
- Querstrom über die Mainboards vermeiden

#### Schritt 5: Netzwerk-Switch montieren

**TP-Link Switch (Ebene 2):**
```
1. 3D-gedruckte Switch-Halterung am Regalboden verschrauben
2. Switch in die Halterung einsetzen
3. Mit Kabelbindern oder Klettband zusätzlich sichern
4. Stromkabel anschließen und führen
```

**Verkabelung planen:**
```
Port 1: Lenovo Mainboard
Port 2: Dell Mainboard  
Port 3: Raspberry Pi
Port 4: Uplink zum Router (Internet)
Port 5: Reserve / Management-Laptop
```

#### Schritt 6: Raspberry Pi montieren

**Installation (Ebene 2, neben Switch):**
```
1. Raspberry Pi Halterung am Regalboden befestigen
2. Optional: Kleinen 40mm Lüfter für aktive Kühlung montieren
3. Pi in die Halterung einsetzen und fixieren
4. microSD-Karte einsetzen (vorher System flashen!)
5. USB-C Stromversorgung anschließen
```

**Funktion des Raspberry Pi:**
- Management und Monitoring (Prometheus, Grafana)
- DNS-Server (Pi-hole)
- VPN-Server (WireGuard)
- DHCP-Server für das Lab-Netzwerk

### Phase 3: Verkabelung und Kabelmanagement

#### Schritt 7: Stromversorgung

**Verkabelung:**
```
Hauptverteiler (Mehrfachsteckdose mit Überspannungsschutz):
├── Lenovo Netzteil
├── Dell Netzteil
├── Raspberry Pi Netzteil (USB-C)
├── TP-Link Switch
├── Lüfter-Netzteil (12V)
└── Reserve-Anschluss
```

**Sicherheit:**
- Mehrfachsteckdose mit Überspannungsschutz verwenden
- Netzteile in 3D-gedruckten Organizern befestigen
- Kabel sauber verlegen und bündeln
- Nichts unter Spannung arbeiten lassen

#### Schritt 8: Netzwerk-Verkabelung

**Ethernet-Kabel:**
```
1. Vom Router zum Switch (Port 4) - 2-3m Kabel
2. Switch (Port 1) zu Lenovo Board - 0.5m Kabel
3. Switch (Port 2) zu Dell Board - 0.75m Kabel
4. Switch (Port 3) zu Raspberry Pi - 0.3m Kabel
5. Alle Kabel mit Management-Clips fixieren
```

**Kabelführung:**
- Vertikal entlang der Regal-Seitenteile
- Horizontal unter den Regalböden
- Kabelmanagement-Clips alle 20-30cm
- Netzwerk- und Stromkabel getrennt führen (EMV)

#### Schritt 9: Kabelmanagement optimieren

**Best Practices:**
```
1. Farbcodierung verwenden:
   - Rot: Stromkabel
   - Blau: Netzwerkkabel
   - Schwarz: USB/Peripherie
   
2. Beschriftung:
   - Alle Kabel an beiden Enden labeln
   - Druckbare Kabel-Labels oder Kabelbinder-Tags
   
3. Kabellängen:
   - Nicht zu lang (vermeidet Kabelsalat)
   - Nicht zu kurz (ermöglicht Wartung)
   - 20-30% Reserve einplanen
   
4. Zugänglichkeit:
   - Wichtige Komponenten leicht erreichbar
   - Service-Loops für Wartung
   - Notabschaltung gut sichtbar
```

### Phase 4: Cooling und Thermal Management

#### Schritt 10: Lüfter-Konfiguration

**PWM-Steuerung (optional aber empfohlen):**
```
Hardware:
- Arduino Nano oder ESP32
- 5x PWM-fähige Lüfter oder PWM-Adapter
- Temperatursensoren (DS18B20) an kritischen Punkten

Software-Steuerung:
- Temperaturschwellen definieren:
  * Unter 40°C: 30% Lüfterleistung (leise)
  * 40-55°C: 50% Lüfterleistung
  * 55-70°C: 75% Lüfterleistung
  * Über 70°C: 100% Lüfterleistung (max)
```

**Manuelle Steuerung:**
- 12V-Spannungsregler für alle Lüfter gemeinsam
- Einstellung auf ca. 7-9V für Balance zwischen Kühlung und Lautstärke

#### Schritt 11: Temperatur-Monitoring

**Sensor-Plazierung:**
```
Sensor 1: Nähe Lenovo CPU
Sensor 2: Nähe Dell CPU
Sensor 3: Unter dem Switch
Sensor 4: Nähe Raspberry Pi
Sensor 5: Allgemeine Umgebungstemperatur
```

**Monitoring-Software:**
- collectd oder telegraf auf jedem Host
- Daten an Raspberry Pi senden
- Grafana-Dashboard für Visualisierung

### Phase 5: Finishing Touches

#### Schritt 12: Ästhetik und Abdeckungen

**Optionale Verbesserungen:**
```
1. Acrylglas-Seitenwände:
   - Staubschutz
   - Transparente Optik (zeigt Hardware)
   - Demontierbar für Wartung
   
2. LED-Beleuchtung:
   - LED-Strips mit RGB (WS2812B)
   - Steuerbar über Raspberry Pi
   - Statusanzeige (grün=ok, gelb=warnung, rot=fehler)
   
3. Beschriftungen:
   - Labels für jede Komponente
   - Netzwerk-Diagramm sichtbar anbringen
   - QR-Codes zu Dokumentationen
```

#### Schritt 13: Sicherheits-Checks

**Pre-Power-On Checkliste:**
```
□ Alle Schrauben fest angezogen?
□ Keine losen Kabel oder Kurzschlussgefahr?
□ Lüfter drehen frei (keine Blockaden)?
□ Mainboards korrekt geerdet?
□ Stromversorgung: richtige Spannung/Polarität?
□ Netzwerk-Kabel richtig gesteckt?
□ Kühlkörper mit Wärmeleitpaste versehen?
□ Antistatik-Maßnahmen befolgt?
□ Überspannungsschutz aktiv?
□ Regal stabil und an Wand befestigt?
```

#### Schritt 14: Erster Power-On

**Schritt-für-Schritt Inbetriebnahme:**
```
1. Nur Raspberry Pi einschalten - Test
2. Switch einschalten - LED-Check
3. Lüfter einschalten - Drehzahl prüfen
4. Lenovo Mainboard einschalten:
   - Beobachte POST-LEDs
   - Höre auf Piep-Codes
   - Prüfe Display-Output
5. Dell Mainboard einschalten:
   - Gleiche Prozedur wie Lenovo
6. Alle Systeme parallel laufen lassen
7. Temperatur über 30 Minuten überwachen
```

**Troubleshooting beim ersten Start:**
- Kein POST-Signal: Überprüfe RAM-Installation
- Überhitzung: Lüfter-Richtung und Wärmeleitpaste prüfen
- Netzwerk-Probleme: Kabel und Switch-Ports testen
- Instabilität: Stromversorgung und Erdung prüfen

## Wartung und Pflege

### Monatlich:
- Staubfilter reinigen
- Temperaturen kontrollieren
- Kabelverbindungen prüfen

### Quartalsweise:
- Lüfter reinigen und schmieren
- Software-Updates durchführen
- Backup-Tests durchführen

### Jährlich:
- Wärmeleitpaste erneuern
- Alle Schrauben nachziehen
- 3D-gedruckte Teile auf Risse prüfen

## Erwartete Kosten

```
IKEA Ivar Regal:              50-80 EUR
TP-Link Switch:               20-30 EUR
Raspberry Pi 4 (4GB):         50-70 EUR
Netzteil Raspberry Pi:        10-15 EUR
Lüfter (5 Stück):            25-50 EUR
Kabel und Kleinteile:        30-50 EUR
3D-Druck Material:           20-40 EUR
Werkzeug (falls nötig):      30-100 EUR
----------------------------------
Gesamt:                      235-435 EUR
```

*Mainboards und SSDs nicht eingerechnet (bereits vorhanden)*

## Sicherheitshinweise

⚠️ **WICHTIG:**
- Niemals an stromführenden Teilen arbeiten
- Mainboards immer geerdet betreiben
- Überspannungsschutz ist Pflicht
- Ausreichende Belüftung sicherstellen
- Brandschutz: Rauchmelder im Raum installieren
- Keine brennbaren Materialien in der Nähe
- Regelmäßige Inspektionen durchführen

## Nächste Schritte

Nach erfolgreichem Aufbau:
1. → Siehe `Installation.md` für Software-Setup
2. → Siehe `Weitere-Ideen.md` für Erweiterungen
3. → Siehe `Hardware-Spezifikationen.md` für technische Details
