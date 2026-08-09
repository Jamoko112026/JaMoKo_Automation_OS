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
- Alle 17 Testfälle im historischen manuellen Testprotokoll als bestanden dokumentiert.
- Keine maschinenlesbaren Runtime-Protokolle innerhalb der v0.1.0-Dateien vorhanden.
- Workflow-Export geprüft und archiviert.
- Gesamtansicht und Referenzlauf als Screenshots archiviert.
- Version 0.1.0 bleibt eine reine, deaktivierte Simulation.

### Dokumentationskonsolidierung – 2026-08-09

- Allgemeine Dokumente auf den tatsächlich archivierten Drei-Knoten-Export
  v0.1.0 ausgerichtet.
- Monolithische Validierungs-, Change-Plan- und Patch-Preview-Logik im Knoten
  `03 – Validate and Simulate` klargestellt.
- Deaktivierten Betrieb ohne Credentials, GitHub-/HTTP-Zugriff oder
  Schreibwirkung festgehalten.
- Historische Testdokumentation von unabhängig reproduzierbaren
  Runtime-Nachweisen abgegrenzt.
- Fehlende allgemeine `.git`-Pfadsperre und fehlenden zentralen
  Fehler-Sanitizer als bekannte technische Einschränkungen dokumentiert.
- v0.2.0-Suffixdateien als nicht implementierten Entwurf abgegrenzt.

### Sicherheitsentscheidung

Version 0.1.0 bleibt eine reine Simulation. Schreibende Funktionen dürfen erst nach erfolgreichem Abschluss aller Tests und einer gesonderten dokumentierten Freigabe entwickelt werden.
