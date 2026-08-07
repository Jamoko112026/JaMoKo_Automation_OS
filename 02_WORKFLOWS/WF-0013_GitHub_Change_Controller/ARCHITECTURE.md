# WF-0013 – GitHub Change Controller

Version: `v0.1.0`
Status: `draft`
Typ: `Controller`

---

## 1. Architekturziel

WF-0013 bildet die kontrollierende Schicht zwischen einem Änderungsauftrag und den spezialisierten GitHub-Workflows.

Der Workflow:

- validiert den Änderungsauftrag
- fordert den aktuellen Dateistand über WF-0012 an
- vergleicht Soll- und Ist-Zustand
- prüft SHA, Inhalt und Freigabe
- bereitet bei gültigem Ergebnis einen Schreibauftrag für WF-0011 vor
- führt selbst keinen Schreibvorgang aus

Der Betriebsmodus von `v0.1.0` lautet:

```text
prepare-only

```

Im Modus `prepare-only` darf WF-0013:

- Änderungsaufträge entgegennehmen
- Eingabedaten normalisieren und validieren
- freigegebene Ziele prüfen
- einen Leseauftrag für WF-0012 vorbereiten
- den von WF-0012 gelieferten Dateistand prüfen
- Soll- und Ist-Zustand vergleichen
- SHA-Konflikte und unveränderte Inhalte erkennen
- die ausdrückliche Freigabe prüfen
- einen kontrollierten Writer-Auftrag für WF-0011 erzeugen
- ein bereinigtes Kontrollergebnis ausgeben

WF-0013 darf in `v0.1.0` nicht:

- selbst Dateien verändern
- Git-Operationen ausführen
- direkt auf die GitHub API zugreifen
- Commits oder Pushes auslösen
- WF-0011 automatisch ausführen
- Konflikte automatisch überschreiben
- eine fehlende Freigabe ergänzen oder ableiten
- Geheimnisse oder Zugangsdaten ausgeben

---

## 2. Architekturprinzipien

Die Architektur folgt diesen verbindlichen Prinzipien:

1. **Trennung der Verantwortlichkeiten**
   WF-0012 liest, WF-0013 kontrolliert und WF-0011 verarbeitet vorbereitete Schreibaufträge.

2. **Kein implizites Schreiben**
   Ein erfolgreich vorbereiteter Writer-Auftrag darf keinen Schreibvorgang auslösen.

3. **Fail closed**
   Fehlende, unbekannte oder widersprüchliche Angaben führen zur Ablehnung.

4. **Explizite Freigabe**
   Nur der Boolean-Wert `true` gilt als erteilte Freigabe.

5. **Optimistische Nebenläufigkeitskontrolle**
   Der erwartete SHA muss mit dem aktuellen SHA übereinstimmen.

6. **Deterministische Entscheidungen**
   Derselbe Eingangszustand muss dasselbe Ergebnis erzeugen.

7. **Minimale Datenweitergabe**
   Nachgelagerte Komponenten erhalten nur die benötigten und validierten Felder.

8. **Bereinigte Ausgaben**
   Zugangsdaten, Header, Tokens und interne Fehlerdetails dürfen nicht ausgegeben werden.

9. **Nachvollziehbarkeit**
   Jede Entscheidung muss über Status, Fehlercode und `request_id` zuordenbar sein.

10. **Versionsgebundene Schnittstellen**
    Die Übergaben an WF-0012 und WF-0011 richten sich nach dokumentierten Verträgen.

---

## 3. Systemgrenze

WF-0013 liegt zwischen dem fachlichen Änderungsauftrag und den spezialisierten GitHub-Workflows.

```text
Änderungsauftrag
        |
        v
WF-0013 GitHub Change Controller
        |
        +--> Leseauftrag für WF-0012
        |            |
        |            v
        |    aktueller Dateistand
        |
        v
vorbereiteter Writer-Auftrag
        |
        v
WF-0011 GitHub Writer
```

WF-0011 wird in `v0.1.0` nicht automatisch ausgeführt.

Der vorbereitete Writer-Auftrag wird ausschließlich als kontrollierte Ausgabe bereitgestellt.

---

## 4. Komponentenübersicht

WF-0013 besteht logisch aus den folgenden Komponenten:

| Komponente | Aufgabe |
|---|---|
| Input Receiver | Nimmt genau einen Änderungsauftrag entgegen |
| Input Normalizer | Vereinheitlicht zulässige Eingangswerte |
| Schema Validator | Prüft Pflichtfelder, Datentypen und Struktur |
| Target Guard | Prüft Owner, Repository, Branch und Dateipfad |
| Reader Request Builder | Erstellt den Leseauftrag für WF-0012 |
| Reader Result Validator | Prüft die Antwort von WF-0012 |
| State Comparator | Vergleicht erwarteten und aktuellen Zustand |
| Approval Guard | Prüft die ausdrückliche Freigabe |
| Decision Engine | Ermittelt das deterministische Gesamtergebnis |
| Writer Payload Builder | Erstellt den Auftrag für WF-0011 |
| Output Sanitizer | Entfernt unzulässige oder sensible Ausgabedaten |
| Result Builder | Erzeugt die kanonische Controller-Ausgabe |

Jede Komponente besitzt genau eine fachliche Hauptverantwortung.

---

## 5. Eingangsmodell

WF-0013 verarbeitet genau einen strukturierten Änderungsauftrag.

Das fachliche Eingangsmodell besteht aus:

```text
execution
target
change
approval
request_id, optional
```

### 5.1 `execution`

`execution` beschreibt den gewünschten Betriebsmodus.

Verbindliche Felder:

```text
mode
```

Für `v0.1.0` ist ausschließlich zulässig:

```text
prepare-only
```

Andere Modi werden abgelehnt.

### 5.2 `target`

`target` beschreibt die zu prüfende GitHub-Datei.

Verbindliche Felder:

```text
owner
repository
branch
path
expected_sha
```

Die Werte müssen:

- als String vorliegen
- nach Entfernung äußerer Leerzeichen nicht leer sein
- den definierten Allowlist-Regeln entsprechen
- frei von Steuerzeichen sein
- ohne Pfad-Traversal auskommen

### 5.3 `change`

`change` beschreibt den gewünschten neuen Dateizustand.

Verbindliche Felder:

```text
proposed_content
commit_message
```

`proposed_content` muss ein String sein.

`commit_message` muss ein nicht leerer String sein und den Regeln von WF-0011 entsprechen.

### 5.4 `approval`

`approval` enthält die ausdrückliche Schreibfreigabe.

Verbindliches Feld:

```text
approved
```

Nur folgender Wert ist gültig:

```json
{
  "approved": true
}
```

Die Werte `"true"`, `1`, `"yes"` oder andere Ersatzwerte gelten nicht als Freigabe.

### 5.5 `request_id`

`request_id` dient der Zuordnung und Nachvollziehbarkeit.

Ist keine `request_id` vorhanden, darf WF-0013 eine kontrollierte interne Kennung erzeugen, sofern dies im Flow ausdrücklich implementiert und dokumentiert ist.

Eine erzeugte Kennung stellt keine fachliche Freigabe dar.

---

## 6. Validierungsschicht

Die Validierung erfolgt in einer festen Reihenfolge.

### 6.1 Strukturprüfung

Geprüft werden:

- genau ein Eingangsobjekt
- zulässige Hauptbereiche
- vorhandene Pflichtfelder
- korrekte Datentypen
- keine unerwarteten Nullwerte
- keine unzulässigen Mehrfachobjekte

### 6.2 Modusprüfung

`execution.mode` muss exakt `prepare-only` entsprechen.

Ein anderer oder fehlender Modus beendet die Verarbeitung.

### 6.3 Zielprüfung

Der Target Guard prüft:

- freigegebenen Owner
- freigegebenes Repository
- freigegebenen Branch
- zulässigen Dateipfad
- zulässige Dateiendung
- Pfadnormalisierung
- Traversal-Versuche
- absolute Pfade
- leere Pfadsegmente
- Steuerzeichen
- unerwartete URL-Bestandteile

### 6.4 Inhaltsprüfung

Geprüft werden:

- Datentyp von `proposed_content`
- konfiguriertes Größenlimit
- Datentyp und Länge der Commit-Nachricht
- unzulässige Steuerzeichen
- verbotene oder nicht unterstützte Änderungsarten

Binärdateien werden in `v0.1.0` nicht unterstützt.

### 6.5 Freigabeprüfung

Die Freigabe wird erst nach den strukturellen und fachlichen Prüfungen ausgewertet.

Eine gültige Freigabe ersetzt keine andere Sicherheitsprüfung.

---

## 7. Kopplung mit WF-0012

WF-0013 liest den aktuellen Dateistand nicht selbst.

Der Reader Request Builder erzeugt einen kontrollierten Leseauftrag für WF-0012.

Der Auftrag enthält ausschließlich die von WF-0012 v0.1.0 unterstützten Felder:

```text
owner
repository
path
ref
```

Dabei bildet WF-0013:

```text
target.owner      → owner
target.repository → repository
target.path       → path
target.branch     → ref
```

ab.

Die eigene `request_id` wird nicht an WF-0012 übergeben. WF-0013 muss sie intern erhalten und das Reader-Ergebnis kontrolliert dem ursprünglichen Änderungsauftrag zuordnen.

WF-0012 muss mindestens folgende bereinigte Ergebnisfelder liefern:

```text
status
mode
source
file
error
```

Für einen erfolgreichen Vergleich werden benötigt:

```text
source.owner
source.repository
source.ref
file.sha
file.path
file.content
file.encoding
```

WF-0013 prüft die Antwort von WF-0012 auf:

- erfolgreiche Ausführung
- `status` exakt `read`
- `mode` exakt `read-only`
- interne Zuordnung zum ursprünglichen Änderungsauftrag
- identischen Owner
- identisches Repository
- identischen Branch beziehungsweise `ref`
- identischen Dateipfad
- vorhandenen und formal gültigen SHA
- `file.encoding` exakt `base64`
- vorhandenen, bereits von WF-0012 dekodierten `file.content`-String
- widerspruchsfreie Status- und Fehlerfelder
- `error` muss bei erfolgreichem Ergebnis `null` sein
- keine sensiblen Daten in der Reader-Antwort

Eine unvollständige oder widersprüchliche Reader-Antwort wird abgelehnt.

---

## 8. Zustandsvergleich

Der State Comparator vergleicht den erwarteten, aktuellen und vorgeschlagenen Zustand.

### 8.1 SHA-Vergleich

Es gilt:

```text
target.expected_sha == reader_result.file.sha
```

Bei Abweichung liegt ein Konflikt vor.

Der Konflikt darf nicht automatisch aufgelöst werden.

Ergebnis:

```text
SHA_CONFLICT
```

### 8.2 Inhaltsvergleich

Es gilt:

```text
change.proposed_content == reader_result.file.content
```

Sind beide Inhalte identisch, ist keine Änderung erforderlich.

Ergebnis:

```text
NO_CHANGE
```

Für identische Inhalte darf kein Writer-Auftrag freigegeben werden.

### 8.3 Änderungsfall

Nur wenn:

- der SHA übereinstimmt
- der vorgeschlagene Inhalt vom aktuellen Inhalt abweicht
- alle übrigen Prüfungen erfolgreich sind

kann die Verarbeitung zur Freigabeprüfung und Payload-Erstellung fortgesetzt werden.

---

## 9. Entscheidungslogik

Die Decision Engine arbeitet deterministisch.

Die Fehlerpriorität lautet:

1. ungültige Eingangsstruktur
2. ungültiger Betriebsmodus
3. unzulässiges Ziel
4. ungültiger Änderungsinhalt
5. ungültiger Leseauftrag
6. Fehler von WF-0012
7. ungültige Reader-Antwort
8. Zielabweichung in der Reader-Antwort
9. SHA-Konflikt
10. unveränderter Inhalt
11. fehlende oder ungültige Freigabe
12. ungültiger Writer-Auftrag
13. erfolgreiche Vorbereitung

Ein Auftrag darf nur bei vollständig erfolgreicher Prüfung den Status `prepared` erhalten.

---

## 10. Writer-Payload für WF-0011

Der Writer Payload Builder erzeugt einen Auftrag gemäß dem dokumentierten Vertrag von WF-0011.

Das Zielmodell besteht aus:

```text
execution
target
source
change
request_id
```

Beispiel:

```json
{
  "execution": {
    "mode": "simulation"
  },
  "target": {
    "owner": "Jamoko112026",
    "repository": "JaMoKo_Automation_OS",
    "branch": "main",
    "path": "path/to/file.md"
  },
  "source": {
    "expected_sha": "CURRENT_FILE_SHA",
    "controller": "WF-0013",
    "controller_version": "v0.1.0"
  },
  "change": {
    "content": "Vollständiger neuer Dateiinhalt",
    "commit_message": "Update controlled file"
  },
  "request_id": "REQ-EXAMPLE-001"
}
```

Der konkrete Writer-Modus muss mit der tatsächlich freigegebenen Version von WF-0011 kompatibel sein.

WF-0013 darf keinen produktiven Modus erfinden oder aus einer Benutzereingabe ungeprüft übernehmen.

Der Writer-Auftrag wird ausschließlich erzeugt, wenn:

- alle Validierungen erfolgreich waren
- WF-0012 einen gültigen aktuellen Zustand geliefert hat
- der erwartete SHA übereinstimmt
- eine tatsächliche Inhaltsänderung vorliegt
- `approval.approved` exakt `true` ist

---

## 11. Ausgangsmodell

WF-0013 liefert genau ein bereinigtes Ergebnisobjekt.

Verbindliche Hauptfelder:

```text
workflow
version
mode
status
request_id
decision
target
comparison
approval
writer_request
execution
error
```

### 11.1 Erfolgreiche Vorbereitung

Beispiel:

```json
{
  "workflow": "WF-0013",
  "version": "v0.1.0",
  "mode": "prepare-only",
  "status": "prepared",
  "request_id": "REQ-EXAMPLE-001",
  "decision": {
    "allowed": true,
    "code": "WRITER_REQUEST_PREPARED"
  },
  "target": {
    "owner": "Jamoko112026",
    "repository": "JaMoKo_Automation_OS",
    "branch": "main",
    "path": "path/to/file.md"
  },
  "comparison": {
    "sha_matches": true,
    "content_changed": true
  },
  "approval": {
    "approved": true
  },
  "writer_request": {
    "prepared": true
  },
  "execution": {
    "writer_called": false,
    "write_executed": false,
    "commit_created": false,
    "push_executed": false
  },
  "error": null
}
```

### 11.2 Abgelehntes Ergebnis

Beispiel:

```json
{
  "workflow": "WF-0013",
  "version": "v0.1.0",
  "mode": "prepare-only",
  "status": "rejected",
  "request_id": "REQ-EXAMPLE-001",
  "decision": {
    "allowed": false,
    "code": "SHA_CONFLICT"
  },
  "writer_request": {
    "prepared": false
  },
  "execution": {
    "writer_called": false,
    "write_executed": false,
    "commit_created": false,
    "push_executed": false
  },
  "error": {
    "code": "SHA_CONFLICT",
    "message": "Der erwartete Dateistand stimmt nicht mit dem aktuellen Dateistand überein."
  }
}
```

`writer_request` darf den vollständigen Writer-Payload nur bei Status `prepared` enthalten.

---

## 12. Sicherheitsarchitektur

### 12.1 Allowlist

Zulässige Kombinationen aus Owner, Repository, Branch und Pfadbereich werden zentral definiert.

Nicht gelistete Ziele werden abgelehnt.

### 12.2 Pfadsicherheit

Der normalisierte Pfad darf nicht enthalten:

```text
..
./
\
://
NUL
Steuerzeichen
```

Absolute Pfade und leere Pfade sind unzulässig.

### 12.3 Geheimnisschutz

Folgende Daten dürfen nicht in Ausgaben oder Fehlerobjekten erscheinen:

- GitHub-Tokens
- Authorization-Header
- Zugangsdaten
- n8n-Credentials
- vollständige HTTP-Header
- interne Stack-Traces
- nicht bereinigte API-Antworten

### 12.4 Seiteneffektfreiheit

Für jede Ausführung von `v0.1.0` müssen folgende Werte gelten:

```text
writer_called = false
write_executed = false
commit_created = false
push_executed = false
```

Ein abweichender Wert stellt einen Architektur- und Sicherheitsfehler dar.

### 12.5 Inhaltsminimierung

Bei Fehlern soll der vollständige Dateiinhalt weder gespiegelt noch protokolliert werden.

Für Vergleiche dürfen intern Hashes, Längen oder kontrollierte Metadaten verwendet werden.

---

## 13. Fehlerklassen

WF-0013 verwendet stabile maschinenlesbare Fehlercodes.

Vorgesehene Fehlerklassen:

```text
INVALID_INPUT
INVALID_MODE
TARGET_NOT_ALLOWED
INVALID_PATH
INVALID_CHANGE
CONTENT_TOO_LARGE
INVALID_APPROVAL
READER_REQUEST_INVALID
READER_FAILED
READER_RESULT_INVALID
TARGET_MISMATCH
SHA_CONFLICT
NO_CHANGE
WRITER_REQUEST_INVALID
INTERNAL_CONTROLLER_ERROR
```

Fehlermeldungen müssen:

- verständlich
- deterministisch
- frei von Geheimnissen
- frei von Stack-Traces
- ohne vollständige Datei- oder Tokeninhalte

sein.

---

## 14. n8n-Komponentenmodell

Die spätere n8n-Implementierung soll mindestens folgende logische Nodes enthalten:

```text
Manual Trigger
Input Fixture
Normalize Input
Validate Schema
Validate Mode
Validate Target
Validate Change
Build Reader Request
Execute or Receive WF-0012 Result
Validate Reader Result
Compare Target
Compare SHA
Compare Content
Validate Approval
Build Writer Request
Validate Writer Request
Build Controller Result
Sanitize Output
```

Fehlerpfade werden in einer zentralen Ergebnisbildung zusammengeführt.

Der Writer-Workflow darf in `v0.1.0` nicht durch einen Execute-Workflow-Node aufgerufen werden.

---

## 15. Zustandsmodell

WF-0013 kennt folgende fachliche Zustände:

```text
received
validated
reader-request-prepared
reader-result-received
compared
rejected
prepared
failed
```

Zulässige Übergänge:

```text
received
  -> validated
  -> reader-request-prepared
  -> reader-result-received
  -> compared
  -> prepared
```

Jede Prüfphase kann nach:

```text
rejected
```

oder bei einem internen technischen Fehler nach:

```text
failed
```

wechseln.

Nach `prepared`, `rejected` oder `failed` erfolgt kein weiterer Verarbeitungsschritt.

---

## 16. Invarianten

Folgende Bedingungen müssen immer gelten:

1. WF-0013 schreibt niemals selbst auf GitHub.
2. WF-0013 führt WF-0011 in `v0.1.0` niemals automatisch aus.
3. Ohne gültigen Reader-Zustand entsteht kein Writer-Auftrag.
4. Bei SHA-Abweichung entsteht kein Writer-Auftrag.
5. Bei identischem Inhalt entsteht kein Writer-Auftrag.
6. Ohne Boolean-Freigabe `true` entsteht kein Writer-Auftrag.
7. Ein unzulässiges Ziel wird vor der Reader-Verarbeitung abgelehnt.
8. Jede Ausgabe enthält `workflow`, `version`, `mode`, `status` und `request_id`.
9. Sensible Daten erscheinen weder in Erfolgs- noch in Fehlerausgaben.
10. `write_executed`, `commit_created` und `push_executed` bleiben immer `false`.

---

## 17. Abgrenzung zu WF-0012

WF-0012 ist verantwortlich für:

- kontrolliertes Lesen
- Validierung des Leseziels
- Abruf des aktuellen Dateistands
- bereinigte Rückgabe von SHA, Inhalt und Metadaten

WF-0013 übernimmt nicht:

- direkte GitHub-Lesezugriffe
- Reader-Credentials
- Reader-spezifische HTTP-Verarbeitung

WF-0013 prüft jedoch, ob die gelieferte Reader-Antwort zum ursprünglichen Auftrag passt.

---

## 18. Abgrenzung zu WF-0011

WF-0011 ist verantwortlich für:

- Validierung seines Writer-Auftrags
- Simulation oder spätere kontrollierte Schreibausführung
- Dateiänderung
- Commit-Erstellung
- optionalen Push innerhalb eines freigegebenen Betriebsmodus
- eigenes bereinigtes Ergebnis

WF-0013 übernimmt nicht:

- GitHub-Schreibzugriffe
- Writer-Credentials
- Commit-Ausführung
- Push-Ausführung
- automatische Aktivierung eines produktiven Writer-Modus

---

## 19. Versionierungsgrenze

Diese Architektur gilt ausschließlich für:

```text
WF-0013 v0.1.0
```

Der verbindliche Betriebsmodus lautet:

```text
prepare-only
```

Eine spätere automatische Übergabe oder Ausführung von WF-0011 erfordert:

- eine neue WF-0013-Version
- einen dokumentierten Schnittstellenvertrag
- abgeschlossene Sicherheits- und Funktionstests
- eine ausdrückliche Freigabe
- eine aktualisierte Architektur
- einen aktualisierten Flow
- aktualisierte Tests
- einen Changelog-Eintrag

---

## 20. Abnahmekriterien

Die Architektur ist technisch umsetzbar, wenn:

1. das Eingangsmodell vollständig definiert ist,
2. die Fehlerpriorität deterministisch implementiert werden kann,
3. die Schnittstelle zu WF-0012 eindeutig ist,
4. die Schnittstelle zu WF-0011 eindeutig ist,
5. SHA- und Inhaltsvergleich getrennt erfolgen,
6. die Freigabe ausschließlich als Boolean geprüft wird,
7. der Writer-Payload nur bei vollständigem Erfolg entsteht,
8. kein Pfad WF-0011 automatisch ausführt,
9. alle Ausgaben zentral bereinigt werden,
10. sämtliche Seiteneffekt-Flags nachweislich `false` bleiben.

---

## 21. Architekturentscheidung

WF-0013 wird in `v0.1.0` als kontrollierender, seiteneffektfreier Orchestrierungsworkflow aufgebaut.

Er liest nicht selbst, schreibt nicht selbst und führt keinen Writer aus.

Sein einziges positives Endergebnis ist ein validierter, nachvollziehbarer und noch nicht ausgeführter Writer-Auftrag.

Damit bildet WF-0013 die Sicherheits- und Entscheidungsschicht zwischen Änderungsabsicht, aktuellem GitHub-Zustand und einer möglichen späteren Schreibverarbeitung.
