# 🏗️ Systemarchitektur – ProtocolBridge
**Version:** 1.0  
**Datum:** Mai 2026  
**Autor:** itgnf  
**Status:** In Bearbeitung

---

## 1. Architektur-Übersicht

ProtocolBridge besteht aus 5 Schichten (Layers). Jede Schicht hat eine klare Aufgabe und kommuniziert nur mit der direkt benachbarten Schicht.

```
┌─────────────────────────────────────────────────┐
│              LAYER 5: DASHBOARD                  │
│         React Web-App  |  Browser/Handy          │
└──────────────────────┬──────────────────────────┘
                       │ HTTP / WebSocket
┌──────────────────────▼──────────────────────────┐
│              LAYER 4: BACKEND API                │
│         FastAPI  |  REST  |  Swagger Docs        │
└──────────┬─────────────────────────┬────────────┘
           │ SQL                     │ Subscribe
┌──────────▼──────────┐   ┌──────────▼────────────┐
│  LAYER 3: DATENBANK │   │  LAYER 3: MQTT BROKER  │
│    TimescaleDB      │   │      Mosquitto         │
└─────────────────────┘   └──────────┬─────────────┘
                                     │ Publish
┌────────────────────────────────────▼────────────┐
│              LAYER 2: GATEWAY                    │
│   Modbus Reader  |  OPC-UA Reader  |  Publisher  │
└──────────┬──────────────────────┬───────────────┘
           │ Modbus TCP/RTU       │ OPC-UA
┌──────────▼──────────┐  ┌───────▼───────────────┐
│  LAYER 1: MASCHINEN │  │  LAYER 1: MASCHINEN    │
│  CNC, Pumpen,       │  │  Roboter, Sensoren     │
│  Förderbänder       │  │  Steuerungen           │
└─────────────────────┘  └────────────────────────┘
```

---

## 2. Komponenten im Detail

### Layer 1 – Maschinen-Ebene
Physische Maschinen und Geräte in der Fabrik. Sie kommunizieren über industrielle Protokolle.

| Protokoll | Beschreibung | Typische Geräte |
|---|---|---|
| Modbus TCP | Ethernet-basiert, Port 502 | CNC-Maschinen, Pumpen |
| Modbus RTU | Seriell über RS-485 | Alte Steuerungen, Sensoren |
| OPC-UA | Modern, sicher, selbstbeschreibend | Neue Maschinen, SPS |

---

### Layer 2 – Gateway
Das Herzstück. Läuft auf einem kleinen PC (Raspberry Pi oder Industrie-PC) direkt in der Fabrik.

**Dateien:**
```
src/gateway/
├── modbus_simulator.py   ← Simuliert Modbus-Maschine (Entwicklung)
├── opcua_simulator.py    ← Simuliert OPC-UA-Maschine (Entwicklung)
├── modbus_reader.py      ← Liest echte Modbus-Daten
├── opcua_reader.py       ← Liest echte OPC-UA-Daten
├── mqtt_publisher.py     ← Sendet Daten an MQTT Broker
└── bridge.py             ← Hauptprogramm: verbindet alles
```

**Datenformat (JSON) das Gateway sendet:**
```json
{
  "machine_id": "CNC-001",
  "protocol": "modbus",
  "timestamp": "2026-05-15T11:00:00Z",
  "values": {
    "temperature": 78.3,
    "rpm": 2400,
    "pressure": 4.2,
    "status": "RUNNING"
  }
}
```

---

### Layer 3 – MQTT Broker (Mosquitto)
Empfängt alle Nachrichten vom Gateway und verteilt sie weiter.

**Topic-Struktur:**
```
factory/
├── CNC-001/data      → Maschinendaten
├── CNC-001/alarm     → Alarmmeldungen
├── Robot-002/data    → Maschinendaten
└── Pump-003/data     → Maschinendaten
```

---

### Layer 3 – Datenbank (TimescaleDB)
Speichert alle Zeitreihendaten dauerhaft.

**Tabellen-Schema:**
```sql
-- Maschinendaten
CREATE TABLE machine_data (
  time        TIMESTAMPTZ NOT NULL,
  machine_id  TEXT NOT NULL,
  temperature DOUBLE PRECISION,
  rpm         INTEGER,
  pressure    DOUBLE PRECISION,
  status      TEXT
);

-- Alarme
CREATE TABLE alarms (
  time        TIMESTAMPTZ NOT NULL,
  machine_id  TEXT NOT NULL,
  type        TEXT,
  value       DOUBLE PRECISION,
  threshold   DOUBLE PRECISION,
  acknowledged BOOLEAN DEFAULT FALSE
);
```

---

### Layer 4 – Backend API (FastAPI)
Stellt Daten für das Dashboard bereit.

**Endpunkte:**
```
GET  /health                          → System-Status
GET  /machines                        → Alle Maschinen
GET  /machines/{id}                   → Maschinenstatus
GET  /machines/{id}/history           → Historische Daten
POST /alerts                          → Alarm erstellen
GET  /alerts                          → Alle Alarme
PUT  /alerts/{id}/acknowledge         → Alarm bestätigen
WS   /ws/machines/{id}                → Echtzeit-Stream
```

**Dateien:**
```
src/api/
├── main.py          ← FastAPI App Entry Point
├── routes/
│   ├── machines.py  ← Maschinen-Endpunkte
│   └── alerts.py    ← Alarm-Endpunkte
├── models.py        ← Datenbankmodelle
├── database.py      ← DB-Verbindung
└── subscriber.py    ← MQTT → Datenbank
```

---

### Layer 5 – Dashboard (React)
Web-Oberfläche für den Fabrikbetreiber.

**Seiten:**
```
/                → Übersicht aller Maschinen
/machines/{id}   → Detailansicht einer Maschine
/alerts          → Alle aktiven Alarme
/settings        → Schwellenwerte konfigurieren
```

**Dateien:**
```
src/dashboard/
├── src/
│   ├── pages/
│   │   ├── Overview.jsx     ← Alle Maschinen
│   │   ├── Machine.jsx      ← Maschinendetail
│   │   └── Alerts.jsx       ← Alarme
│   ├── components/
│   │   ├── MachineCard.jsx  ← Status-Karte
│   │   ├── LiveChart.jsx    ← Echtzeit-Graph
│   │   └── AlarmBadge.jsx   ← Alarm-Anzeige
│   └── App.jsx
```

---

## 3. Datenfluss (Step by Step)

```
1. Maschine (CNC-001) hat Temperatur 85°C
       ↓
2. Gateway liest Wert via Modbus alle 2 Sekunden
       ↓
3. Gateway sendet JSON an MQTT Broker
   Topic: factory/CNC-001/data
       ↓
4. API-Subscriber empfängt Nachricht
   → Speichert in TimescaleDB
   → Prüft Schwellenwerte (> 90°C = Alarm)
       ↓
5. Dashboard empfängt via WebSocket
   → Graph aktualisiert sich live
   → Status bleibt Grün (85°C < 90°C)
```

---

## 4. Deployment (Docker Compose)

```
docker-compose up
→ startet alle 5 Services gleichzeitig:
   ✅ Mosquitto (MQTT Broker)
   ✅ TimescaleDB (Datenbank)
   ✅ Gateway (Python)
   ✅ API (FastAPI)
   ✅ Dashboard (React)
```

---

## 5. Änderungshistorie

| Datum | Version | Änderung |
|---|---|---|
| Mai 2026 | 1.0 | Erstversion erstellt |
