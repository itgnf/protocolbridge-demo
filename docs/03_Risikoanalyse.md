# ⚠️ Risikoanalyse – ProtocolBridge
**Version:** 1.0  
**Datum:** Mai 2026  
**Autor:** itgnf  
**Modell:** Spiralmodell – Risiken werden pro Spirale neu bewertet  
**Status:** In Bearbeitung

---

## 1. Bewertungsschema

**Wahrscheinlichkeit:**
- 🟢 Niedrig (1) – tritt selten auf
- 🟡 Mittel (2) – könnte auftreten
- 🔴 Hoch (3) – wird wahrscheinlich auftreten

**Auswirkung:**
- 🟢 Gering (1) – leicht behebbar
- 🟡 Mittel (2) – verzögert Projekt um 1–2 Wochen
- 🔴 Hoch (3) – blockiert das gesamte Projekt

**Risiko-Score = Wahrscheinlichkeit × Auswirkung**

---

## 2. Technische Risiken

### R-01 – Modbus Simulator funktioniert nicht wie erwartet
| Feld | Wert |
|---|---|
| Wahrscheinlichkeit | 🟡 Mittel (2) |
| Auswirkung | 🟡 Mittel (2) |
| **Risiko-Score** | **4 / 9** |
| Phase | Phase 2 – Simulatoren |

**Beschreibung:**  
pymodbus könnte Kompatibilitätsprobleme haben oder der Simulator verhält sich anders als eine echte Maschine.

**Gegenmaßnahme:**  
- Früh testen (Woche 1)
- Alternative: `diagslave` als fertiger Simulator
- Backup: öffentliche Test-Modbus-Server im Internet nutzen

---

### R-02 – OPC-UA Komplexität unterschätzt
| Feld | Wert |
|---|---|
| Wahrscheinlichkeit | 🔴 Hoch (3) |
| Auswirkung | 🟡 Mittel (2) |
| **Risiko-Score** | **6 / 9** |
| Phase | Phase 2 & 3 |

**Beschreibung:**  
OPC-UA ist deutlich komplexer als Modbus. Zertifikate, Namespaces und Sicherheitsmodi können den Einstieg erschweren.

**Gegenmaßnahme:**  
- Mit dem Prosys OPC-UA Simulator starten (hat einfache Defaults)
- Sicherheit erstmal deaktivieren (None-Mode) für Entwicklung
- OPC-UA erst in Phase 2 angehen, nicht Phase 1

---

### R-03 – Performance auf Raspberry Pi zu langsam
| Feld | Wert |
|---|---|
| Wahrscheinlichkeit | 🟡 Mittel (2) |
| Auswirkung | 🟢 Gering (1) |
| **Risiko-Score** | **2 / 9** |
| Phase | Phase 3 & 4 |

**Beschreibung:**  
Bei vielen gleichzeitigen Maschinen könnte der Raspberry Pi 4 mit der Last überfordert sein.

**Gegenmaßnahme:**  
- Polling-Interval konfigurierbar machen (Standard: 2 Sekunden)
- TimescaleDB Indizes korrekt setzen
- Zuerst auf PC entwickeln, später auf Pi testen

---

### R-04 – MQTT Datenverlust bei Verbindungsabbruch
| Feld | Wert |
|---|---|
| Wahrscheinlichkeit | 🟡 Mittel (2) |
| Auswirkung | 🔴 Hoch (3) |
| **Risiko-Score** | **6 / 9** |
| Phase | Phase 3 |

**Beschreibung:**  
Wenn der MQTT Broker kurz ausfällt, gehen Maschinendaten verloren.

**Gegenmaßnahme:**  
- MQTT QoS Level 1 oder 2 verwenden (garantierte Zustellung)
- Lokaler Puffer im Gateway (SQLite als Zwischenspeicher)
- Automatischer Reconnect im Gateway implementieren

---

### R-05 – Datenbankschema muss später geändert werden
| Feld | Wert |
|---|---|
| Wahrscheinlichkeit | 🔴 Hoch (3) |
| Auswirkung | 🟡 Mittel (2) |
| **Risiko-Score** | **6 / 9** |
| Phase | Phase 4 |

**Beschreibung:**  
Anforderungen an gespeicherte Daten ändern sich während der Entwicklung.

**Gegenmaßnahme:**  
- Datenbankmigrationen von Anfang an nutzen (Alembic)
- Flexibles JSON-Feld für Maschinenwerte (erweiterbar)
- Schema früh und gut dokumentieren (Issue #14)

---

## 3. Projektrisiken

### R-06 – Kein echter Pilotkunde in Phase 1
| Feld | Wert |
|---|---|
| Wahrscheinlichkeit | 🟡 Mittel (2) |
| Auswirkung | 🟡 Mittel (2) |
| **Risiko-Score** | **4 / 9** |

**Beschreibung:**  
Ohne echten Fabrik-Kunden als Feedback-Geber werden falsche Features gebaut.

**Gegenmaßnahme:**  
- Frühzeitig Kontakt zu lokalen Produktionsbetrieben aufnehmen
- Demo mit Simulatoren zeigen (kein echter Aufwand für Kunden)
- LinkedIn für B2B-Kontakte nutzen

---

## 4. Risiko-Matrix Übersicht

```
Auswirkung
   Hoch │       │ R-04  │
        │       │ R-02  │ R-05
  Mittel│       │ R-01  │
        │ R-03  │ R-06  │
  Gering│       │       │
        └───────┴───────┴──────
          Niedrig  Mittel  Hoch
                            Wahrscheinlichkeit

🟥 Kritisch (Score 6–9): R-02, R-04, R-05
🟧 Wichtig  (Score 3–5): R-01, R-06
🟩 Gering   (Score 1–2): R-03
```

---

## 5. Maßnahmen-Priorität

| Priorität | Risiko | Maßnahme | Bis wann |
|---|---|---|---|
| 🔴 1 | R-04 MQTT Datenverlust | QoS + Reconnect implementieren | Phase 3 |
| 🔴 2 | R-02 OPC-UA Komplexität | Prosys Simulator nutzen | Phase 2 |
| 🔴 3 | R-05 DB Schema | Alembic Migrationen einrichten | Phase 4 |
| 🟡 4 | R-01 Modbus Simulator | Früh testen in Woche 1 | Phase 2 |
| 🟡 5 | R-06 Kein Pilotkunde | LinkedIn Outreach starten | Phase 3 |

---

## 6. Änderungshistorie

| Datum | Version | Änderung |
|---|---|---|
| Mai 2026 | 1.0 | Erstversion – 6 Risiken identifiziert |
