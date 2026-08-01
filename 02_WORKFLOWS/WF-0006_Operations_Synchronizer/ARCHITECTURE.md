# WF-0006 – Architecture

## Version

1.0.0

## Zweck

WF-0006 synchronisiert definierte Operations-Objekte aus dem JaMoKo-Kontext mit dem Trello Operations Dashboard.

## Architekturprinzip

Der Workflow trennt Datenquelle, Indexierung, Vergleich und Aktion klar voneinander.

## Ablauf

### 1. Configuration

Lädt die zentrale Workflow-Konfiguration:

- Workflow-ID
- Version
- Trello-Board
- Status-Mapping
- Synchronisationsregeln

### 2. Load Operations

Lädt die zu synchronisierenden Operations-Objekte.

Version 1.0.0 verwendet hierfür kontrollierte, statische Daten.

### 3. Build Operations Index

Erzeugt einen Index nach `objectId`.

Beispiel:

```text
CUST-0001 → Operations-Objekt
PROJ-0001 → Operations-Objekt
WF-0006   → Operations-Objekt