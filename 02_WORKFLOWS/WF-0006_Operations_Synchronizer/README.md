# WF-0006 – Operations Synchronizer

## Status

**Stable**

## Version

**1.0.0**

## Typ

Synchronization Workflow

---

# Zweck

WF-0006 synchronisiert Operations-Objekte zwischen dem JaMoKo OS und dem JaMoKo Operations Dashboard in Trello.

Der Workflow stellt sicher, dass alle relevanten Arbeitsobjekte aktuell, eindeutig und nachvollziehbar im Operations Dashboard abgebildet werden.

---

# Ziele

Version 1.0.0 unterstützt folgende Funktionen:

- Laden der Workflow-Konfiguration
- Laden definierter Operations-Objekte
- Laden vorhandener Trello-Karten
- Aufbau eines Operations Index
- Aufbau eines Trello Index
- Vergleich über die `objectId`
- Erkennung neuer Objekte
- Erkennung geänderter Objekte
- Erkennung unveränderter Objekte
- Erkennung mehrfacher Zuordnungen
- Erstellen fehlender Trello-Karten
- Aktualisieren bestehender Trello-Karten
- Logging der Änderungen
- Erstellung einer Zusammenfassung des Synchronisationslaufs

---

# Datenquelle

Version **1.0.0** verwendet kontrollierte Operations-Daten.

Die direkte Anbindung an das JaMoKo OS ist Bestandteil der Version **1.1**.

---

# Zielsystem

**JaMoKo Operations Dashboard (Trello)**

---

# Architektur

Der Workflow folgt dem JaMoKo Workflow Pattern.

```text
Configuration
        │
        ▼
Load
        │
        ▼
Index
        │
        ▼
Compare
        │
        ▼
Create / Update
        │
        ▼
Logging
        │
        ▼
Summary
```

Eine detaillierte Beschreibung befindet sich in **ARCHITECTURE.md**.

---

# Dokumentation

Dieses Workflow-Projekt besteht aus folgenden Dokumenten:

- README.md
- ARCHITECTURE.md
- FLOW.md
- SPECIFICATION.md
- TESTS.md
- CHANGELOG.md
- KNOWN_ISSUES.md

---

# Abhängigkeiten

- n8n
- Trello API
- JaMoKo Operations Dashboard

---

# Roadmap

## Version 1.1

Geplant sind unter anderem:

- direkte Anbindung an das JaMoKo OS
- zentrale Verwaltung der Trello-Board-ID
- Validation Layer
- erweiterte Fehlerbehandlung
- Workflow-Architekturstandard (`STD-0003`)

---

# Release

**Version:** 1.0.0

**Datum:** 2026-08-01

**Status:** Stable

WF-0006 ist der erste produktive Referenz-Workflow des **JaMoKo Automation OS** und bildet die Grundlage für zukünftige Workflow-Entwicklungen.