# WF-0008 – Object Loader

## Status

**Draft**

## Version

**0.1.0**

## Typ

Data Integration Workflow

---

# Zweck

WF-0008 liest definierte Objekte aus dem JaMoKo OS ein und überführt sie in ein einheitliches Operations-Format.

Der Workflow bildet die Datenquelle für nachgelagerte Automationen, insbesondere für:

- WF-0006 – Operations Synchronizer
- Trello-Synchronisation
- Dashboards
- weitere JaMoKo-Automationen

---

# Ziel

Objekte aus dem JaMoKo OS sollen automatisiert eingelesen und in einer standardisierten Struktur bereitgestellt werden.

Dadurch entfallen statische Testdaten in nachgelagerten Workflows.

---

# Eingabe

Objektakten aus dem JaMoKo OS, insbesondere:

```text
01_Objects/**/object.md