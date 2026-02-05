# OpenKNX Hue Bridge

Die OpenKNX Hue Bridge integriert Philips Hue Lampen in das KNX-System.

## Features

- Bidirektionale Synchronisation zwischen KNX und Philips Hue
- WebServer zur UUID-Entdeckung der Hue Lampen
- mDNS-Service (openknx-bridge.local)
- Unterstützung für bis zu 20 Hue Lampen-Kanäle
- Konfigurierbar über ETS

### Unterstützte Lampentypen

- Schaltbar (On/Off)
- Dimmbar
- Farbtemperatur
- RGB-Farbe

## Anwenderdokumentation

Die Anwenderdokumentation ist [hier](./doc/Applikationsbeschreibung.md) zu finden.

## Installation

Eine vorkomplierte Firmware ist [hier](https://github.com/OpenKNX/OAM-Hue/releases) zu finden. ZIP Datei herunterladen, entpacken und der Anleitung im Readme folgen.

## Hardware

Als Hardware kann jede OpenKNX oder OpenKNX-Ready Hardware mit LAN oder WLAN verwendet werden.
Die vorkompilierte Firmware unterstützt:

- [REG1-LAN-TP-Base](http://device.openknx.de/REG1-LAN-TP-Base)
- [Adafruit ESP32 Feather V2](https://github.com/OpenKNX/OpenKNX/wiki/Adafruit-ESP32-Feather-V2)

### Optional bei Adafruit ESP32 Feather V2: Zusätzlicher Prog Taster und LED

An Pin GPIO 7 (RX) und/oder GPIO 20 (am Stecker) kann jeweils ein zusätzlicher Taster angeschlossen werden. Dieser muss gegen GND schalten.

An PIN GPIO 8 (TX) und/oder GPIO 22 (am Stecker) kann mit einem 100 Ohm Wiederstand eine LED (Anode) angeschlossen werden. Die Kathode mit GND verbinden.

## Lizenz

Diese Software steht unter der [GNU GPL v3](LICENSE).
