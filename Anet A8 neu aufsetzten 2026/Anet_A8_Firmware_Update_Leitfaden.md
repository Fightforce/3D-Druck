# Anet A8 Firmware-Update auf Marlin 1.1.9 — Kompletter Leitfaden

**Status:** Für Eddy (Berlin) — Hobby-Nutzung (1-2x pro Monat)  
**Hardware:** Anet A8 von Nov 2016, Ubuntu PC mit GTX 1080  
**Ziel:** Sichere Marlin-Firmware mit Thermal-Runaway-Schutz + Sicherheits-Upgrades

---

## **Wichtiger Kontext**

⚠️ **Sicherheit zuerst!**
- Das original Anet A8 von 2016 hat **keine Thermal-Runaway-Protection** → Brandrisiko!
- Marlin 1.1.9 ist die **neueste stabile Version für 8-Bit-Boards** wie deinen A8
- Marlin 2.x ist **nicht kompatibel** mit dem 8-Bit Atmel-Prozessor
- Der USB-Programmer (USBasp) ist notwendig, um sicher zu flashen

---

## **Vorbereitungen (jetzt, während du auf USBasp wartest)**

### **1. Arduino IDE installieren**

Auf deinem Ubuntu-PC:

```bash
# Arduino IDE via Snap installieren (einfach und aktuell)
sudo snap install arduino

# ODER manuell von arduino.cc runterladen (falls Snap Probleme macht)
# https://www.arduino.cc/en/software
```

Verifiziere:
```bash
arduino --version
# Sollte "2.x.x" anzeigen (aktuell)
```

### **2. Anet A8 Board-Definition installieren**

Das ist notwendig, damit Arduino IDE deinen A8 erkennt.

```bash
# Erstelle das Anet-Hardware-Verzeichnis
mkdir -p ~/Arduino/hardware/anet/avr

# Klone das Anet-Repository (enthält Board-Definitionen)
cd ~/Arduino/hardware
git clone https://github.com/SkynetThings/Marlin-Anet-A8.git anet_temp

# Kopiere die Board-Definitionen an den richtigen Ort
cp -r ~/Arduino/hardware/anet_temp/anet/avr/* ~/Arduino/hardware/anet/avr/

# Cleanup
rm -rf ~/Arduino/hardware/anet_temp
```

**Überprüfe, ob es funktioniert:**
```bash
ls -la ~/Arduino/hardware/anet/avr/
# Du solltest sehen: boards.txt, platform.txt, etc.
```

### **3. Marlin 1.1.9 Firmware-Quelle herunterladen**

```bash
cd ~/3D-Drucker  # Oder dein bevorzugtes Verzeichnis
mkdir -p Anet_A8_Firmware && cd Anet_A8_Firmware

# Marlin 1.1.9 von GitHub laden (neueste für 8-Bit)
wget https://github.com/MarlinFirmware/Marlin/archive/refs/tags/1.1.9.zip
unzip 1.1.9.zip
cd Marlin-1.1.9

# Alternativ: Git klonen (falls wget nicht funktioniert)
# git clone --depth=1 --branch 1.1.9 https://github.com/MarlinFirmware/Marlin.git Marlin-1.1.9
```

### **4. Konfigurationsdateien für Anet A8 besorgen**

Die Konfigurationsdateien sollten aus dem Marlin-Repository kopiert und mit deinen Druckereinstellungen angepasst werden.

```bash
cd ~/3D-Drucker/Anet_A8_Firmware/Marlin-1.1.9

# Marlin Configurations-Repository klonen
git clone --depth=1 https://github.com/MarlinFirmware/Configurations.git

# Anet A8 Beispiel-Konfiguration kopieren
cp Configurations/Marlin/Configuration.h Marlin/
cp Configurations/Marlin/Configuration_adv.h Marlin/

# (Diese werden später mit deinen spezifischen Einstellungen editiert)
```

---

## **Phase 1: Backup der aktuellen Firmware (WICHTIG!)**

**Das wird gemacht, sobald du den USBasp hast und mit ihm verbunden bist.**

```bash
# Stelle sicher, dass der Drucker EINGESCHALTET ist
# Verbinde USBasp mit deinem PC via USB

# Überprüfe, ob der USBasp erkannt wird
lsusb | grep "USBasp"
# Sollte etwas wie "Future Technology Devices..." zeigen

# Backup erstellen mit avrdude
sudo apt install avrdude

avrdude -c usbasp -p m1284p -U flash:r:anet_a8_original_firmware.bin:r

# Das sollte eine Datei erstellen: anet_a8_original_firmware.bin
# Diese Datei SICHERN und ARCHIVIEREN (ist dein "Notfall-Fallback")
```

---

## **Phase 2: Arduino Bootloader prüfen/installieren**

Manche Anet A8 haben keinen korrekten Arduino-Bootloader — das wird während des Flashens automatisch erkannt.

**Mit dem USBasp verbunden:**

```bash
# Starte Arduino IDE
arduino &

# Gehe zu: Tools > Board > Anet A8 V1.0
# Gehe zu: Tools > Programmer > USBasp
# Gehe zu: Tools > Burn Bootloader

# Die IDE wird den Bootloader über USBasp flashen
# Warte, bis "Done uploading" erscheint
```

---

## **Phase 3: Marlin Firmware kompilieren**

**Starte Arduino IDE:**

```bash
arduino ~/3D-Drucker/Anet_A8_Firmware/Marlin-1.1.9/Marlin/Marlin.ino &
```

### **Schritt 1: Board-Definitionen setzen**

In der Arduino IDE:
- **Tools > Board > Anet A8 V1.0**
- **Tools > Processor > ATmega1284 or ATmega1284p (16 MHz)**
- **Tools > Programmer > USBasp**

### **Schritt 2: Konfiguration anpassen**

Öffne `Marlin/Configuration.h` in Arduino IDE:

**Wichtige Einstellungen für deinen Anet A8:**

```cpp
// SICHERHEIT - UNBEDINGT AKTIVIEREN
#define THERMAL_PROTECTION_HOTENDS   // Hotend Thermal Runaway Protection
#define THERMAL_PROTECTION_BED       // Bed Thermal Runaway Protection

// Drucker-Typ
#define SERIAL_PORT 0                // USB-Kommunikation
#define BAUDRATE 115200              // Standard-Baudrate

// Z-Axis (wichtig für Höhenanpassung)
#define Z_PROBE_OFFSET_FROM_EXTRUDER 0  // Wird später kalibriert

// Bewegungsgrenzen
#define X_BED_SIZE 220               // Anet A8 standard
#define Y_BED_SIZE 220
#define Z_MAX_POS 240

// PID-Heizer (Stock-Werte sind usually OK)
// Aber nach Firmware-Flash sollten diese neu kalibriert werden

// Display-Typ (für das LCD)
#define ULTIPANEL
#define NEWPANEL
```

**Öffne auch `Configuration_adv.h`:**

```cpp
// Weitere Sicherheit
#define WATCH_TEMP_PERIOD 40         // Überwache Temperatur-Stabilität
```

### **Schritt 3: Firmware kompilieren**

In Arduino IDE:
- **Sketch > Verify/Compile** (Ctrl+R)
- Warte, bis die Kompilation abgeschlossen ist
- Status sollte sein: **"Compiling sketch..." → Done**

Falls **Fehler** erscheinen: Überprüfe, dass alle Dateien korrekt im `Marlin/`-Verzeichnis sind.

---

## **Phase 4: Firmware flashen mit USBasp**

⚠️ **KRITISCH: Während des Flashens darf nichts unterbrochen werden!**

1. **Drucker ausschalten**, aber stromversorgt lassen
2. **USBasp mit Drucker verbinden** (6-pin ISP-Anschluss auf Mainboard)
3. **USBasp mit PC verbinden** (USB)
4. **Arduino IDE öffnen** (falls nicht offen)

In Arduino IDE:
- **Sketch > Upload Using Programmer** (Shift+Ctrl+U)
- Warte **2-3 Minuten** — Status sollte sein: "Uploading..." → Done

**Falls Fehler:**
```
avrdude: stk500_recv(): programmer is not responding
```
→ USBasp Verbindung überprüfen (korrekte 6-pin Anschlüsse?)

---

## **Phase 5: Nach dem Flash — Kalibrierung & EEPROM Reset**

Sobald Marlin lädt:

### **1. EEPROM zurücksetzen**

Das ist **notwendig**, wenn du von Stock-Firmware wechselst!

Auf dem Drucker-Display:
- **Configuration > Reset EEPROM**
- Oder per Terminal: `M502` dann `M500`

### **2. Bed Level neu kalibrieren**

Mit heißem Hotend (200°C) und heißem Bed (60°C):
- Nutze die **Paper-Method**: Papier unter die Düse schieben, bis leichter Widerstand
- Alle **4 Ecken** und **Mitte** prüfen
- Mehrmals wiederholen (einstellen einer Ecke beeinflusst andere)

### **3. Z-Offset bei Bedarf kalibrieren**

Falls der Drucker zu nah am Bed startet oder zu weit entfernt:
- Drucker ausschalten
- LCD: **Configuration > Babystep Z** (während das Bed geheizt ist)

---

## **Sicherheits-Hardware-Upgrades (für später)**

Nachdem die Firmware läuft, empfehle ich diese **kostengünstigen Verbesserungen:**

### **Sofort (5-10€)**
- ✅ **Thermische Feuchtigkeitsmessgeräte** für Hotend & Bed prüfen
- ✅ **Kabelanschlüsse** überprüfen (ältere können korrodieren)
- ✅ **Feuer-Warnmelder** über dem Drucker installieren (Sicherheit!)

### **Bald nach Hobby-Bedarf (20-50€)**
- 🔧 **Glas-Druckplatte** (bessere Ebenheit statt Acryl)
- 🔧 **Thermal Runaway Sensor** (redundante Sicherheit)
- 🔧 **Bessere Hotend-Heizer-Patrone** (heute Standardqualität besser als 2016)

### **Später, wenn Modding Spaß macht (100€+)**
- 🎯 **Auto-Leveling-Sensor** (BLTouch oder ähnlich)
- 🎯 **Besseres Hotend** (E3D V6, etc.)
- 🎯 **Mainboard-Upgrade** auf 32-bit (wenn gewünscht)

---

## **Troubleshooting**

### **Drucker startet nicht nach Flash**

```bash
# EEPROM komplett löschen und reset
# Verbinde USBasp und führe aus:
avrdude -c usbasp -p m1284p -e

# Dann kann man das Backup zurück flashen:
avrdude -c usbasp -p m1284p -U flash:w:anet_a8_original_firmware.bin:r
```

### **Drucker zeigt "Thermal Runaway Error"**

Das ist **GEWÜNSCHT** — es bedeutet, Marlin schützt dich! Das Bed oder der Hotend muss:
1. Richtig mit dem Mainboard verbunden sein
2. Thermistoren korrekt angeschlossen sein
3. Die Kabel sollten nicht beschädigt sein

### **Druck-Qualität ist anders nach Firmware-Update**

Das ist normal! Marlin hat andere Z-Offset-Verhaltensweisen:
- Recalibriere Bed-Level
- Überprüfe PID-Heizer-Werte (normalerweise OK)
- Erste 2-3 Drucke sind Kalibrierungsdrucke

---

## **Checkliste für Firmware-Update**

- [ ] Arduino IDE installiert & getestet
- [ ] Anet A8 Board-Definitionen installiert
- [ ] Marlin 1.1.9 heruntergeladen
- [ ] Konfigurationsdateien kopiert & angepasst
- [ ] Marlin erfolgreich kompiliert
- [ ] **USBasp bestellt & angekommen** ← DU BIST HIER
- [ ] Firmware-Backup erstellt
- [ ] Bootloader installiert
- [ ] Marlin geflasht
- [ ] EEPROM reset
- [ ] Bed Level neu kalibriert
- [ ] Testdruck erfolgreich

---

## **Wichtige Links & Resourcen**

- **Marlin Official:** https://marlinfw.org/
- **Arduino IDE Download:** https://www.arduino.cc/en/software
- **Anet A8 Configurations:** https://github.com/MarlinFirmware/Configurations/tree/master/config/examples/Anet/A8
- **AVRdude Dokumentation:** https://www.nongnu.org/avrdude/user-manual/avrdude.html

---

## **Kontakt & Fragen**

Falls während des Updates Probleme entstehen:
1. Mach einen **Screenshot des Fehlers**
2. Schreib den **exakten Fehlermeldung-Text**
3. Sag mir, bei welchem **Schritt** du stuck bist
4. Ich helfe dir weiter! 🔧

**Viel Erfolg beim Update!** 🚀
