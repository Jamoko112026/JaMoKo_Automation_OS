# WF-0010 – Specification

## Version

0.1.0

## Status

draft

---

## Zweck

WF-0010 analysiert den Objektbestand des JaMoKo OS und überprüft dessen strukturelle Qualität.

Der Workflow erkennt Inkonsistenzen, fehlende Referenzen und Verstöße gegen definierte Standards.

WF-0010 erzeugt ausschließlich einen Audit Report.

---

## Eingangsdaten

Der Workflow verarbeitet:

- Objektdaten aus WF-0008 – Object Loader
- Registerdaten des JaMoKo OS
- Statusregister
- Beziehungsregister

---

## Prüfbereich

Version 0.1.0 unterstützt folgende Prüfungen:

### Objekt-IDs

- doppelte IDs
- fehlende IDs
- ungültiges ID-Format

---

### Status

- fehlender Status
- ungültiger Statuswert
- nicht registrierter Status

---

### Register

- Objekt ohne Registereintrag
- Registereintrag ohne Objekt
- doppelte Registereinträge

---

### Beziehungen

- fehlende Zielobjekte
- ungültige Beziehungstypen

---

## Audit-Regeln

Jedes Objekt wird unabhängig geprüft.

Jede erkannte Abweichung erzeugt genau einen Audit-Eintrag.

Mehrere Fehler eines Objekts werden einzeln dokumentiert.

---

## Audit-Level

Zulässige Schweregrade:

- info
- warning
- error

---

## Audit-Eintrag

Jeder Audit-Eintrag enthält mindestens:

- objectId
- path
- category
- severity
- rule
- message

Beispiel:

```json
{
  "objectId": "FIN-0001",
  "path": "01_Objects/Finance/FIN-0001/object.md",
  "category": "status",
  "severity": "error",
  "rule": "missing_status",
  "message": "Das Pflichtfeld 'status' fehlt."
}
```

---

## Verarbeitung

WF-0010 führt folgende Schritte aus:

1. Auditdaten laden
2. Objekte einzeln prüfen
3. Register vergleichen
4. Beziehungen prüfen
5. Audit-Einträge erzeugen
6. Ergebnisse zusammenführen
7. Audit Summary erzeugen

---

## Audit Summary

Die Zusammenfassung enthält mindestens:

- geprüfte Objekte
- geprüfte Registereinträge
- Anzahl Informationen
- Anzahl Warnungen
- Anzahl Fehler
- Gesamtstatus

---

## Gesamtstatus

Zulässige Werte:

- passed
- warning
- failed

Regeln:

- keine Fehler → passed
- Warnungen vorhanden → warning
- mindestens ein Fehler → failed

---

## Sicherheitsregeln

Version 0.1.0:

- verändert keine Dateien
- verändert keine Register
- erzeugt keine Reparaturvorschläge
- schreibt nicht nach GitHub
- erstellt keine Commits

Der Workflow erzeugt ausschließlich Audit-Ergebnisse.

---

## Fehlerbehandlung

Ein technischer Fehler wird erzeugt wenn:

- Objektdaten fehlen
- Register nicht geladen werden können
- Pflichtinformationen unvollständig sind
- Prüfregeln nicht ausgeführt werden können

Technische Fehler werden getrennt von Audit-Ergebnissen dokumentiert.

---

## Ausgabe

WF-0010 liefert:

- Audit Summary
- Audit Report
- Fehlerliste
- Workflow-ID
- Workflow-Version
- Zeitstempel

---

## Nicht Bestandteil von Version 0.1.0

- automatische Reparaturen
- automatische Freigaben
- GitHub-Änderungen
- Commits
- Patch-Erzeugung
- Qualitätsbewertung durch KI

---

## Abhängigkeiten

- WF-0008 – Object Loader
- WF-0009 – Object Repair Engine
- JaMoKo OS
- GitHub API
- n8n
- STD-0003 – Workflow Architecture Standard