# WF-0010 – Object Auditor

## Status

draft

## Version

0.1.0

## Typ

Audit Workflow

## Pipeline-Rolle

Auditor

---

## Zweck

WF-0010 prüft die strukturelle Qualität und Konsistenz des JaMoKo OS.

Der Workflow analysiert Objektakten und Registereinträge, erkennt Abweichungen und erzeugt einen nachvollziehbaren Audit Report.

WF-0010 verändert keine Dateien.

---

## Ziel

Der Objektbestand des JaMoKo OS soll regelmäßig auf Vollständigkeit, Eindeutigkeit und Konsistenz geprüft werden.

---

## Erste Ausbaustufe

Version 0.1.0 prüft:

- doppelte Objekt-IDs
- ungültige Statuswerte
- Objektakten ohne Registereintrag
- Registereinträge ohne vorhandene Objektakte

---

## Eingang

WF-0010 verarbeitet Daten aus:

- WF-0008 – Object Loader
- `00_Registry/registry.md`
- `00_Registry/status_registry.md`

---

## Ausgabe

Der Workflow erzeugt einen Audit Report mit:

- Anzahl geprüfter Objekte
- Anzahl geprüfter Registereinträge
- gefundene Fehler
- Warnungen
- betroffene Objekt-IDs
- betroffene Dateipfade
- Audit-Status
- Workflow-Version
- Zeitstempel

---

## Audit-Ergebnis

Mögliche Statuswerte:

- `passed`
- `warning`
- `failed`

---

## Sicherheitsgrenze

WF-0010:

- liest Daten
- vergleicht Daten
- bewertet Abweichungen
- erzeugt Berichte

WF-0010:

- verändert keine Objektakte
- verändert kein Register
- erstellt keinen Commit
- schreibt nicht nach GitHub
- genehmigt keine Reparatur automatisch

---

## Position in der Pipeline

```text
WF-0008 – Object Loader
        ↓
WF-0009 – Object Repair Engine
        ↓
WF-0010 – Object Auditor
        ↓
Manual Review
        ↓
WF-0011 – GitHub Writer