# WF-0011 – Changelog

Alle wesentlichen Änderungen am Workflow GitHub Writer werden in diesem Dokument festgehalten.

---

## [0.1.0] – 2026-08-03

### Status

released

### Hinzugefügt

- Workflow-Akte für WF-0011 angelegt.
- Simulationsmodus als einziger zulässiger Betriebsmodus definiert.
- Eingangsschema und Pflichtfelder dokumentiert.
- Prüfung der manuellen Freigabe vorgesehen.
- Prüfung des Auditstatus vorgesehen.
- Validierung von Objekt-ID, Zielpfad und Zielfeld definiert.
- `sourceSha` als verpflichtender Ausgangsbezug aufgenommen.
- Erstellung eines Change Plans beschrieben.
- Patch-Simulation ohne Schreibwirkung definiert.
- Einheitliche Erfolgs- und Ablehnungsergebnisse festgelegt.
- Zehn strukturierte Fehlercodes dokumentiert.
- 17 Testfälle einschließlich Sicherheitsprüfungen erstellt.
- Schreibschutzgarantie festgelegt:
  - keine Dateiänderung,
  - kein Branch,
  - kein Commit,
  - kein Push,
  - kein Pull Request,
  - keine schreibende GitHub-API.

### Dokumentation

Folgende Dokumente wurden für Version 0.1.0 erstellt:

- `README.md`
- `SPECIFICATION.md`
- `ARCHITECTURE.md`
- `FLOW.md`
- `TESTS.md`
- `CHANGELOG.md`
- `KNOWN_ISSUES.md`

### Abschlussstand

- n8n-Workflow vollständig aufgebaut.
- Alle 17 Testfälle bestanden.
- Testergebnisse dokumentiert.
- Workflow-Export geprüft und archiviert.
- Gesamtansicht und Referenzlauf als Screenshots archiviert.
- Version 0.1.0 bleibt eine reine, deaktivierte Simulation.

### Sicherheitsentscheidung

Version 0.1.0 bleibt eine reine Simulation. Schreibende Funktionen dürfen erst nach erfolgreichem Abschluss aller Tests und einer gesonderten dokumentierten Freigabe entwickelt werden.