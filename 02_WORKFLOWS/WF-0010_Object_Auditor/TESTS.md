# WF-0010 – Tests

## Version

0.1.0

## Status

draft

---

# Ziel

Sicherstellen, dass WF-0010 den Objektbestand des JaMoKo OS reproduzierbar und vollständig prüft.

Alle Audit-Regeln müssen nachvollziehbare Ergebnisse liefern.

---

# Test 001

## Audit Configuration

Erwartung:

Die Audit-Konfiguration wird vollständig geladen.

Prüfen:

- Workflow-ID
- Version
- Audit-Regeln
- Statuswerte
- Schweregrade

Status:

PASS

---

# Test 002

## Object Data

Erwartung:

Alle Objekte werden erfolgreich geladen.

Prüfen:

- Anzahl Objekte
- objectId
- objectType
- status
- path

Status:

PASS

---

# Test 003

## Registry Data

Erwartung:

Alle benötigten Register werden geladen.

Prüfen:

- Registry
- ID Registry
- Status Registry
- Relationship Registry

Status:

PASS

---

# Test 004

## Identity Audit

Erwartung:

Der Workflow erkennt:

- doppelte IDs
- fehlende IDs
- ungültige ID-Formate

Status:

PASS

---

# Test 005

## Status Audit

Erwartung:

Der Workflow erkennt:

- fehlenden Status
- ungültigen Status
- unbekannten Status

Status:

PASS

---

# Test 006

## Registry Audit

Erwartung:

Der Workflow erkennt:

- Objekt ohne Registereintrag
- Registereintrag ohne Objekt
- doppelte Registereinträge

Status:

PASS

---

# Test 007

## Relationship Audit

Erwartung:

Der Workflow erkennt:

- fehlende Zielobjekte
- fehlende Ziel-ID
- ungültige Beziehungstypen

Status:

PASS

---

# Test 008

## Merge Audit Findings

Erwartung:

Alle Audit-Ergebnisse werden vollständig zusammengeführt.

Status:

PASS

---

# Test 009

## Audit Status

Erwartung:

Der Gesamtstatus wird korrekt berechnet.

Mögliche Werte:

- passed
- warning
- failed

Status:

PASS

---

# Test 010

## Audit Summary

Erwartung:

Die Summary enthält:

- Workflow-ID
- Workflow-Version
- geprüfte Objekte
- geprüfte Register
- Anzahl Infos
- Anzahl Warnungen
- Anzahl Fehler
- Audit Status
- Zeitstempel

Status:

PASS

---

# Regressionstests

Nach Änderungen an:

- Audit-Regeln
- Parser
- Registry-Struktur
- Statusprüfung
- Relationship Audit

werden sämtliche Tests erneut durchgeführt.

---

# Freigabekriterium

WF-0010 darf nur veröffentlicht werden, wenn:

- alle Testfälle erfolgreich sind
- keine technischen Fehler auftreten
- die Audit Summary vollständig erzeugt wird
- der Workflow reproduzierbare Ergebnisse liefert