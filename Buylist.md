# Detaillierte Einkaufsliste für HomeLab Server Rack

Diese Liste basiert auf den Hardware-Spezifikationen und enthält alle notwendigen Komponenten für den Aufbau des Mainboard-Server-Racks. Preise sind Schätzwerte (Stand: 2024, Deutschland).

## 🏗️ Grundgerüst & Montage

| Komponente | Modell / Spezifikation | Anzahl | Einzelpreis | Gesamt |
|------------|------------------------|:------:|------------:|-------:|
| **Rack-Regal** | IKEA Ivar (50x30 cm) oder vergleichbares Holzregal | 1 | 60,00 € | 60,00 € |
| **3D-Druck Filament** | PETG (für Hitzebeständigkeit), 1kg Rolle | 1 | 20,00 € | 20,00 € |
| **Montagematerial** | Schrauben, Muttern, Standoffs (M2.5, M3) | 1 Set | 15,00 € | 15,00 € |
| **Kabelmanagement** | Kabelbinder (Klett), Clips, Kabelschläuche | 1 Set | 10,00 € | 10,00 € |
| **Summe** | | | | **105,00 €** |

## 🖥️ Server Hardware & Compute

| Komponente | Modell / Spezifikation | Anzahl | Einzelpreis | Gesamt |
|------------|------------------------|:------:|------------:|-------:|
| **Raspberry Pi** | RPi 4 Model B (4GB oder 8GB) | 1 | 60,00 € | 60,00 € |
| **Pi Gehäuse** | Flirc Case (passiv) oder offenes Acryl-Case | 1 | 25,00 € | 25,00 € |
| **Mainboards** | *Vorhandene Laptop-Mainboards (Lenovo/Dell)* | 2 | - | - |
| **RAM (Lenovo)** | Crucial/Kingston 32GB Kit (2x16GB) DDR4 SO-DIMM | 1 | 70,00 € | 70,00 € |
| **RAM (Dell)** | Crucial/Kingston 32GB Kit (2x16GB) DDR4 SO-DIMM | 1 | 70,00 € | 70,00 € |
| **Summe** | | | | **225,00 €** |

## 💾 Speicher (Storage)

| Komponente | Modell / Spezifikation | Anzahl | Einzelpreis | Gesamt |
|------------|------------------------|:------:|------------:|-------:|
| **NVMe SSD (Lenovo)** | Crucial P3 oder Samsung 970 EVO Plus (500GB) | 1 | 35,00 € | 35,00 € |
| **NVMe SSD (Dell)** | Crucial P3 oder Samsung 970 EVO Plus (500GB) | 1 | 35,00 € | 35,00 € |
| **SD-Karte (Pi)** | Samsung EVO Plus 64GB (Class 10 / U3) | 1 | 12,00 € | 12,00 € |
| **USB-Stick (Boot)** | SanDisk Ultra Fit 32GB (für ISO Installationen) | 1 | 8,00 € | 8,00 € |
| **Summe** | | | | **90,00 €** |

## 🌐 Netzwerk

| Komponente | Modell / Spezifikation | Anzahl | Einzelpreis | Gesamt |
|------------|------------------------|:------:|------------:|-------:|
| **Switch** | TP-Link TL-SG105E (5-Port Gigabit, Managed) | 1 | 25,00 € | 25,00 € |
| **Patch-Kabel** | Cat6 Ethernet, kurz (0.25m - 0.5m) | 5 | 3,00 € | 15,00 € |
| **Uplink-Kabel** | Cat6 Ethernet, lang (zum Router) | 1 | 5,00 € | 5,00 € |
| **Summe** | | | | **45,00 €** |

## ❄️ Kühlung & Thermal

| Komponente | Modell / Spezifikation | Anzahl | Einzelpreis | Gesamt |
|------------|------------------------|:------:|------------:|-------:|
| **Lüfter (Main)** | Arctic P12 PWM PST (120mm), 5er Value Pack | 1 Pack | 25,00 € | 25,00 € |
| **Lüfter (Pi)** | Noctua NF-A4x10 FLX (40mm, 5V) | 1 | 14,00 € | 14,00 € |
| **Wärmeleitpaste** | Arctic MX-4 (4g Tube) | 1 | 8,00 € | 8,00 € |
| **Reinigung** | Isopropanol (Reinigungsalkohol) & Druckluft | 1 | 10,00 € | 10,00 € |
| **Summe** | | | | **57,00 €** |

## ⚡ Stromversorgung

| Komponente | Modell / Spezifikation | Anzahl | Einzelpreis | Gesamt |
|------------|------------------------|:------:|------------:|-------:|
| **Pi Netzteil** | Official Raspberry Pi USB-C Power Supply (5.1V/3A) | 1 | 10,00 € | 10,00 € |
| **Steckdosenleiste** | 6-fach mit Überspannungsschutz | 1 | 15,00 € | 15,00 € |
| **Netzteile (Laptops)** | *Vorhandene Original-Netzteile* | 2 | - | - |
| **Summe** | | | | **25,00 €** |

---

## 💰 Gesamtkalkulation

| Kategorie | Kosten |
|-----------|-------:|
| Grundgerüst | 105,00 € |
| Server Hardware | 225,00 € |
| Speicher | 90,00 € |
| Netzwerk | 45,00 € |
| Kühlung | 57,00 € |
| Stromversorgung | 25,00 € |
| **GESAMT (Must-Have)** | **~ 547,00 €** |

---

## 🌟 Nice-to-Have Upgrades (Optional)

Diese Komponenten sind nicht zwingend erforderlich, verbessern aber Sicherheit und Funktionalität.

| Komponente | Modell / Nutzen | Priorität | Preis |
|------------|-----------------|:---------:|------:|
| **USV** | APC Back-UPS BX700U-GR (Schutz vor Stromausfall) | Hoch | 90,00 € |
| **Temp-Sensoren** | DS18B20 (Überwachung der Rack-Temperatur) | Mittel | 10,00 € |
| **Lüftersteuerung** | Arduino Nano (PWM Steuerung für Lüfter) | Mittel | 15,00 € |
| **Externe SSD** | Crucial X8 1TB (für Backups am Pi) | Hoch | 80,00 € |
| **LED Beleuchtung** | WS2812B LED Strip (Statusanzeige/Optik) | Niedrig | 20,00 € |
| **Upgrade Gesamt** | | | **+ 215,00 €** |
