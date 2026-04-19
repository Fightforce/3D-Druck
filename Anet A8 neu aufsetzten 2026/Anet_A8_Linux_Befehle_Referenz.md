# Linux/Ubuntu Befehls-Referenz für Anet A8 Firmware-Update

## 🚀 Quick-Copy Befehle (für dein Ubuntu mit USBasp)

---

### **VORBEREITUNG (einmalig)**

```bash
# Arduino IDE installieren
sudo snap install arduino

# Anet A8 Board-Definitionen Setup
mkdir -p ~/Arduino/hardware/anet/avr
cd ~/Arduino/hardware
git clone https://github.com/SkynetThings/Marlin-Anet-A8.git anet_temp
cp -r ~/Arduino/hardware/anet_temp/anet/avr/* ~/Arduino/hardware/anet/avr/
rm -rf ~/Arduino/hardware/anet_temp

# Arbeitsverzeichnis erstellen
mkdir -p ~/3D-Drucker/Anet_A8_Firmware
cd ~/3D-Drucker/Anet_A8_Firmware

# Marlin 1.1.9 herunterladen
wget https://github.com/MarlinFirmware/Marlin/archive/refs/tags/1.1.9.zip
unzip 1.1.9.zip
cd Marlin-1.1.9

# Konfigurationen holen
git clone --depth=1 https://github.com/MarlinFirmware/Configurations.git
cp Configurations/Marlin/Configuration.h Marlin/
cp Configurations/Marlin/Configuration_adv.h Marlin/
```

---

### **WENN USBasp ANKOMMT - Backup erstellen**

```bash
# avrdude installieren
sudo apt install avrdude

# Überprüfe, dass USBasp erkannt wird
lsusb | grep -i usbasp
# Sollte "Future Technology Devices Int." anzeigen

# **DRUCKER MUSS EINGESCHALTET SEIN!**
# Verbinde USBasp mit Drucker-Mainboard (6-pin ISP)

# Firmware-Backup erstellen
avrdude -c usbasp -p m1284p -U flash:r:anet_a8_original_firmware.bin:r

# Überprüfe, dass die Datei da ist
ls -lh anet_a8_original_firmware.bin
# Sollte ca. 128-256 KB sein

# **WICHTIG: Diese Datei archivieren!**
# cp anet_a8_original_firmware.bin ~/Backups/
```

---

### **BOOTLOADER INSTALLIEREN (mit Arduino IDE GUI)**

```bash
# Starte Arduino IDE
arduino ~/3D-Drucker/Anet_A8_Firmware/Marlin-1.1.9/Marlin/Marlin.ino &

# Dann in der GUI:
# Tools > Board > "Anet A8 V1.0"
# Tools > Processor > "ATmega1284 or ATmega1284p (16 MHz)"
# Tools > Programmer > "USBasp"
# Tools > Burn Bootloader

# Warte bis "Done uploading" erscheint
```

---

### **MARLIN FIRMWARE KOMPILIEREN (mit Arduino IDE GUI)**

```bash
# Arduino IDE ist bereits offen mit Marlin.ino

# In Arduino IDE:
# Sketch > Verify/Compile (oder Ctrl+R)

# Warte auf "Done compiling" Meldung
# Falls Fehler: Überprüfe dass alle Dateien in ~/3D-Drucker/Anet_A8_Firmware/Marlin-1.1.9/Marlin/ sind
```

---

### **MARLIN FLASHEN (mit Arduino IDE GUI)**

```bash
# Arduino IDE mit Marlin ist offen
# USBasp ist verbunden (USB + 6-pin zum Drucker)
# **DRUCKER EINGESCHALTET, ABER NICHTS ZUM FLASHEN UNTERBRECHEN!**

# In Arduino IDE:
# Sketch > Upload Using Programmer (oder Shift+Ctrl+U)

# Warte 2-3 Minuten — Status sollte "Done uploading" zeigen
```

---

### **NACH DEM FLASH - EEPROM RESET**

```bash
# Option 1: Über Drucker-Display
# Drucker hochfahren → LCD-Menu → Configuration → Reset EEPROM

# Option 2: Über Terminal mit avrdude/USB
# (Später, wenn Drucker läuft und USB-Treiber installiert sind)

# Verbinde Drucker via USB an PC
# Der Port sollte sein: /dev/ttyUSB0 oder /dev/ttyACM0

# Überprüfe:
ls -la /dev/tty*

# Dann mit einem G-Code Sender (z.B. Pronterface)
# oder Terminal:
# echo "M502" > /dev/ttyUSB0  # Reset to defaults
# echo "M500" > /dev/ttyUSB0  # Save to EEPROM
```

---

### **TROUBLESHOOTING - Falls was schiefläuft**

```bash
# 1. Überprüfe ob USBasp verbunden ist
lsusb
# Sollte etwas mit "Future Technology" oder "USBasp" zeigen

# 2. Überprüfe, dass Drucker erkannt wird
avrdude -c usbasp -p m1284p

# 3. EEPROM komplett löschen (WARNUNG: macht Drucker "blank")
avrdude -c usbasp -p m1284p -e

# 4. Firmware-Backup zurück flashen (Notfall)
avrdude -c usbasp -p m1284p -U flash:w:anet_a8_original_firmware.bin:r
```

---

### **NACH ERFOLGREICHER INSTALLATION**

```bash
# Drucker neu starten
# Warte bis Marlin-Startup-Screen auf LCD angezeigt wird

# Überprüfe Firmware-Version auf LCD
# Sollte zeigen: "Marlin 1.1.9" oder ähnlich

# Bed-Level neu kalibrieren
# Drucker hochfahren → Heize Hotend & Bed auf Druck-Temp
# Dann manuell: Configuration > Babystep Z
# Oder klassisch: Paper-Method (Papier unter Düse)

# USB-Treiber für spätere Kommunikation (wenn nötig)
sudo apt install libusb-0.1-4:i386
```

---

## 📋 **Ablauf-Checkliste zum Copy-Pasten**

```bash
# === EINMALIGE VORBEREITUNG ===
sudo snap install arduino
mkdir -p ~/Arduino/hardware/anet/avr && cd ~/Arduino/hardware && \
git clone https://github.com/SkynetThings/Marlin-Anet-A8.git anet_temp && \
cp -r ~/Arduino/hardware/anet_temp/anet/avr/* ~/Arduino/hardware/anet/avr/ && \
rm -rf ~/Arduino/hardware/anet_temp
mkdir -p ~/3D-Drucker/Anet_A8_Firmware && cd ~/3D-Drucker/Anet_A8_Firmware
wget https://github.com/MarlinFirmware/Marlin/archive/refs/tags/1.1.9.zip && unzip 1.1.9.zip && cd Marlin-1.1.9
git clone --depth=1 https://github.com/MarlinFirmware/Configurations.git
cp Configurations/Marlin/Configuration.h Marlin/
cp Configurations/Marlin/Configuration_adv.h Marlin/

# === WENN USBasp ANKOMMT ===
sudo apt install avrdude
lsusb | grep -i usbasp
# Drucker EINGESCHALTET + USBasp verbunden!
avrdude -c usbasp -p m1284p -U flash:r:anet_a8_original_firmware.bin:r
ls -lh anet_a8_original_firmware.bin

# === Arduino IDE STARTEN (GUI) ===
arduino ~/3D-Drucker/Anet_A8_Firmware/Marlin-1.1.9/Marlin/Marlin.ino &

# === IN ARDUINO IDE ===
# Tools > Board > "Anet A8 V1.0"
# Tools > Processor > "ATmega1284 or ATmega1284p (16 MHz)"
# Tools > Programmer > "USBasp"
# Tools > Burn Bootloader
# [Warte bis fertig]
# Sketch > Verify/Compile (Ctrl+R)
# [Warte bis fertig]
# Sketch > Upload Using Programmer (Shift+Ctrl+U)
# [Warte bis fertig]

# === NACH ERFOLGREICHEM FLASH ===
# Drucker-Display: Configuration > Reset EEPROM
# Bed-Level neu kalibrieren (Paper-Method)
```

---

## 🔍 **Fehlersuche - Die häufigsten Probleme**

| **Error** | **Lösung** |
|-----------|-----------|
| `avrdude: stk500_recv(): programmer is not responding` | USBasp-Kabel überprüfen, 6-pin Anschluss korrekt? Drucker eingeschaltet? |
| `Selected serial port does not exist` | Drucker nicht angeschlossen/erkannt. Treiber installieren: `sudo apt install libusb-0.1-4:i386` |
| `Build failed - not enough space` | Marlin zu groß. Überprüfe Configuration.h, deaktiviere optionale Features |
| `THERMAL RUNAWAY ERROR` nach Flash | NORMAL! Heizer-Kabel überprüfen (sollten nicht lose sein). Ist ein Sicherheitsfeature. |
| Drucker zeigt "Omni A8" statt "Marlin" | Stock-Bootloader noch da. Führe "Burn Bootloader" erneut aus. |

---

## **Pro-Tips für Linux-User**

```bash
# Alias für häufige Befehle erstellen (in ~/.bashrc)
echo 'alias avrdude_backup="avrdude -c usbasp -p m1284p -U flash:r:firmware_backup.bin:r"' >> ~/.bashrc
echo 'alias avrdude_restore="avrdude -c usbasp -p m1284p -U flash:w:firmware_backup.bin:r"' >> ~/.bashrc
source ~/.bashrc

# Dann einfach nutzen:
# avrdude_backup
# avrdude_restore

# Drucker-Logs live anschauen (nach USB-Verbindung)
minicom -D /dev/ttyUSB0 -b 115200
# Oder: screen /dev/ttyUSB0 115200
# Drücke Ctrl+A dann X um zu beenden (minicom) oder Ctrl+A Ctrl+D (screen)
```

---

## **Weitere hilfreiche Befehle**

```bash
# Überprüfe Arduino IDE Installation
which arduino
arduino --version

# Überprüfe Installation von Board-Definitionen
ls -la ~/Arduino/hardware/anet/avr/

# Zeige aktuelle Drucker-Firmware-Größe
wc -c ~/3D-Drucker/Anet_A8_Firmware/Marlin-1.1.9/Marlin/Marlin.ino

# Listet alle seriellen Geräte auf
dmesg | grep -i "serial\|usb" | tail -20

# Überprüfe Festplatte-Speicher
df -h

# Zeige Details über USB-Geräte
lsusb -v | grep -A 5 "Future\|USBasp"
```

---

**Viel Erfolg! Wenn was Fragen entstehen, einfach die genaue Fehlermeldung posten!** 🔧
