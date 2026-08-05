# WF-0012 – GitHub Reader

## Changelog

Alle wesentlichen Änderungen an WF-0012 werden in dieser Datei chronologisch dokumentiert.

Das Format orientiert sich an **Keep a Changelog** und verwendet semantische Versionierung.

---

## Versionsstatus

| Version | Datum | Status |
|---|---|---|
| `v0.1.0` | 2026-08-05 | `released` |

---

## [Unreleased]

### Geplant

- Bereinigten n8n-Workflow exportieren
- Workflow-Export unter `exports/` ablegen
- Bereinigte Testnachweise unter `screenshots/` ablegen
- GitHub-Credential-Berechtigungen dokumentieren

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
- folgende Dateidaten werden übernommen:
  - Dateiname
  - Dateipfad
  - Datei-SHA
  - Encoding
  - dekodierter Inhalt
  - Dateigröße
- Base64-Dekodierung des GitHub-Dateiinhalts umgesetzt
- strikte Prüfung der Base64-Struktur ergänzt
- Validierung des zurückgegebenen Dateipfads und Datei-SHAs vorgesehen
- folgende normalisierte Fehlercodes implementiert:
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
- GitHub-Fehlerstatus `401`, `403`, `404` und sonstige API-Fehler werden eindeutig normalisiert

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
- nicht vorhandene Datei normalisieren
- ungültige GitHub-Antwort ablehnen
- fehlerhafte Base64-Daten ablehnen
- Authentifizierungsfehler normalisieren
- verweigerten GitHub-Zugriff normalisieren
- sonstigen GitHub-API-Fehler normalisieren

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