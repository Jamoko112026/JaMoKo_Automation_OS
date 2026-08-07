# WF-0013 – GitHub Change Controller

Version: `v0.1.0`
Status: `draft`
Typ: `Controller`
Betriebsmodus: `prepare-only`

---

## 1. Testziel

Die Tests bestätigen, dass WF-0013:

- Änderungsaufträge zuverlässig validiert
- nur freigegebene Ziele verarbeitet
- unsichere Dateipfade ablehnt
- den aktuellen Dateistand ausschließlich über WF-0012 bezieht
- SHA-Konflikte erkennt
- unnötige Änderungen verhindert
- ausschließlich eine ausdrückliche Boolean-Freigabe akzeptiert
- einen korrekten Writer-Payload vorbereitet
- WF-0011 niemals automatisch ausführt
- niemals direkt auf GitHub schreibt
- Fehler und Ausgaben sicher bereinigt
- `write_executed` immer auf `false` setzt

---

## 2. Testvoraussetzungen

Vor dem Test müssen folgende Voraussetzungen erfüllt sein:

- WF-0012 ist verfügbar und getestet.
- WF-0011 ist vorhanden, wird aber nicht automatisch ausgeführt.
- WF-0013 befindet sich im Modus `prepare-only`.
- Das Test-Repository ist in der Allowlist eingetragen.
- Eine bekannte Textdatei mit aktuellem SHA-Wert ist vorhanden.
- GitHub-Credentials sind ausschließlich in WF-0012 beziehungsweise WF-0011 hinterlegt.
- WF-0013 besitzt keine GitHub-Credentials.
- Alle Tests verwenden ausschließlich unkritische Testinhalte.
- Der Workflow ist während der Tests nicht produktiv schreibend verbunden.

---

## 3. Referenz-Testdaten

Zulässiges Testziel:

```json
{
  "owner": "Jamoko112026",
  "repository": "JaMoKo_Automation_OS",
  "branch": "main",
  "path": "test-fixtures/controller-target.md",
  "expected_sha": "1111111111111111111111111111111111111111"
}
```
Zulässiger Änderungsauftrag:

```json
{
  "execution": {
    "mode": "prepare-only"
  },
  "target": {
    "owner": "Jamoko112026",
    "repository": "JaMoKo_Automation_OS",
    "branch": "main",
    "path": "test-fixtures/controller-target.md",
    "expected_sha": "1111111111111111111111111111111111111111"
  },
  "change": {
    "proposed_content": "Neuer kontrollierter Testinhalt.\n",
    "commit_message": "Update controller test fixture"
  },
  "approval": {
    "approved": true
  },
  "request_id": "REQ-WF0013-TEST-001"
}
```
Gültiges Reader-Ergebnis von WF-0012:

```json
{
  "status": "success",
  "request_id": "REQ-WF0013-TEST-001",
  "target": {
    "owner": "Jamoko112026",
    "repository": "JaMoKo_Automation_OS",
    "branch": "main",
    "path": "test-fixtures/controller-target.md"
  },
  "file": {
    "sha": "1111111111111111111111111111111111111111",
    "content": "Alter kontrollierter Testinhalt.\n"
  }
}
```
Der tatsächliche SHA-Wert darf im Testsystem abweichen. Entscheidend ist, dass `target.expected_sha` und `file.sha` für den Erfolgsfall exakt übereinstimmen.

---

## 4. Testmethodik

Jeder Testfall wird isoliert ausgeführt.

Für jeden Test werden mindestens folgende Werte dokumentiert:

- Testfall-ID
- Eingangsauftrag
- simuliertes oder reales Reader-Ergebnis
- erwarteter Controller-Status
- erwarteter Entscheidungscode
- Vorhandensein oder Fehlen von `writer_request`
- Wert von `write_executed`
- beobachtetes Ergebnis
- Testergebnis

Zulässige Testergebnisse:

```text
passed
failed
blocked
```
WF-0011 darf während der Tests nicht automatisch ausgeführt werden.

Ein Test gilt nur als bestanden, wenn:

- der erwartete Status erreicht wird
- der erwartete Entscheidungscode ausgegeben wird
- keine unzulässigen Seiteneffekte auftreten
- `write_executed` den Wert `false` besitzt
- sensible Daten nicht in der Ausgabe erscheinen

---

## 5. Testmatrix

| ID | Testfall | Erwarteter Code | Writer-Payload |
| --- | --- | --- | --- |
| T-001 | Gültiger Änderungsauftrag | `WRITER_REQUEST_PREPARED` | vorhanden |
| T-002 | Eingang ist kein gültiges Objekt | `INVALID_INPUT` | nicht vorhanden |
| T-003 | Unzulässiger Betriebsmodus | `INVALID_MODE` | nicht vorhanden |
| T-004 | Ziel nicht in Allowlist | `TARGET_NOT_ALLOWED` | nicht vorhanden |
| T-005 | Unsicherer Dateipfad | `INVALID_PATH` | nicht vorhanden |
| T-006 | Ungültige Änderungsdaten | `INVALID_CHANGE` | nicht vorhanden |
| T-007 | Inhalt überschreitet Größenlimit | `CONTENT_TOO_LARGE` | nicht vorhanden |
| T-008 | Freigabe fehlt oder ist nicht Boolean `true` | `INVALID_APPROVAL` | nicht vorhanden |
| T-009 | Reader-Auftrag kann nicht erstellt werden | `READER_REQUEST_INVALID` | nicht vorhanden |
| T-010 | WF-0012 meldet einen Fehler | `READER_FAILED` | nicht vorhanden |
| T-011 | Reader-Ergebnis ist strukturell ungültig | `READER_RESULT_INVALID` | nicht vorhanden |
| T-012 | Reader-Ziel weicht vom Auftrag ab | `TARGET_MISMATCH` | nicht vorhanden |
| T-013 | Erwarteter und aktueller SHA unterscheiden sich | `SHA_CONFLICT` | nicht vorhanden |
| T-014 | Vorgeschlagener Inhalt ist unverändert | `NO_CHANGE` | nicht vorhanden |
| T-015 | Writer-Payload kann nicht gültig erstellt werden | `WRITER_REQUEST_INVALID` | nicht vorhanden |
| T-016 | Unerwarteter interner Controller-Fehler | `INTERNAL_CONTROLLER_ERROR` | nicht vorhanden |
| T-017 | Ausgabe enthält keine Geheimnisse | fallabhängig | fallabhängig |
| T-018 | WF-0011 wird niemals automatisch ausgeführt | `WRITER_REQUEST_PREPARED` | vorhanden, nicht ausgeführt |
| T-019 | Direkter GitHub-Schreibzugriff bleibt ausgeschlossen | `WRITER_REQUEST_PREPARED` | vorhanden, nicht ausgeführt |
| T-020 | Wiederholte identische Eingabe liefert dasselbe Ergebnis | `WRITER_REQUEST_PREPARED` | vorhanden |

---

## 6. Positive Testfälle

### 6.1 T-001 – Writer-Auftrag erfolgreich vorbereiten

Voraussetzungen:

- alle Pflichtfelder sind vorhanden
- `execution.mode` ist `prepare-only`
- das Ziel ist freigegeben
- der Pfad ist sicher
- `approval.approved` ist Boolean `true`
- WF-0012 liefert ein gültiges Ergebnis
- erwarteter und aktueller SHA stimmen überein
- vorgeschlagener und aktueller Inhalt unterscheiden sich
- der Writer-Payload erfüllt den Vertrag von WF-0011

Erwartetes Ergebnis:

```text
status = prepared
code = WRITER_REQUEST_PREPARED
writer_request.execution.mode = simulation
write_executed = false
```
Zusätzlich gilt:

- `writer_request` ist vorhanden
- WF-0011 wurde nicht aufgerufen
- kein Commit wurde erstellt
- kein Push wurde ausgeführt
- GitHub wurde nicht schreibend angesprochen

### 6.2 T-020 – Deterministische Wiederholung

Der vollständige Erfolgsfall wird zweimal mit identischen Eingangs- und Reader-Daten ausgeführt.

Erwartetes Ergebnis:

- Status und Entscheidungscode sind identisch
- der fachliche Inhalt des Writer-Payloads ist identisch
- `write_executed` bleibt in beiden Durchläufen `false`
- es entstehen keine Seiteneffekte

---

## 7. Eingabe- und Validierungstests

### 7.1 T-002 – Ungültige Grundstruktur

Beispiele:

- `null`
- Array statt Objekt
- leerer Eingang
- fehlende Hauptobjekte

Erwartetes Ergebnis:

```text
code = INVALID_INPUT
write_executed = false
```
### 7.2 T-003 – Ungültiger Betriebsmodus

Beispiele:

```text
execute
write
production
simulation
```
Erwartetes Ergebnis:

```text
code = INVALID_MODE
write_executed = false
```
Nur `prepare-only` ist als Controller-Modus zulässig.

### 7.3 T-004 – Nicht freigegebenes Ziel

Mindestens einer dieser Werte liegt außerhalb der Allowlist:

- Owner
- Repository
- Branch
- Dateipfad

Erwartetes Ergebnis:

```text
code = TARGET_NOT_ALLOWED
write_executed = false
```
### 7.4 T-005 – Unsicherer Dateipfad

Beispiele:

```text
../secret.txt
/absolute/path.txt
test-fixtures/../../secret.txt
```
Zusätzlich werden Pfade mit Steuerzeichen oder NUL-Zeichen geprüft.

Erwartetes Ergebnis:

```text
code = INVALID_PATH
write_executed = false
```
### 7.5 T-006 – Ungültige Änderungsdaten

Beispiele:

- `proposed_content` fehlt
- `proposed_content` ist kein String
- `commit_message` fehlt
- `commit_message` ist leer
- `commit_message` erfüllt den Vertrag von WF-0011 nicht

Erwartetes Ergebnis:

```text
code = INVALID_CHANGE
write_executed = false
```
### 7.6 T-007 – Inhalt zu groß

`proposed_content` überschreitet die dokumentierte Maximalgröße.

Erwartetes Ergebnis:

```text
code = CONTENT_TOO_LARGE
write_executed = false
```
### 7.7 T-008 – Ungültige Freigabe

Zu prüfen sind mindestens:

```text
approval fehlt
approved fehlt
approved = false
approved = "true"
approved = 1
approved = null
```
Erwartetes Ergebnis:

```text
code = INVALID_APPROVAL
write_executed = false
```
Nur der Boolean-Wert `true` gilt als Freigabe.

---

## 8. Reader-Integrationstests

### 8.1 T-009 – Ungültiger Reader-Auftrag

Der Controller kann aus dem geprüften Ziel keinen vertragskonformen Auftrag für WF-0012 erzeugen.

Erwartetes Ergebnis:

```text
code = READER_REQUEST_INVALID
write_executed = false
```
### 8.2 T-010 – Reader fehlgeschlagen

WF-0012 liefert einen Fehlerstatus oder ist nicht erfolgreich ausführbar.

Erwartetes Ergebnis:

```text
code = READER_FAILED
write_executed = false
```
### 8.3 T-011 – Ungültiges Reader-Ergebnis

Beispiele:

- Ergebnis ist kein Objekt
- Pflichtfelder fehlen
- SHA fehlt
- Inhalt ist kein String
- Status ist unbekannt
- `request_id` ist nicht zuordenbar

Erwartetes Ergebnis:

```text
code = READER_RESULT_INVALID
write_executed = false
```
### 8.4 T-012 – Abweichendes Ziel

Mindestens eines der vom Reader bestätigten Zielfelder stimmt nicht mit dem ursprünglichen Änderungsauftrag überein.

Zu prüfen sind:

- Owner
- Repository
- Branch
- Dateipfad

Erwartetes Ergebnis:

```text
code = TARGET_MISMATCH
write_executed = false
```
---

## 9. Zustandsvergleichstests

### 9.1 T-013 – SHA-Konflikt

`target.expected_sha` stimmt nicht mit dem von WF-0012 gelieferten aktuellen SHA überein.

Erwartetes Ergebnis:

```text
code = SHA_CONFLICT
write_executed = false
```
Es darf kein Writer-Payload entstehen.

### 9.2 T-014 – Keine inhaltliche Änderung

`change.proposed_content` entspricht exakt dem aktuellen Dateiinhalt.

Erwartetes Ergebnis:

```text
code = NO_CHANGE
write_executed = false
```
Es darf kein Writer-Payload entstehen.

### 9.3 Reihenfolge der Zustandsprüfung

Wenn gleichzeitig ein SHA-Konflikt und identischer Inhalt vorliegen, muss die in der Spezifikation definierte Fehlerpriorität gelten.

Das beobachtete Ergebnis muss reproduzierbar sein und dem kanonischen Entscheidungscode entsprechen.

---

## 10. Writer-Payload-Tests

### 10.1 T-015 – Ungültiger Writer-Payload

Die Voraussetzungen für eine Änderung sind erfüllt, aber der erzeugte Writer-Payload verletzt den Schnittstellenvertrag von WF-0011.

Erwartetes Ergebnis:

```text
code = WRITER_REQUEST_INVALID
write_executed = false
```
Der ungültige Payload darf nicht als ausführbarer Auftrag ausgegeben werden.

### 10.2 Writer-Modus

Im positiven Fall muss gelten:

```text
writer_request.execution.mode = simulation
```
Andere Writer-Modi sind in WF-0013 `v0.1.0` unzulässig.

### 10.3 Keine automatische Writer-Ausführung

Auch bei `WRITER_REQUEST_PREPARED` muss nachgewiesen werden:

- WF-0011 wurde nicht automatisch aufgerufen
- keine Writer-Credentials wurden verwendet
- kein Commit wurde erstellt
- kein Push wurde ausgeführt
- kein GitHub-Schreibzugriff fand statt
- `write_executed` ist `false`

---

## 11. Sicherheits- und Bereinigungstests

### 11.1 T-017 – Geheimnisschutz

Eingaben und simulierte Fehler werden mit Testmarkern versehen, die Geheimnisse darstellen.

Zu prüfen sind mindestens:

- GitHub-Token
- Authorization-Header
- n8n-Credentials
- interne Header
- technische Rohfehler mit sensiblen Details

Erwartetes Ergebnis:

- kein Testmarker erscheint in der Controller-Ausgabe
- interne Fehler werden bereinigt
- notwendige fachliche Fehlerangaben bleiben erhalten
- `write_executed` bleibt `false`

Es dürfen ausschließlich künstliche Testwerte verwendet werden.

### 11.2 T-019 – Kein direkter GitHub-Schreibzugriff

Während eines vollständigen Erfolgsfalls werden die ausgeführten Komponenten und Verbindungen geprüft.

Erwartetes Ergebnis:

- WF-0013 verwendet keine GitHub-Schreib-Credentials
- WF-0013 ruft keinen schreibenden GitHub-Endpunkt auf
- WF-0013 führt keine Git-Befehle aus
- WF-0013 erstellt weder Commit noch Push
- die einzige positive Ausgabe ist ein vorbereiteter Writer-Payload

### 11.3 Seiteneffekt-Flags

Für jeden Testfall muss gelten:

```text
write_executed = false
commit_created = false
push_executed = false
```
Sofern ein Flag im jeweiligen Ausgangsmodell enthalten ist, muss es ausdrücklich `false` sein. Ein fehlendes vorgeschriebenes Flag gilt als Testfehler.

---

## 12. Interner Fehlerfall

### 12.1 T-016 – Unerwarteter Controller-Fehler

Ein kontrollierter interner Testfehler wird ausgelöst, ohne echte Zugangsdaten oder produktive Systeme einzubeziehen.

Erwartetes Ergebnis:

```text
code = INTERNAL_CONTROLLER_ERROR
write_executed = false
```
Zusätzlich gilt:

- keine interne Fehlerspur mit Geheimnissen wird ausgegeben
- kein Writer-Payload wird bereitgestellt
- es entstehen keine Seiteneffekte

---

## 13. Fehlerpriorität

Kombinierte Fehlerfälle prüfen, ob immer der in der Spezifikation festgelegte priorisierte Entscheidungscode entsteht.

Mindestens folgende Kombinationen werden getestet:

1. ungültiger Modus und ungültige Freigabe
2. nicht freigegebenes Ziel und SHA-Konflikt
3. Reader-Fehler und abweichendes Ziel
4. SHA-Konflikt und unveränderter Inhalt
5. ungültiger Writer-Payload nach ansonsten erfolgreicher Prüfung

Für identische Eingangsdaten muss bei jeder Wiederholung derselbe Entscheidungscode entstehen.

---

## 14. Testprotokoll

| ID | Datum | Status | Beobachteter Code | `write_executed` | Bemerkung |
| --- | --- | --- | --- | --- | --- |
| T-001 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-002 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-003 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-004 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-005 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-006 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-007 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-008 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-009 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-010 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-011 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-012 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-013 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-014 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-015 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-016 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-017 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-018 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-019 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |
| T-020 | offen | blocked | nicht ausgeführt | nicht geprüft | Implementierung fehlt |

---

## 15. Abnahmekriterien

`TESTS.md` gilt für `v0.1.0` als erfüllt, wenn:

1. alle Testfälle ausgeführt und dokumentiert sind,
2. alle erwarteten Entscheidungscodes bestätigt wurden,
3. die Fehlerpriorität deterministisch nachgewiesen ist,
4. der Erfolgsfall ausschließlich `WRITER_REQUEST_PREPARED` erzeugt,
5. der Writer-Modus ausschließlich `simulation` lautet,
6. WF-0011 in keinem Test automatisch ausgeführt wurde,
7. kein direkter GitHub-Schreibzugriff stattgefunden hat,
8. alle vorgeschriebenen Seiteneffekt-Flags `false` sind,
9. keine Geheimnisse oder Zugangsdaten ausgegeben wurden,
10. alle Pflichtprüfungen den Status `passed` besitzen.

Bis zur vorhandenen Implementierung und vollständigen Testausführung bleibt der Dokumentstatus:

```text
draft
```
---

## 16. Testergebnis für die Dokumentationsphase

In der aktuellen Dokumentationsphase beschreibt diese Datei die verbindlichen Tests für die spätere Implementierung von WF-0013.

Die Testfälle sind noch nicht technisch ausgeführt.

Daher gilt:

```text
Gesamtstatus: blocked
Grund: Implementierung von WF-0013 noch nicht vorhanden
```
Das ist kein fachlicher Fehler der Testakte.

Die Testakte bildet die Grundlage für Implementierung, technische Abnahme und die spätere Freigabe von WF-0013.
