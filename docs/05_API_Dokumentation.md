# 📡 API Dokumentation – ProtocolBridge
**Version:** 0.3.0
**Datum:** Juni 2026
**Autor:** itgnf
**Base URL:** `http://localhost:8000`
**Swagger UI:** `http://localhost:8000/docs`

---

## Inhaltsverzeichnis

1. Übersicht
2. Authentifizierung
3. Endpoints – Info
4. Endpoints – Maschinen
5. Endpoints – Alarme
6. WebSocket
7. Fehlercodes
8. Beispiele

---

## 1. Übersicht

Die ProtocolBridge REST API stellt Maschinendaten aus der TimescaleDB bereit.
Daten fließen über folgende Pipeline in die API:

```
Modbus/OPC-UA → Gateway → MQTT → Subscriber → TimescaleDB → API → Dashboard
```

Alle Antworten sind im JSON Format. Zeitstempel sind im ISO 8601 Format (UTC).

---

## 2. Authentifizierung

In Version 0.3.0 ist keine Authentifizierung erforderlich.
Alle Endpoints sind öffentlich zugänglich.

---

## 3. Endpoints – Info

### GET /

Gibt allgemeine Informationen über die API zurück.

**Request:**
```
GET http://localhost:8000/
```

**Response 200:**
```json
{
  "name"   : "ProtocolBridge API",
  "version": "0.3.0",
  "docs"   : "/docs",
  "ws"     : "/ws/machines"
}
```

---

### GET /health

Prüft ob die API und die Datenbank erreichbar sind.

**Request:**
```
GET http://localhost:8000/health
```

**Response 200 – Alles OK:**
```json
{
  "status"  : "ok",
  "uptime_s": 3600,
  "started" : "2026-06-09 10:00:00",
  "services": {
    "api"     : "ok",
    "database": "ok"
  }
}
```

**Response 200 – DB nicht verbunden:**
```json
{
  "status"  : "ok",
  "uptime_s": 10,
  "started" : "2026-06-09 10:00:00",
  "services": {
    "api"     : "ok",
    "database": "nicht verbunden"
  }
}
```

---

## 4. Endpoints – Maschinen

### GET /machines

Gibt alle bekannten Maschinen mit ihrem letzten Messwert zurück.

**Request:**
```
GET http://localhost:8000/machines
```

**Response 200:**
```json
{
  "machines": [
    {
      "machine_id" : "CNC-001",
      "status"     : "RUNNING",
      "temperature": 78.3,
      "rpm"        : 2400.0,
      "pressure"   : 4.2,
      "runtime"    : null,
      "last_seen"  : "2026-06-09T12:00:00+00:00"
    }
  ],
  "total": 1
}
```

**Statuswerte:**

| Wert | Bedeutung |
|---|---|
| `RUNNING` | Maschine läuft normal |
| `WARNING` | Schwellenwert überschritten (status_code = 2) |

---

### GET /machines/{machine_id}

Gibt den aktuellen Status einer einzelnen Maschine zurück.

**Request:**
```
GET http://localhost:8000/machines/CNC-001
```

**Response 200:**
```json
{
  "machine_id" : "CNC-001",
  "status"     : "RUNNING",
  "temperature": 78.3,
  "rpm"        : 2400.0,
  "pressure"   : 4.2,
  "runtime"    : null,
  "last_seen"  : "2026-06-09T12:00:00+00:00",
  "alert"      : null
}
```

**Response 404 – Maschine nicht gefunden:**
```json
{
  "detail": "Maschine 'XYZ-999' nicht gefunden"
}
```

---

### GET /machines/{machine_id}/history

Gibt den Datenverlauf einer Maschine zurück.

**Request:**
```
GET http://localhost:8000/machines/CNC-001/history?minuten=30&limit=100
```

**Query Parameter:**

| Parameter | Typ | Standard | Beschreibung |
|---|---|---|---|
| `minuten` | integer | 30 | Verlauf der letzten N Minuten (1–1440) |
| `limit` | integer | 100 | Maximale Anzahl Einträge (1–1000) |

**Response 200:**
```json
{
  "machine_id": "CNC-001",
  "minuten"   : 30,
  "eintraege" : 15,
  "history"   : [
    {
      "time"       : "2026-06-09T12:00:00+00:00",
      "temperature": 78.3,
      "rpm"        : 2400.0,
      "pressure"   : 4.2,
      "status_code": 1,
      "runtime"    : null
    },
    {
      "time"       : "2026-06-09T11:58:00+00:00",
      "temperature": 82.1,
      "rpm"        : 2350.0,
      "pressure"   : 4.5,
      "status_code": 1,
      "runtime"    : null
    }
  ]
}
```

**Response 404 – Keine Daten im Zeitraum:**
```json
{
  "detail": "Keine Daten fuer 'CNC-001' in den letzten 30 Minuten"
}
```

---

## 5. Endpoints – Alarme

### GET /alerts

Gibt alle konfigurierten Alarm-Schwellenwerte zurück.

**Request:**
```
GET http://localhost:8000/alerts
```

**Response 200:**
```json
{
  "alerts": [
    {
      "machine_id"     : "CNC-001",
      "temperature_max": 90.0,
      "rpm_min"        : 500.0,
      "rpm_max"        : 3500.0,
      "pressure_max"   : 9.0
    }
  ],
  "total": 1
}
```

**Response 200 – Keine Alarme konfiguriert:**
```json
{
  "alerts": [],
  "total" : 0
}
```

---

### POST /alerts

Konfiguriert Alarm-Schwellenwerte für eine Maschine.
Wird ein Alarm für dieselbe Maschine erneut gesetzt, wird er überschrieben.

**Request:**
```
POST http://localhost:8000/alerts
Content-Type: application/json
```

**Request Body:**
```json
{
  "machine_id"     : "CNC-001",
  "temperature_max": 90.0,
  "rpm_min"        : 500.0,
  "rpm_max"        : 3500.0,
  "pressure_max"   : 9.0
}
```

**Body Parameter:**

| Parameter | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|
| `machine_id` | string | ja | – | ID der Maschine |
| `temperature_max` | float | nein | 90.0 | Max. Temperatur in °C |
| `rpm_min` | float | nein | 500.0 | Min. Drehzahl in RPM |
| `rpm_max` | float | nein | 3500.0 | Max. Drehzahl in RPM |
| `pressure_max` | float | nein | 9.0 | Max. Druck in bar |

**Response 200:**
```json
{
  "message"   : "Alarm fuer CNC-001 konfiguriert",
  "machine_id": "CNC-001",
  "config"    : {
    "machine_id"     : "CNC-001",
    "temperature_max": 90.0,
    "rpm_min"        : 500.0,
    "rpm_max"        : 3500.0,
    "pressure_max"   : 9.0
  }
}
```

**Response 422 – Validierungsfehler:**
```json
{
  "detail": [
    {
      "loc"  : ["body", "machine_id"],
      "msg"  : "field required",
      "type" : "value_error.missing"
    }
  ]
}
```

---

## 6. WebSocket

### WS /ws/machines

Sendet alle 2 Sekunden den aktuellen Status aller Maschinen.

**Verbindung:**
```javascript
const ws = new WebSocket("ws://localhost:8000/ws/machines");
ws.onmessage = (event) => {
  const daten = JSON.parse(event.data);
  console.log(daten);
};
```

**Empfangene Daten (alle 2 Sekunden):**
```json
[
  {
    "machine_id" : "CNC-001",
    "temperature": 78.3,
    "rpm"        : 2400.0,
    "pressure"   : 4.2,
    "status_code": 1,
    "runtime"    : null,
    "timestamp"  : 1749467200.0
  }
]
```

**Verbindung trennen:**
```javascript
ws.close();
```

---

## 7. Fehlercodes

| HTTP Code | Bedeutung | Ursache |
|---|---|---|
| `200` | OK | Anfrage erfolgreich |
| `404` | Not Found | Maschine nicht gefunden oder keine Daten im Zeitraum |
| `422` | Unprocessable Entity | Ungültige Request-Daten (Validierungsfehler) |
| `500` | Internal Server Error | Datenbankfehler oder unerwarteter Fehler |

---

## 8. Beispiele

### Python – Alle Maschinen abrufen

```python
import requests

r = requests.get("http://localhost:8000/machines")
data = r.json()

for maschine in data["machines"]:
    print(f"{maschine['machine_id']}: {maschine['temperature']}°C")
```

### Python – Alarm konfigurieren

```python
import requests

payload = {
    "machine_id"     : "CNC-001",
    "temperature_max": 85.0,
    "pressure_max"   : 8.0,
}
r = requests.post("http://localhost:8000/alerts", json=payload)
print(r.json()["message"])
```

### JavaScript – WebSocket verbinden

```javascript
const ws = new WebSocket("ws://localhost:8000/ws/machines");

ws.onopen = () => console.log("Verbunden");

ws.onmessage = (event) => {
  const maschinen = JSON.parse(event.data);
  maschinen.forEach(m => {
    console.log(`${m.machine_id}: ${m.temperature}°C`);
  });
};

ws.onclose = () => console.log("Getrennt");
```

---

## 9. Änderungshistorie

| Datum | Version | Änderung |
|---|---|---|
| Juni 2026 | 0.3.0 | Erstversion – alle Endpoints dokumentiert |
