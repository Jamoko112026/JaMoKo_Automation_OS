# WF-0010 – Known Issues

## Version

0.1.0

## Status

draft

---

# Zweck

Dieses Dokument beschreibt bekannte Einschränkungen von WF-0010 sowie geplante Verbesserungen.

Die aufgeführten Punkte sind bewusst dokumentiert und stellen keine unbeabsichtigten Fehler dar.

---

# Bekannte Einschränkungen

## Audit Configuration

Version 0.1.0 verwendet eine statische Audit-Konfiguration.

Eine zentrale Konfigurationsverwaltung wird in einer späteren Version eingeführt.

---

## Object Loader

WF-0010 verarbeitet derzeit normalisierte Objektdaten aus WF-0008.

Eine direkte Verarbeitung beliebiger Datenquellen ist nicht Bestandteil von Version 0.1.0.

---

## Register

Der Workflow prüft ausschließlich bekannte Register.

Neue Register müssen künftig explizit in die Audit-Konfiguration aufgenommen werden.

---

## Relationship Audit

Beziehungen werden nur geprüft, wenn sie strukturiert vorliegen.

Freitextbeschreibungen werden nicht ausgewertet.

---

## Decision Audit

Verweise auf Entscheidungen (DEC-xxxx) werden noch nicht geprüft.

Geplant für Version 0.2.0.

---

## Standard Audit

Verweise auf Standards (STD-xxxx) werden noch nicht geprüft.

Geplant für Version 0.2.0.

---

## Repository Audit

Die Repository-Struktur wird derzeit nicht vollständig geprüft.

Version 0.1.0 konzentriert sich auf:

- Objektakten
- Register
- Beziehungen
- Statuswerte

---

## Health Score

Ein System Health Score wird noch nicht berechnet.

Geplant:

```text
JaMoKo OS Health

██████████████░░░░░░ 72 %

Objects:      42
Warnings:      3
Errors:        1
```

---

## Performance

Version 0.1.0 wurde für kleine bis mittlere Objektbestände entwickelt.

Eine Optimierung für mehrere tausend Objekte erfolgt später.

---

## Schreibzugriffe

WF-0010 besitzt bewusst keine Schreibrechte.

Der Workflow:

- verändert keine Dateien
- verändert keine Register
- erstellt keine Commits
- schreibt nicht nach GitHub

Diese Trennung ist Bestandteil der Architektur.

---

## KI-Unterstützung

Version 0.1.0 bewertet ausschließlich definierte Regeln.

Freie inhaltliche Bewertungen oder KI-basierte Qualitätsanalysen erfolgen noch nicht.

---

# Geplante Erweiterungen

## Version 0.2.0

- Decision Audit
- Standard Audit
- System Health Score
- zusätzliche Registry-Prüfungen
- erweiterte Relationship-Prüfung

---

## Version 0.3.0

- Repository Audit
- Strukturprüfung
- Dokumentationsprüfung
- Historienprüfung
- Performance-Optimierung

---

## Version 1.0.0

Vollständiger System-Auditor für das JaMoKo OS.

Der Workflow bewertet den gesamten Objektbestand reproduzierbar, erstellt einen vollständigen Audit Report und dient als zentrale Qualitätskontrolle des Systems.

---

# Bekannte Risiken

- unvollständige Register können Folgefehler erzeugen
- fehlerhafte Objektstrukturen beeinflussen mehrere Audit-Regeln
- neue Objekttypen benötigen zusätzliche Prüfregeln

Diese Risiken werden bewusst dokumentiert und bei zukünftigen Versionen schrittweise reduziert.