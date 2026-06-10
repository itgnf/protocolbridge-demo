<div align="center">

# PROTOCOL**BRIDGE**

### IIoT Middleware – Modbus · OPC-UA · MQTT

[![Status](https://img.shields.io/badge/Status-Produktionsreif-brightgreen)](https://github.com/itgnf/protocolbridge-demo)
[![Tests](https://img.shields.io/badge/Tests-38%20passing-brightgreen)](#)
[![License](https://img.shields.io/badge/License-BSL%201.1-blue)](#lizenz)
[![Python](https://img.shields.io/badge/Python-3.11+-blue)](https://python.org)
[![React](https://img.shields.io/badge/React-18-61dafb)](https://react.dev)

**Fabriken sprechen verschiedene Sprachen. ProtocolBridge übersetzt.**

[Demo anfragen](#kontakt) · [API Docs](#api) · [Dokumentation](docs/)

</div>

---

## Das Problem

In modernen Fabriken sprechen Maschinen verschiedene Protokolle:

- **Modbus TCP** – ältere CNC-Maschinen und SPS
- **OPC-UA** – moderne Industriestandard-Geräte  
- **MQTT** – IoT-Sensoren und Edge Devices

Sie verstehen sich nicht. Manager sehen nicht was passiert. Proprietäre Middleware kostet tausende Euro pro Jahr.

## Die Lösung

ProtocolBridge liest alle Protokolle, normalisiert die Daten und macht sie über eine einheitliche REST API und ein Echtzeit-Dashboard verfügbar.

```
Maschinen  →  Gateway  →  TimescaleDB  →  Dashboard
Modbus         Bridge       REST API       Live-Charts
OPC-UA         Normalize    WebSocket      Alarme
MQTT           Log          Swagger        Push-Notifications
```

## Features

**Gateway**
- Liest Modbus TCP, OPC-UA und MQTT gleichzeitig
- Automatischer Reconnect bei Verbindungsabbruch
- Konfigurierbares Polling-Interval
- Strukturiertes Logging mit Statistiken

**Backend API**
- REST API mit automatischer Swagger Dokumentation
- WebSocket für Echtzeit-Updates alle 2 Sekunden
- TimescaleDB für optimierte Zeitreihendaten
- Verlaufsabfragen mit Zeitraum-Filter

**Dashboard**
- Live-Charts für Temperatur, RPM und Druck
- Alarm-System mit Rot/Gelb/Grün Status
- Browser Push-Benachrichtigungen
- Responsives Design

**Qualität**
- 38 Unit Tests (ModbusReader, MQTTPublisher, Bridge)
- GitHub Actions CI Pipeline
- Vollständige API Dokumentation
- Setup-Guide mit Fehlerbehandlung

## Tech Stack

| Bereich | Technologie |
|---------|------------|
| Protokolle | pymodbus 3.6.4 · asyncua · paho-mqtt |
| Backend | Python · FastAPI · TimescaleDB · WebSocket |
| Frontend | React · Vite · Tailwind CSS · Recharts |
| Infra | Docker Compose · GitHub Actions · pytest |

## API Endpoints

```
GET  /health                     → System Status
GET  /machines                   → Alle Maschinen
GET  /machines/{id}              → Maschinenstatus
GET  /machines/{id}/history      → Verlauf (bis 24h)
POST /alerts                     → Alarm konfigurieren
WS   /ws/machines                → Live-Updates
```

Vollständige Dokumentation: [docs/05_API_Dokumentation.md](docs/05_API_Dokumentation.md)

## Preismodell

| Paket | Preis | Maschinen |
|-------|-------|-----------|
| Starter | 99 € / Monat | bis 5 Maschinen |
| Professional | 199 € / Monat | bis 20 Maschinen |
| Enterprise | 299 € / Monat | unbegrenzt |

## Lizenz

Dieses Repository enthält die öffentliche Dokumentation von ProtocolBridge.

Der Quellcode ist unter der [Business Source License 1.1](LICENSE.txt) verfügbar. Kommerzielle Nutzung erfordert eine separate Lizenz.

## Kontakt

**Jonas** · itgnf  
jonasitgnf@gmail.com  
[github.com/itgnf](https://github.com/itgnf)

---

<div align="center">
<sub>Gebaut mit Python · FastAPI · React · Docker · TimescaleDB</sub>
</div>
