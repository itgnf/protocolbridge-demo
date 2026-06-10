# ProtocolBridge

> IIoT Middleware – Modbus, OPC-UA und MQTT in einer REST API und einem Echtzeit-Dashboard vereint.

[![CI](https://github.com/itgnf/protocolbridge/actions/workflows/ci.yml/badge.svg)](https://github.com/itgnf/protocolbridge/actions)
![Python](https://img.shields.io/badge/Python-3.11+-blue)
![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-green)
![React](https://img.shields.io/badge/React-18-61dafb)
![Tests](https://img.shields.io/badge/Tests-38%20passing-brightgreen)

---

## Das Problem

In Fabriken sprechen Maschinen verschiedene Protokolle – Modbus, OPC-UA, MQTT. Sie verstehen sich nicht gegenseitig. Manager sehen nicht, was in der Fabrik passiert. Proprietäre Middleware kostet tausende Euro.

## Die Lösung

ProtocolBridge liest alle Protokolle, normalisiert die Daten und zeigt alles in einem einheitlichen Dashboard und einer REST API an.

**Preismodell:** 99 – 299 € / Monat pro Fabrik

---

## Architektur

```
Maschinen (Modbus/OPC-UA/MQTT)
        ↓
Gateway (ModbusReader → Bridge → MQTTPublisher)
        ↓
Backend (MQTT Subscriber → TimescaleDB → FastAPI)
        ↓
Frontend (React Dashboard → Live-Charts → Alarme)
```

---

## Tech Stack

| Bereich | Technologie |
|---------|------------|
| Protokolle | pymodbus 3.6.4, asyncua, paho-mqtt |
| Backend | Python, FastAPI, TimescaleDB |
| Frontend | React, Vite, Tailwind CSS, Recharts |
| Infra | Docker Compose, GitHub Actions, pytest |

---

## Schnellstart

**Voraussetzungen:** Python 3.11+, Docker Desktop, Node.js 18+, Git

```bash
# 1. Repository klonen
git clone https://github.com/itgnf/protocolbridge.git
cd Protocolbridge

# 2. Python Pakete installieren
python -m pip install pymodbus==3.6.4 asyncua paho-mqtt fastapi uvicorn psycopg2-binary

# 3. Docker starten
docker-compose up -d

# 4. Simulatoren starten
python src/gateway/modbus_simulator.py &
python src/gateway/bridge.py &
python src/api/subscriber.py &

# 5. Backend starten
cd src/api && uvicorn main:app --port 8000 &

# 6. Frontend starten
cd frontend && npm install && npm run dev
```

**URLs:**
- Dashboard: http://localhost:5173
- API Docs: http://localhost:8000/docs
- Health: http://localhost:8000/health

---

## Tests

```bash
pytest tests/ -v
# 38 passed
```

---

## Projektstruktur

```
Protocolbridge/
├── src/
│   ├── gateway/          # Modbus, OPC-UA, MQTT Bridge
│   └── api/              # FastAPI Backend + Subscriber
├── frontend/             # React Dashboard
├── tests/                # 38 Unit Tests
├── docs/                 # Dokumentation
└── docker-compose.yml    # TimescaleDB + Mosquitto
```

---

## Entwickler

**Jonas** · [@itgnf](https://github.com/itgnf) · jonasitgnf@gmail.com
