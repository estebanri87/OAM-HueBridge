# Applikationsbeschreibung OpenKNX Hue Bridge

## Wichtige Hinweise

* Diese KNXprod wird nicht von der KNX Association offiziell unterstützt!
* Die Erzeugung der KNXprod geschieht auf eigene Verantwortung!
* Diese Firmware ist aktuell in Entwicklung (Version 0.1.0 - Proof of Concept)

## Module

Die OpenKNX Hue Bridge besteht aus folgenden Modulen:

- [Basiseinstellungen](https://github.com/OpenKNX/OGM-Common/blob/v1/doc/Applikationsbeschreibung-Common.md)
- [Netzwerk](https://github.com/OpenKNX/OFM-Network/blob/v1/doc/Applikationsbeschreibung-Netzwerk.md)
- [Hue Bridge](https://github.com/estebanri87/OFM-HueBridgeModule/blob/v1/docs/Applikationsbeschreibung.md)
- [Logik](https://github.com/OpenKNX/OFM-LogicModule/blob/v1/doc/Applikationsbeschreibung-Logik.md)
- [Funktionsblöcke](https://github.com/OpenKNX/OFM-FunctionBlocks/blob/v1/doc/Applikationsbeschreibung-FunctionBlocks.md)





## Übersicht

Die **OpenKNX Hue Bridge** ist eine Firmware für ESP32-basierte OpenKNX-Geräte, die Philips Hue Beleuchtungssysteme in KNX-Installationen integriert. Das Gerät verbindet sich mit einer vorhandenen Philips Hue Bridge und stellt die Hue-Lampen über KNX-Kommunikationsobjekte zur Verfügung.

### Systemübersicht

```
┌─────────────────┐     Hue API v2      ┌──────────────────┐      KNX       ┌─────────┐
│  Philips Hue    │ ◄─────────────────► │  OpenKNX Hue     │ ◄───────────► │ KNX Bus │
│  Bridge         │  HTTP/JSON/SSE      │  Bridge (ESP32)  │               │         │
└─────────────────┘                     └──────────────────┘               └─────────┘
      │                                         │
      ├── Hue Lampen                            ├── KO: Schalten
      ├── Hue LED Stripes                       ├── KO: Helligkeit
      ├── Hue Spots                             ├── KO: Farbtemperatur
      └── Hue Panels                            └── KO: RGB Farbe
```

**Datenfluss**: `Philips Hue Bridge ↔ ESP32/HueBridge Module ↔ KNX Bus`

Die Integration ist **bidirektional**:
- KNX-Befehle werden an die Hue-Lampen weitergeleitet
- Statusänderungen von Hue-Lampen werden an den KNX-Bus gemeldet
- Änderungen in der Hue-App werden automatisch auf dem KNX-Bus synchronisiert

## Features

### Phase 1 (aktuell implementiert)

- ✅ **Automatische Bridge-Erkennung** via mDNS (`_hue._tcp.local`)
- ✅ **Button-Press-Authentifizierung** für sicheren Zugriff auf die Hue Bridge
- ✅ **Bidirektionale Status-Synchronisation** via Hue Event Stream (SSE)
- ✅ **Unterstützung für bis zu 20 Hue-Lampen-Kanäle**
- ✅ **Vollständige Lampensteuerung**:
  - Ein/Aus-Schalten
  - Dimmen (0-100%)
  - Farbtemperatur (2000-6500 Kelvin)
  - RGB-Farben
- ✅ **ETS-Konfiguration** über OpenKNXproducer
- ✅ **Web-Server** zur UUID-Entdeckung und Diagnostik
- ✅ **mDNS-Service** (`openknx-bridge.local`)

### Geplante Features (Phase 2+)

- Hue-Sensoren (Bewegung, Temperatur, Helligkeit)
- Hue-Schalter und Dimmer
- Szenen-Unterstützung
- Gruppen-Steuerung
- Zeitgesteuerte Aktionen
- Erweiterte Farbsteuerung (HSV)

## Module

Die OpenKNX Hue Bridge besteht aus folgenden Modulen:

### Basis-Module

- **[Basiseinstellungen](https://github.com/OpenKNX/OGM-Common/blob/v1/doc/Applikationsbeschreibung-Common.md)**  
  Grundlegende OpenKNX-Funktionen, Firmware-Update, Diagnose

- **[Netzwerk](https://github.com/OpenKNX/OFM-Network/blob/v1/doc/Applikationsbeschreibung-Netzwerk.md)**  
  Ethernet (LAN) oder WLAN-Verbindung, NTP-Zeitserver

- **[Konfigurationsübertragung](https://github.com/OpenKNX/OFM-ConfigTransfer)**  
  Backup und Restore der Konfiguration

- **[Dateiübertragung](https://github.com/OpenKNX/OFM-FileTransferModule)**  
  Übertragung von Dateien zwischen ETS und Gerät

### Funktions-Module

- **[Philips Hue Integration](../lib/OFM-HueBridgeModule/README.md)** ⭐  
  Kernfunktionalität für Hue-Integration (Dokumentation im Modul)

- **[Logik](https://github.com/OpenKNX/OFM-LogicModule/blob/v1/doc/Applikationsbeschreibung-Logik.md)**  
  50 Logikkanäle zur Verknüpfung von KNX-Objekten

- **[Funktionsblöcke](https://github.com/OpenKNX/OFM-FunctionBlocks/blob/v1/doc/Applikationsbeschreibung-FunctionBlocks.md)**  
  15 Funktionsblöcke für Timer, Zähler, Rechner, etc.

## Inbetriebnahme

### Voraussetzungen

1. **Hardware**: 
   - ESP32-basiertes OpenKNX-Gerät mit Ethernet oder WLAN
   - Unterstützt: REG1-LAN-TP-Base, Adafruit ESP32 Feather V2
   
2. **Philips Hue System**:
   - Philips Hue Bridge (v2 oder Square)
   - Mindestens eine Hue-Lampe
   - Hue Bridge im selben Netzwerk wie OpenKNX-Gerät

### Ersteinrichtung

#### Schritt 1: Firmware flashen
1. Vorkompilierte Firmware von [GitHub Releases](https://github.com/OpenKNX/OAM-HueBridge/releases) herunterladen
2. Firmware auf ESP32-Gerät flashen (siehe README im Release)
3. Gerät mit Netzwerk (LAN oder WLAN) verbinden

#### Schritt 2: ETS-Projekt konfigurieren
1. KNXprod-Datei in ETS importieren
2. Gerät in ETS-Projekt einfügen
3. Physikalische Adresse zuweisen
4. **Philips Hue Integration** aktivieren:
   - Im Reiter "Philips Hue" die Integration einschalten
   - Bridge-Erkennung: Automatisch (empfohlen) oder manuell mit IP-Adresse
5. Programmierung auf das Gerät übertragen

#### Schritt 3: Hue Bridge koppeln (Authentifizierung)
Nach dem ersten Start der Firmware:

1. **LED am OpenKNX-Gerät beobachten**:
   - Schnelles Blinken = Wartet auf Button-Press der Hue Bridge
   
2. **Button auf Hue Bridge drücken**:
   - Innerhalb von 30 Sekunden den Link-Button auf der Hue Bridge drücken
   - Button befindet sich auf der Oberseite der Bridge
   
3. **Verbindung bestätigen**:
   - LED wechselt zu langsamem Blinken = Verbindung hergestellt
   - App-Key wird automatisch generiert und gespeichert
   - Keine weitere Authentifizierung bei Neustart erforderlich

4. **Geräte-Scan**:
   - Alle Hue-Lampen werden automatisch erkannt
   - Liste kann in ETS abgerufen werden (via Konfigurationsübertragung)

#### Schritt 4: Hue-Kanäle konfigurieren
1. In ETS zu "Philips Hue" → "Kanal 1" navigieren
2. **Kanal aktivieren** und Hue-Lampe auswählen aus Dropdown
3. **Kommunikationsobjekte** den gewünschten Gruppenadressen zuweisen:
   - Schalten (DPT 1.001)
   - Helligkeit (DPT 5.001) - optional
   - Farbtemperatur (DPT 7.600) - optional
   - RGB Farbe (DPT 232.600) - optional
4. Schritte für weitere Lampen wiederholen (max. 20 Kanäle)
5. Konfiguration auf Gerät übertragen

## Konfiguration

### Allgemeine Einstellungen (Philips Hue)

| Parameter | Beschreibung | Standardwert |
|-----------|--------------|--------------|
| **Integration aktivieren** | Aktiviert das Hue-Modul | Nein |
| **Bridge-Erkennung** | Automatisch (mDNS) oder manuell | Automatisch |
| **Bridge IP-Adresse** | Manuelle IP-Eingabe bei Bedarf | Auto erkannt |
| **App-Key** | Wird automatisch generiert | (leer) |
| **Update-Intervall** | Fallback-Polling in Sekunden | 5s |
| **Verbindungs-Timeout** | Timeout für Bridge-Verbindung | 30s |
| **Anzahl Kanäle** | Aktive Lampen-Kanäle | 1-20 |

### Kanal-Einstellungen (pro Hue-Lampe)

| Parameter | Beschreibung |
|-----------|--------------|
| **Kanal aktiv** | Aktiviert diesen Kanal |
| **Hue-Gerät** | Wahl der Hue-Lampe aus erkannten Geräten |
| **Geräte-Name** | Anzeigename (aus Hue übernommen, editierbar) |
| **Schalten KO** | Ein/Aus (DPT 1.001) |
| **Helligkeit KO** | Dimmen 0-100% (DPT 5.001) |
| **Farbtemperatur KO** | Warmweiß bis Kaltweiß (DPT 7.600) |
| **RGB Farbe KO** | RGB-Farbwert (DPT 232.600) |
| **Status-Rückmeldung** | Sendet Status bei Änderung |
| **Bei Neustart Status einlesen** | Liest aktuellen Status von Hue |
| **Übergangszeit** | Sanfte Übergänge in Sekunden |

## Unterstützte Hue-Geräte

### Lampentypen

| Typ | Schalten | Dimmen | Farbtemperatur | RGB | Status |
|-----|----------|--------|----------------|-----|--------|
| **White** | ✅ | ❌ | ❌ | ❌ | ✅ |
| **White Ambiance** | ✅ | ✅ | ✅ | ❌ | ✅ |
| **White and Color** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Hue Go / Bloom** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **LED Stripes** | ✅ | ✅ | ✅ | ✅ | ✅ |

### Beispiel-Produkte
- Philips Hue White (E27, GU10)
- Philips Hue White Ambiance
- Philips Hue White and Color Ambiance
- Philips Hue Lightstrip Plus
- Philips Hue Go
- Philips Hue Bloom
- Philips Hue Spot / Bar

## Kommunikationsobjekte

### Beispiel: Farblampe (Hue White and Color)

| KO | Name | DPT | Flags | Beschreibung |
|----|------|-----|-------|--------------|
| 1 | Schalten | 1.001 | K, S, Ü | Ein/Aus Steuerung |
| 2 | Helligkeit | 5.001 | K, S, Ü | Helligkeit 0-100% |
| 3 | Farbtemperatur | 7.600 | K, S, Ü | Kelvin (2000-6500K) |
| 4 | RGB Farbe | 232.600 | K, S, Ü | RGB Farbwert |
| 5 | Status Schalten | 1.001 | K, Ü | Rückmeldung Ein/Aus |
| 6 | Status Helligkeit | 5.001 | K, Ü | Rückmeldung Helligkeit |
| 7 | Status Erreichbar | 1.001 | K, Ü | Lampe online? |
| 8 | Verbindungsfehler | 1.005 | K, Ü | Bridge nicht erreichbar |

**Flags**: K = Kommunikation, S = Schreiben, Ü = Übertragen, L = Lesen

### Nutzungsbeispiele

#### Einfaches Schalten
```
KNX GA 1/2/1 (Schalten) → Hue Lampe Ein
Hue App: Lampe ausschalten → KNX GA 1/2/5 (Status) = Aus
```

#### Dimmen mit Farbtemperatur
```
KNX GA 1/2/2 (Helligkeit) = 50% → Hue Lampe 50%
KNX GA 1/2/3 (Farbtemperatur) = 2700K → Warmweißes Licht
```

#### RGB-Steuerung
```
KNX GA 1/2/4 (RGB) = #FF0000 → Rote Farbe
Hue App: Farbe ändern zu Blau → KNX GA 1/2/4 = #0000FF
```

## Fehlerbehebung

### Hue Bridge wird nicht gefunden

**Problem**: Automatische Bridge-Erkennung schlägt fehl

**Lösungen**:
1. Prüfen ob Hue Bridge und OpenKNX-Gerät im selben Netzwerk sind
2. mDNS-Dienst im Router aktiviert?
3. Manuelle IP-Adresse der Hue Bridge in ETS eintragen
4. Firewall-Einstellungen prüfen (Port 80/443)

### Authentifizierung schlägt fehl

**Problem**: Button-Press auf Hue Bridge funktioniert nicht

**Lösungen**:
1. Button innerhalb 30 Sekunden nach ESP32-Start drücken
2. LED-Status am OpenKNX-Gerät beobachten (schnelles Blinken = bereit)
3. Hue Bridge Neustart versuchen
4. Bereits existierende App-Keys prüfen (max. 20 Apps erlaubt)
5. In Hue App → Einstellungen → Hue Bridge → Geräte verwalten

### Lampen reagieren nicht

**Problem**: KNX-Befehle werden nicht ausgeführt

**Lösungen**:
1. Prüfen ob Hue Bridge erreichbar ist (KO "Status Erreichbar")
2. Korrekte Hue-Geräte-ID im Kanal konfiguriert?
3. Lampe in Hue App steuerbar?
4. Event Stream aktiv? (Console-Log prüfen)

### Status-Updates fehlen

**Problem**: Änderungen in Hue App erscheinen nicht auf KNX

**Lösungen**:
1. Event Stream (SSE) Verbindung prüfen
2. Polling-Intervall in ETS verringern (z.B. 5s)
3. Netzwerk-Stabilität prüfen
4. Status KO in ETS aktiviert und mit GA verknüpft?

## Technische Details

### Hue API Version
- Verwendet **Hue API v2** (CLIP API)
- Event Stream via **Server-Sent Events (SSE)**
- HTTPS/HTTP REST API

### Netzwerk-Anforderungen
- Ethernet (empfohlen) oder WLAN
- Port 80 und 443 für Hue Bridge
- mDNS für automatische Discovery

### Performance
- Event Stream: Echtzeit-Updates (< 100ms)
- Fallback Polling: 1-60 Sekunden konfigurierbar
- Maximale Kanäle: 20 Lampen gleichzeitig

### Speicher
- App-Key wird im Flash gespeichert (persistent)
- Überlebt Neustarts und Firmware-Updates
- Zurücksetzen via OpenKNX-Konsole möglich

## Hardware

### Unterstützte Geräte

#### REG1-LAN-TP-Base
- ESP32-basiertes OpenKNX Hutschienen-Gerät
- Ethernet (RJ45)
- KNX TP-Bus Anbindung
- [Mehr Informationen](http://device.openknx.de/REG1-LAN-TP-Base)

#### Adafruit ESP32 Feather V2
- ESP32-S3 Development Board
- WLAN (2.4 GHz)
- KNX TP-Kopplung erforderlich
- [Mehr Informationen](https://github.com/OpenKNX/OpenKNX/wiki/Adafruit-ESP32-Feather-V2)

**Optional**: Zusätzliche Prog-Taster und LEDs an GPIO 7, 8, 20, 22

### Hardware-Anforderungen
- ESP32 oder ESP32-S3 Mikrocontroller
- Mindestens 4 MB Flash
- Ethernet oder WLAN
- KNX TP-Kopplung (optional)

## Weiterführende Links

### Dokumentation
- [GitHub Repository](https://github.com/OpenKNX/OAM-HueBridge)
- [Release Downloads](https://github.com/OpenKNX/OAM-HueBridge/releases)
- [OpenKNX Wiki](https://github.com/OpenKNX/OpenKNX/wiki)
- [Hue Module Dokumentation](../lib/OFM-HueBridgeModule/README.md)

### OpenKNX
- [OpenKNX Hauptseite](https://openknx.de)
- [OpenKNX GitHub](https://github.com/OpenKNX)
- [OpenKNX Forum](https://knx-user-forum.de)

### Philips Hue
- [Hue Developer Portal](https://developers.meethue.com)
- [Hue API v2 Documentation](https://developers.meethue.com/develop/hue-api-v2/)

## Lizenz

Diese Software steht unter der **GNU GPL v3** Lizenz.  
Siehe [LICENSE](../LICENSE) für Details.

---

**Stand**: Februar 2026 | **Version**: 0.1.0 (Beta)




