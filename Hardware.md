# Hardware

## Drucker

| Komponente    | Details        |
|---------------|----------------|
| Modell        | Anet A8        |
| Prozessor     | ATmega1284p (8-Bit) |
| Board         | Anet A8 v1.0   |

## ISP-Programmer

| Komponente | Details                  |
|------------|--------------------------|
| Modell     | USBasp                   |
| Anschluss  | J3-Header am Anet A8 Board |

### J3 Pinbelegung am Anet A8 Board Pin 1 ist unten rechts, wenn die Nase rechts ist

| Pin | Signal | Testpunkt |
|-----|--------|-----------|
| 3   | MISO   | T27       |
| 4   | VCC    | —         |
| 5   | SCK    | T23       |
| 6   | MOSI   | T25       |
| 7   | RST    | —         |
| 8   | GND    | —         |

## Recovery-Hardware

| Komponente    | Details                        |
|---------------|--------------------------------|
| Arduino Nano  | Für ArduinoISP und HVPP-Recovery |
| Adapterplatte | Für Arduino Nano               |
