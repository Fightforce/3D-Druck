# Anet A8 — 3D-Druck Dokumentation

Dieses Repository dient als Dokumentation und Arbeitsgrundlage rund um den Anet A8 3D-Drucker.

## Inhalt

| Ordner/Datei | Beschreibung |
|---|---|
| `Anet A8 neu aufsetzten 2026/` | Deutsche Schritt-für-Schritt-Anleitungen |
| `CLAUDE.md` | Anweisungen für Claude Code (Build, Flash, Konfiguration) |
| `Hardware.md` | Übersicht der verwendeten Hardware |
| `Software.md` | Übersicht der installierten und verwendeten Software |
| `backup_eeprom.hex` | EEPROM-Backup vom Original-Drucker |
| `backup_stock_firmware.hex` | Original-Firmware-Backup |
| `tmp/` | Zum lokalen Speichern von temporären Dateien |

## Fortschritt

### Abgeschlossen
- [x] Claude Code Terminal eingerichtet
- [x] Original-Firmware und EEPROM gesichert
- [x] USBasp ISP-Programmer erworben
- [x] Anet A8 beim Flashen gebrickt (MISO-Pin wahrscheinlich durch ESD beschädigt)

### Ausstehend
- [ ] Arduino Nano per ArduinoISP anschließen → ISP-Recovery versuchen
- [ ] Falls ISP scheitert: HVPP mit Arduino Nano und 12V
- [ ] Marlin 1.1.9 auf den Anet A8 flashen
- [ ] EEPROM zurücksetzen (M502 / M500)
