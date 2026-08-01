# WF-0009 – Tests

## Version

0.1.0

---

## Ziel

Sicherstellen, dass WF-0009 reproduzierbare Reparaturvorschläge erzeugt.

---

## Test 001

Repair Report laden

Erwartung:

- Repair Report wird erfolgreich eingelesen

Status:

PASS

---

## Test 002

Repair Items splitten

Erwartung:

- Jeder Reparaturfall wird einzeln verarbeitet

Status:

PASS

---

## Test 003

GitHub Object Download

Erwartung:

- Objektdatei wird vollständig geladen

Status:

PASS

---

## Test 004

Markdown Decode

Erwartung:

- Base64 wird korrekt dekodiert

Status:

PASS

---

## Test 005

Object Parser

Erwartung:

- ID
- Typ
- Name
- Status
- Beschreibung

werden korrekt erkannt.

Status:

PASS

---

## Test 006

Repair Proposal

Erwartung:

Für fehlendes Feld

status

wird ein gültiger Vorschlag erzeugt.

Status:

PASS

---

## Test 007

Proposal Validation

Erwartung:

Ungültige Vorschläge werden erkannt.

Status:

PASS

---

## Test 008

Merge

Erwartung:

Alle Vorschläge werden korrekt zusammengeführt.

Status:

PASS

---

## Test 009

Repair Summary

Erwartung:

Summary enthält:

- processedObjects
- validProposals
- rejectedProposals
- errors

Status:

PASS

---

## Regression

Nach jeder Änderung an:

- Parser
- Validation
- Proposal Builder

werden alle Tests erneut ausgeführt.

---

## Freigabekriterium

WF-0009 darf erst veröffentlicht werden, wenn alle Tests erfolgreich sind.