# 📋 Anforderungsanalyse – ProtocolBridge
**Version:** 1.0  
**Datum:** Mai 2026  
**Autor:** itgnf  
**Status:** In Bearbeitung

---

## 1. Projektbeschreibung

ProtocolBridge ist eine IIoT-Middleware (Industrial Internet of Things) die verschiedene industrielle Kommunikationsprotokolle liest, normalisiert und über eine einheitliche REST API sowie ein Echtzeit-Dashboard zugänglich macht.

**Ziel:** Fabrikbetreiber sollen alle ihre Maschinen – unabhängig vom Protokoll – in einer einzigen Oberfläche überwachen können.

---

## 2. Zielgruppe

| Zielgruppe | Beschreibung |
|---|---|
| Kleine Produktionsbetriebe | 5–50 Maschinen, kein IT-Team |
| Mittlere Fertigungsunternehmen | 50–200 Maschinen, 1–2 IT-Mitarbeiter |
| Systemintegratoren | Bauen Lösungen für Fabrikbetreiber |

---

## 3. Funktionale Anforderungen

### 3.1 Protokoll-Unterstützung
- [ ] **FA-01** – System liest Daten über Modbus TCP
- [ ] **FA-02** – System liest Daten über Modbus RTU (seriell)
- [ ] **FA-03** – System liest Daten über OPC-UA
- [ ] **FA-04** – System sendet Daten über MQTT weiter
- [ ] **FA-05** – Mehrere Maschinen gleichzeitig überwachen

### 3.2 Datenspeicherung
- [ ] **FA-06** – Alle Maschinendaten werden mit Zeitstempel gespeichert
- [ ] **FA-07** – Historische Daten mind. 30 Tage abrufbar
- [ ] **FA-08** – Datenverlust bei Verbindungsabbruch wird verhindert

### 3.3 REST API
- [ ] **FA-09** – GET /machines liefert alle registrierten Maschinen
- [ ] **FA-10** – GET /machines/{id} liefert aktuellen Maschinenstatus
- [ ] **FA-11** – GET /machines/{id}/history liefert historische Daten
- [ ] **FA-12** – POST /alerts erstellt einen neuen Alarm
- [ ] **FA-13** – API ist vollständig mit Swagger dokumentiert

### 3.4 Dashboard
- [ ] **FA-14** – Echtzeit-Graphen für Temperatur, RPM, Druck
- [ ] **FA-15** – Alarm-Anzeige bei Überschreitung von Schwellenwerten
- [ ] **FA-16** – Dashboard funktioniert im Browser (PC & Handy)
- [ ] **FA-17** – Maschinenstatus: Grün (OK) / Gelb (Warnung) / Rot (Fehler)

### 3.5 Alarme
- [ ] **FA-18** – Benutzer kann Schwellenwerte pro Maschine definieren
- [ ] **FA-19** – System sendet Alarm wenn Schwellenwert überschritten
- [ ] **FA-20** – Alarme werden im Dashboard angezeigt

---

## 4. Nicht-funktionale Anforderungen

| ID | Anforderung | Messgröße |
|---|---|---|
| NFA-01 | Latenz von Maschine bis Dashboard | < 500ms |
| NFA-02 | Systemverfügbarkeit | > 99% |
| NFA-03 | Läuft auf Raspberry Pi 4 | RAM < 512MB |
| NFA-04 | Deployment via Docker | docker-compose up |
| NFA-05 | Unterstützte Betriebssysteme | Linux, Windows |
| NFA-06 | Kostenloser Tech-Stack | 0€ Lizenzkosten |
| NFA-07 | Code-Testabdeckung | > 80% |

---

## 5. Abgrenzung (was das System NICHT macht)

- ❌ Kein direktes Steuern von Maschinen (nur lesen)
- ❌ Keine Unterstützung von Profibus in Phase 1
- ❌ Keine mobile App (nur Web-Dashboard)
- ❌ Keine Cloud-Anbindung in Phase 1 (nur lokal)

---

## 6. Technologie-Stack

| Bereich | Technologie | Begründung |
|---|---|---|
| Gateway | Python + pymodbus | Beste Modbus-Bibliothek |
| OPC-UA | Python + asyncua | Open Source, aktiv gepflegt |
| Message Broker | Mosquitto (MQTT) | Kostenlos, weit verbreitet |
| Backend API | FastAPI (Python) | Schnell, automatische Swagger Docs |
| Datenbank | TimescaleDB | Optimiert für Zeitreihendaten |
| Frontend | React + Recharts | Modernes Dashboard |
| Container | Docker + Compose | Einfaches Deployment |

---

## 7. Änderungshistorie

| Datum | Version | Änderung |
|---|---|---|
| Mai 2026 | 1.0 | Erstversion erstellt |
