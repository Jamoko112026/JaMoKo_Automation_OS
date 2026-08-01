# WF-0008 – Known Issues

## Version

0.1.0

## Status

Draft

---

## Offene Punkte

- Die Anbindung an das JaMoKo OS erfolgt derzeit noch über eine definierte Testquelle.
- Das Parsen komplexer Objektfelder wird in Version 1.1 erweitert.
- Rekursive Objektbeziehungen werden noch nicht ausgewertet.
- Änderungen an Objektakten werden noch nicht historisiert.
- Fehlerbehandlung für nicht lesbare Dateien wird weiter ausgebaut.

---

## Technische Grenzen

Version 0.1.0 unterstützt zunächst ausschließlich standardisierte Objektakten (`object.md`).

Andere Dateitypen werden ignoriert.

---

## Nächste Ausbaustufe (Version 1.1)

- Direkte Anbindung an das JaMoKo OS
- Rekursives Einlesen aller Objektordner
- Erweiterte Validierung
- Unterstützung zusätzlicher Objektfelder
- Performance-Optimierung bei großen Objektmengen