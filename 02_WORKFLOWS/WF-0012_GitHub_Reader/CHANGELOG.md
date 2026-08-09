# WF-0012 – GitHub Reader

## Changelog

Alle wesentlichen Änderungen an WF-0012 werden in dieser Datei chronologisch dokumentiert.

Das Format orientiert sich an **Keep a Changelog** und verwendet semantische Versionierung.

---

## Versionsstatus

| Version | Datum | Status |
|---|---|---|
| `v0.1.1` | 2026-08-08 | `released` |
| `v0.1.0` | 2026-08-05 | `released` |

---

## [Unreleased]

### Added

- Separaten technischen Export
  `exports/WF-0012_GitHub_Reader_v0.1.1.json` zur Umsetzung des veröffentlichten
  `v0.1.1`-Dokumentvertrags angelegt
- Vollständige Validierung der sieben garantierten Erfolgsfelder im finalen
  Vertrags-Guard ergänzt
- Sicheren zentralen Ausgabe- und Fehlerpfad ohne Ziel- oder Inhaltsdaten in
  Fehlerergebnissen ergänzt
- Zentralen Kardinalitäts-Guard direkt hinter den Triggerpfaden ergänzt, der
  null oder mehrere Eingangsobjekte mit dem bestehenden Fehlercode
  `INPUT_INVALID` auf genau eine minimale Fehlerausgabe reduziert

### Changed

- Zielwerte werden nicht mehr getrimmt; Rand-Leerzeichen führen zur Ablehnung
- Datei-SHA wird vor Erfolgsbildung exakt gegen `^[a-fA-F0-9]{40}$` geprüft
- Base64-dekodierter Inhalt wird durch eine bytegenaue UTF-8-Rundreise auf
  verlustfreie Dekodierung geprüft
- `file.name` und `file.size` aus der finalen Erfolgsausgabe entfernt
- Execute-Workflow-Trigger mit `alwaysOutputData` abgesichert, damit eine
  Null-Item-Übergabe den Kardinalitäts-Guard über einen leeren Platzhalter
  erreichen kann

### Testing

- Lokale Code- und Strukturtests `T-015` bis `T-030` für den v0.1.1-Export
  erfolgreich durchgeführt
- Operativer Import- und Laufzeitnachweis in n8n bleibt offen
- Laufzeitnachweis der Null-Item-Semantik von `alwaysOutputData` am
  Execute-Workflow-Trigger bleibt ausdrücklich offen

### Geplant

- Konformitätsnachweis für den `v0.1.1`-Dokumentvertrag mit bereinigten
  Testnachweisen unter `screenshots/` ablegen
- GitHub-Credential-Berechtigungen dokumentieren

Die technische Export-Konformität ist damit separat offen. Die Veröffentlichung
des `v0.1.1`-Dokumentvertrags behauptet bis zu diesem Nachweis keine vollständig
operative Umsetzung.

---

## [0.1.1] – 2026-08-08

### Added

- Verbindlichen Erfolgs- und Fehlerausgabevertrag von `v0.1.1` in
  `SPECIFICATION.md` und `ARCHITECTURE.md` vollständig veröffentlicht
- Garantierte Zielmetadaten sowie die eindeutigen Felder für Datei-SHA und
  dekodierten vollständigen Dateiinhalt festgelegt
- Fehlerklassen, Inhaltsintegrität und Vertragsgrenzen zu WF-0013 und WF-0011
  präzisiert

### Changed

- `file.sha` verbindlich auf `^[a-fA-F0-9]{40}$` festgelegt; ungültige oder leere
  Werte führen zu `READ_VALIDATION_FAILED`
- Base64-dekodierten Byteinhalt auf strikte und verlustfreie UTF-8-Dekodierung
  festgelegt; ungültige Bytefolgen oder verlustbehaftete Ersatzzeichen führen zu
  `CONTENT_DECODE_FAILED`
- JSON-Strukturbildung eindeutig von jeder Veränderung an Zielwerten, SHA oder
  Dateiinhalt abgegrenzt
- `file.name` und `file.size` ausdrücklich als nicht garantiert und nicht
  vertragsrelevant eingeordnet

---

## [0.1.0] – 2026-08-05

### Added

- Workflow-Akte `WF-0012_GitHub_Reader` angelegt
- GitHub Reader als ausführbaren n8n-Workflow umgesetzt
- Read-only-Betriebsmodus verbindlich implementiert
- Eingabeschema mit folgenden Pflichtfeldern umgesetzt:
  - `owner`
  - `repository`
  - `path`
  - `ref`
- Erfolgsausgabe mit Status `read` umgesetzt
- kontrollierte Fehlerausgabe mit Status `rejected` umgesetzt
- Branches, Tags und Commit-SHAs als GitHub-Referenzen vorgesehen
- statische Freigabe für folgende Kombination eingerichtet:
  - Owner: `Jamoko112026`
  - Repository: `JaMoKo_Automation_OS`
- Prüfung der konkreten Owner-Repository-Kombination implementiert
- relative Repository-Pfade als einzige zulässige Pfadform festgelegt
- Schutz gegen absolute Pfade und Pfadüberschreitungen umgesetzt
- GitHub-Dateiabruf als ausschließlich lesende Operation umgesetzt
- folgende vertragsrelevante Dateidaten werden übernommen:
  - Dateipfad
  - Datei-SHA
  - Encoding
  - dekodierter Inhalt
- Dateiname und Dateigröße können technisch vorkommen, sind aber weder garantiert
  noch vertragsrelevant
- Base64-Dekodierung des GitHub-Dateiinhalts umgesetzt
- strikte Prüfung der Base64-Struktur ergänzt
- Validierung des zurückgegebenen Dateipfads und Datei-SHAs vorgesehen
- folgende standardisierte Fehlercodes implementiert:
  - `INPUT_INVALID`
  - `REF_MISSING`
  - `OWNER_NOT_ALLOWED`
  - `REPOSITORY_NOT_ALLOWED`
  - `PATH_INVALID`
  - `AUTHENTICATION_FAILED`
  - `ACCESS_DENIED`
  - `FILE_NOT_FOUND`
  - `GITHUB_API_ERROR`
  - `CONTENT_DECODE_FAILED`
  - `READ_VALIDATION_FAILED`
- verbindliche Schreibschutzwerte umgesetzt:
  - `writeRequested: false`
  - `writeExecuted: false`
  - `commitCreated: false`
  - `pushExecuted: false`
- Write Protection Guard auf Erfolgs- und Fehlerpfaden umgesetzt
- automatische Ausführung von WF-0011 ausgeschlossen
- mögliche spätere Schnittstelle zu WF-0011 dokumentiert
- 14 verbindliche Testfälle dokumentiert und durchgeführt
- Nachweisstruktur für Screenshots und bereinigte n8n-Exporte angelegt
- Known-Issue-System mit Status- und Prioritätswerten eingeführt

### Changed

- Base64-Dekodierung um eine strikte Struktur- und Rückkodierungsprüfung erweitert
- Testumfang auf die tatsächlich implementierten Testfälle `T-001` bis `T-014` vereinheitlicht
- Workflow-Status nach erfolgreichem technischen Test von `draft` auf `testing` gesetzt

### Fixed

- Tolerantes Verhalten von `Buffer.from(..., "base64")` abgesichert
- Ungültige Base64-Inhalte werden jetzt zuverlässig mit `CONTENT_DECODE_FAILED` abgelehnt
- Verzeichnisantworten der GitHub API werden kontrolliert mit `READ_VALIDATION_FAILED` abgelehnt
- GitHub-Fehlerstatus `401`, `403`, `404` und sonstige API-Fehler werden eindeutig
  den dokumentierten Fehlercodes zugeordnet

### Testing

Alle definierten Testfälle `T-001` bis `T-014` wurden erfolgreich durchgeführt:

- gültige Datei lesen
- Pfadüberschreitung ablehnen
- nicht erlaubten Owner ablehnen
- nicht erlaubtes Repository ablehnen
- Schreibschutz nach erfolgreichem Lesen bestätigen
- Schreibschutz nach Ablehnung bestätigen
- fehlendes Pflichtfeld ablehnen
- leere Referenz ablehnen
- nicht vorhandene Datei kontrolliert als `FILE_NOT_FOUND` ablehnen
- ungültige GitHub-Antwort ablehnen
- fehlerhafte Base64-Daten ablehnen
- Authentifizierungsfehler kontrolliert ablehnen
- verweigerten GitHub-Zugriff kontrolliert ablehnen
- sonstigen GitHub-API-Fehler kontrolliert ablehnen

Testergebnis: `passed`

### Documentation

- `README.md` erstellt
- `SPECIFICATION.md` erstellt
- `ARCHITECTURE.md` erstellt
- `FLOW.md` erstellt
- `TESTS.md` erstellt und auf den geprüften Stand aktualisiert
- `KNOWN_ISSUES.md` erstellt
- `CHANGELOG.md` aktualisiert
- Verzeichnisse `exports/` und `screenshots/` angelegt

### Security

- Workflow technisch auf `read-only` begrenzt
- schreibende GitHub-Operationen ausgeschlossen
- Allowlist-Prüfung vor dem GitHub-Zugriff umgesetzt
- Eingabedaten grundsätzlich als nicht vertrauenswürdig behandelt
- absolute Pfade und Pfadüberschreitungen blockiert
- Credentials ausschließlich über den n8n Credential Store eingebunden
- externe Schreibschutzwerte werden nicht übernommen
- Write Protection Guard setzt alle Schreibschutzwerte intern auf `false`
- rohe GitHub- und n8n-Fehlerobjekte werden nicht ausgegeben
- Ausgabe von Token, Headern und Credential-Details untersagt
- automatische Weiterleitung an WF-0011 ausgeschlossen

### Known limitations

- GitHub-Credential-Berechtigungen müssen noch abschließend dokumentiert werden
- Verhalten bei sehr großen Dateien wurde noch nicht untersucht
- Binärdateien werden in `v0.1.0` nicht unterstützt
- Allowlist ist zunächst statisch
- weiterführende Validierung unterschiedlicher Referenzformate ist noch nicht vollständig umgesetzt
- automatische Übergabe an WF-0011 ist nicht freigegeben
- bereinigter n8n-Export und Screenshots müssen noch archiviert werden

---

## Änderungsregeln

Jede zukünftige Änderung wird zunächst unter `[Unreleased]` eingetragen.

Verwendete Änderungskategorien:

```text
Added
Changed
Deprecated
Removed
Fixed
Security
Documentation
Testing
Known limitations
```
