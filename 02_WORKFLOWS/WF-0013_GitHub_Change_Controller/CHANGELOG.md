# WF-0013 – Changelog

Alle wesentlichen Änderungen an `WF-0013 – GitHub Change Controller` werden in dieser Datei chronologisch dokumentiert.

Das Format orientiert sich an Keep a Changelog. Die Versionierung folgt Semantic Versioning.

---

## [Unreleased]

### Geplant

- Inhaltsgrößenlimit für `proposed_content` festlegen
- Commit-Nachrichtenregeln vervollständigen
- Request-ID-Format definieren
- Allowlist technisch implementieren
- Pfadnormalisierung und Traversal-Schutz implementieren
- Vertrag mit WF-0012 technisch prüfen
- Writer-Payload mit WF-0011 abgleichen
- State Comparator implementieren
- Decision Engine mit deterministischer Fehlerpriorität implementieren
- zentralen Output Sanitizer implementieren
- n8n-Workflow im Modus `prepare-only` aufbauen
- Sicherheits- und Funktionstests ausführen
- bereinigten n8n-Export prüfen und archivieren

---

## [0.1.0] – 2026-08-05

### Hinzugefügt

- Workflow-Akte für `WF-0013 – GitHub Change Controller` angelegt
- Betriebsmodus `prepare-only` festgelegt
- Eingabeschema für Änderungsaufträge definiert
- Pflichtfelder und Datentypen dokumentiert
- Ziel-Allowlist für Owner und Repository konzipiert
- Regeln für sichere relative Dateipfade definiert
- ausschließlichen Leseweg über WF-0012 festgelegt
- SHA-Prüfung gegen den aktuellen Reader-Stand definiert
- Erkennung unveränderter Inhalte festgelegt
- strikte Freigaberegel `approval === true` definiert
- Statusmodell mit `prepared` und `rejected` eingeführt
- Writer-Payload für WF-0011 konzipiert
- automatische Ausführung von WF-0011 ausgeschlossen
- direkten GitHub-Lese- und Schreibzugriff ausgeschlossen
- Sicherheitsinvarianten dokumentiert
- deterministische Fehlercodes und Fehlerpriorität festgelegt
- bereinigte Erfolgs- und Ablehnungsausgaben definiert
- vollständigen Ablauf in `FLOW.md` dokumentiert
- Architektur und Komponentengrenzen in `ARCHITECTURE.md` dokumentiert
- Spezifikation in `SPECIFICATION.md` dokumentiert
- Funktions-, Sicherheits- und Regressionstests in `TESTS.md` definiert
- offene Risiken und akzeptierte Einschränkungen in `KNOWN_ISSUES.md` dokumentiert

### Sicherheitsentscheidungen

- WF-0013 besitzt keine GitHub-Credentials
- WF-0013 liest GitHub nicht direkt
- WF-0013 schreibt GitHub nicht direkt
- WF-0013 führt WF-0011 nicht automatisch aus
- jeder Schreibauftrag erfordert einen aktuellen SHA-Vergleich
- ausschließlich der Boolean-Wert `true` gilt als Freigabe
- unbekannte oder sicherheitskritische Zusatzfelder werden nicht verarbeitet
- interne Fehlerdetails und Geheimnisse dürfen nicht ausgegeben werden
- `write_executed` bleibt in jedem Endzustand `false`

### Einschränkungen

- noch keine technische n8n-Implementierung
- noch keine ausgeführten Tests
- noch kein verbindliches Inhaltsgrößenlimit
- noch kein technischer Schemaabgleich mit WF-0011 und WF-0012
- noch kein freigegebener n8n-Export
- nur eine Textdatei pro Änderungsauftrag
- keine Binärdateien
- keine Branch-Erstellung
- keine Pull-Request-Erstellung
- keine automatische Konfliktauflösung
- keine automatische Writer-Ausführung

### Status

```text
draft