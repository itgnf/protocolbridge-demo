# 🔌 ProtocolBridge

> **IIoT Middleware – Maschinen reden endlich miteinander**

ProtocolBridge verbindet industrielle Maschinen die verschiedene Protokolle sprechen (Modbus, OPC-UA, MQTT) und macht alle Daten über eine einheitliche REST API und ein Echtzeit-Dashboard zugänglich.

---

## 🎯 Das Problem

In deutschen Fabriken sprechen Maschinen verschiedene "Sprachen":

```
CNC-Maschine   → Modbus RTU   ─╗
Roboterarm     → Profibus     ─╬─→ ??? → niemand sieht was passiert
Temperatursensor → OPC-UA     ─╝
```

Kein Manager weiß in Echtzeit was in der Fabrik passiert.  
Ausfälle werden zu spät erkannt. Wartung ist reaktiv statt vorausschauend.

## ✅ Die Lösung

```
Alle Protokolle → ProtocolBridge → Eine API → Ein Dashboard
```

---

## 🏗️ Architektur

```
[Modbus Maschinen]  ──┐
[OPC-UA Maschinen]  ──┼──→ [Gateway] ──→ [MQTT Broker]
[weitere...]        ──┘                        │
                                               ▼
                                        [TimescaleDB]
                                               │
                                               ▼
                                        [FastAPI Backend]
                                               │
                                               ▼
                                      [React Dashboard]
```

---

## 🚀 Schnellstart (mit Simulatoren)

### Voraussetzungen
- Python 3.11+
- Docker & Docker Compose
- Node.js 20+

### Installation

```bash
# 1. Repository klonen
git clone https://github.com/itgnf/protocolbridge.git
cd protocolbridge

# 2. Python Abhängigkeiten installieren
pip install -r requirements.txt

# 3. Alle Services starten
docker-compose up -d

# 4. Simulatoren starten (kein echte Hardware nötig)
python src/gateway/modbus_simulator.py &
python src/gateway/opcua_simulator.py &

# 5. Gateway starten
python src/gateway/bridge.py

# 6. Dashboard öffnen
# http://localhost:3000
```

---

## 📁 Projektstruktur

```
protocolbridge/
│
├── 📁 src/
│   ├── gateway/          ← Protokoll-Treiber & Bridge
│   ├── api/              ← FastAPI REST Backend
│   └── dashboard/        ← React Frontend
│
├── 📁 docs/
│   ├── 01_Anforderungen.md
│   ├── 02_Architektur.md
│   ├── 03_Risikoanalyse.md
│   ├── 04_Testplan.md
│   └── 05_API_Dokumentation.md
│
├── 📁 tests/             ← Automatische Tests
├── 📁 docker/            ← Docker Konfiguration
├── docker-compose.yml    ← Alle Services
└── requirements.txt      ← Python Abhängigkeiten
```

---

## 🔌 Unterstützte Protokolle

| Protokoll | Status | Version |
|---|---|---|
| Modbus TCP | ✅ Implementiert | Phase 1 |
| Modbus RTU | ✅ Implementiert | Phase 1 |
| OPC-UA | ✅ Implementiert | Phase 2 |
| MQTT (Broker) | ✅ Implementiert | Phase 1 |
| Profibus | 🔜 Geplant | Phase 3 |
| BACnet | 🔜 Geplant | Phase 3 |

---

## 📡 API Endpunkte

```
GET  /health                    → System-Status
GET  /machines                  → Alle Maschinen
GET  /machines/{id}             → Maschinenstatus
GET  /machines/{id}/history     → Historische Daten
POST /alerts                    → Alarm erstellen
GET  /alerts                    → Alle Alarme
WS   /ws/machines/{id}          → Echtzeit-Stream
```

📖 Vollständige API Dokumentation: `http://localhost:8000/docs`

---

## 🧪 Tests ausführen

```bash
# Alle Tests
pytest tests/

# Mit Coverage-Bericht
pytest tests/ --cov=src --cov-report=html
```

---

## 🛠️ Tech Stack

| Bereich | Technologie |
|---|---|
| Gateway | Python + pymodbus + asyncua |
| Message Broker | Mosquitto (MQTT) |
| Backend | FastAPI + Python |
| Datenbank | TimescaleDB (PostgreSQL) |
| Frontend | React + Recharts + Tailwind |
| Container | Docker + Docker Compose |
| Tests | pytest + GitHub Actions |

---

## 📅 Roadmap

- [x] Phase 1 – Anforderungen & Architektur
- [ ] Phase 2 – Simulatoren aufsetzen
- [ ] Phase 3 – Gateway Core entwickeln
- [ ] Phase 4 – Backend API
- [ ] Phase 5 – React Dashboard
- [ ] Phase 6 – Dokumentation & Release

---

## 📄 Dokumentation

Alle Dokumente befinden sich im `/docs` Ordner:

- [Anforderungsanalyse](docs/01_Anforderungen.md)
- [Systemarchitektur](docs/02_Architektur.md)
- [Risikoanalyse](docs/03_Risikoanalyse.md)
- [Testplan](docs/04_Testplan.md)
- [API Dokumentation](docs/05_API_Dokumentation.md)

---

## 👤 Autor

**itgnf**  
GitHub: [@itgnf](https://github.com/itgnf)

---

## 📜 Lizenz

MIT License – siehe [LICENSE](LICENSE)
