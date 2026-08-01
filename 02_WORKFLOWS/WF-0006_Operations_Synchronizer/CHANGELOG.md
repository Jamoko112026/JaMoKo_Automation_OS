# WF-0006 – Changelog

## 1.0.0 – 2026-08-01

### Added

- Vollständige Workflow-Architektur aufgebaut.
- Zentrale Konfigurations-Node (`Load Configuration`) eingeführt.
- Operations-Objekte als zentrale Datenquelle definiert.
- Operations Index implementiert.
- Trello Index implementiert.
- Vergleich zwischen JaMoKo-Objekten und Trello-Karten implementiert.
- Automatische Erkennung neuer Objekte.
- Automatische Aktualisierung bestehender Karten.
- Status-Mapping zentral konfigurierbar.
- Create- und Update-Pipeline getrennt aufgebaut.
- Zusammenfassungs-Node (`Summary`) integriert.
- Logging für Create- und Update-Prozesse ergänzt.
- Projektdokumentation vervollständigt (`README`, `FLOW`, `SPECIFICATION`, `TESTS`, `ARCHITECTURE`, `KNOWN_ISSUES`).

### Changed

- Workflow von Version 0.1.0 (Konzept) zu Version 1.0.0 (erste stabile Referenzimplementierung) weiterentwickelt.
- Architektur klar in die Phasen **Configuration → Load → Index → Compare → Action → Summary** gegliedert.
- Konfiguration zentralisiert, um spätere Erweiterungen zu vereinfachen.

### Fixed

- Erkennung doppelter `objectId`-Einträge.
- Erkennung von Trello-Karten ohne gültige JaMoKo-Objekt-ID.
- Konsistente Übergabe der Konfigurationsdaten zwischen den Workflow-Nodes.

### Status

**Release 1.0.0 – Stable**

Der Workflow dient als Referenzimplementierung für zukünftige JaMoKo-Workflows.