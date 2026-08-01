# WF-0006 – Specification

## Version

1.0.0

## Status

Stable

---

# Ziel

WF-0006 synchronisiert Operations-Objekte zwischen dem JaMoKo OS und dem Trello Operations Dashboard.

---

# Fachliche Anforderungen

Der Workflow muss:

- neue Operations-Objekte erkennen
- bestehende Trello-Karten erkennen
- Änderungen feststellen
- fehlende Karten erzeugen
- bestehende Karten aktualisieren
- doppelte objectIds erkennen
- doppelte Trello-Karten erkennen
- einen Synchronisationsbericht erzeugen

---

# Eingabedaten

Operations-Objekte besitzen mindestens:

- objectId
- objectType
- title
- status
- description
- source
- priority

---

# Ausgabe

Für jedes Objekt entsteht genau eines der Ergebnisse:

- create
- update
- none
- error

---

# Workflow-Phasen

1. Configuration
2. Load Operations
3. Build Operations Index
4. Load Trello
5. Build Trello Index
6. Compare
7. Create
8. Update
9. Logging
10. Summary

---

# Validierung

Der Workflow prüft:

- objectId vorhanden
- objectId eindeutig
- Trello-Karte eindeutig
- Vergleich möglich

---

# Fehlerfälle

- keine Operations-Objekte
- doppelte objectId
- mehrere Trello-Karten mit gleicher objectId
- API-Fehler
- ungültige Daten

---

# Nicht Bestandteil von Version 1.0

- direkte JaMoKo-OS-Anbindung
- Validation Layer
- Board-ID aus zentraler Konfiguration
- Recovery-Mechanismen

---

# Referenzen

- README.md
- FLOW.md
- ARCHITECTURE.md
- TESTS.md
- CHANGELOG.md
- KNOWN_ISSUES.md