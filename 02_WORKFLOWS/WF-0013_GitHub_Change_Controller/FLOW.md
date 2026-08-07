# WF-0013 – GitHub Change Controller – Flow

Version: `v0.1.0`
Status: `draft`
Typ: `Ablaufbeschreibung`
Betriebsmodus: `prepare-only`

---

## 1. Ziel des Ablaufs

WF-0013 prüft einen strukturierten Änderungsauftrag gegen den aktuellen Zustand einer freigegebenen GitHub-Datei.

Der Workflow:

1. validiert den Änderungsauftrag und den Betriebsmodus
2. prüft Repository, Allowlist und Dateipfad
3. prüft den vorgeschlagenen Änderungsinhalt
4. erzeugt einen Leseauftrag für WF-0012
5. liest den aktuellen Zustand ausschließlich über WF-0012
6. ordnet das Reader-Ergebnis intern dem Änderungsauftrag zu
7. vergleicht Zielinformationen, SHA und Dateiinhalt
8. prüft die ausdrückliche Freigabe
9. bereitet bei vollständigem Erfolg einen Writer-Payload für WF-0011 im Modus `simulation` vor
10. gibt ein bereinigtes Ergebnisobjekt aus
11. endet immer ohne Schreibzugriff

---

## 2. Hauptablauf

```text
Änderungsauftrag
      |
      v
Eingangsstruktur validieren
      |
      v
Betriebsmodus prüfen
      |
      v
Ziel und Allowlist prüfen
      |
      v
Zielpfad prüfen
      |
      v
Änderungsinhalt prüfen
      |
      v
Reader-Auftrag für WF-0012 erzeugen
      |
      v
WF-0012 ausführen oder Reader-Ergebnis übernehmen
      |
      v
Reader-Ergebnis validieren
      |
      v
Zielinformationen vergleichen
      |
      v
SHA vergleichen
      |
      v
Aktuellen Dateiinhalt dekodieren und prüfen
      |
      v
Aktuellen und vorgeschlagenen Inhalt vergleichen
      |
      v
Ausdrückliche Freigabe prüfen
      |
      v
Writer-Auftrag validieren
      |
      v
Writer-Payload im Modus simulation vorbereiten
      |
      v
Bereinigtes Ergebnisobjekt ausgeben
      |
      v
Ende ohne Schreibzugriff
```

Die Prüfungen werden in der dargestellten Reihenfolge ausgeführt. Sobald eine Prüfung fehlschlägt, wird der Ablauf kontrolliert beendet. Nachfolgende Verarbeitungsschritte dürfen dann nicht mehr ausgeführt werden.

---

## 3. Entscheidungsreihenfolge

Die Prüfungen erfolgen verbindlich in dieser Priorität:

1. Eingangsstruktur
2. Betriebsmodus
3. Ziel- und Allowlist-Prüfung
4. Pfadprüfung
5. Änderungsinhalt
6. Reader-Auftrag
7. Reader-Ausführung oder Reader-Ergebnis
8. Validität des Reader-Ergebnisses
9. Übereinstimmung des Ziels
10. SHA-Vergleich
11. Inhaltsvergleich
12. Freigabe
13. Writer-Auftrag
14. erfolgreiche Vorbereitung

Bei mehreren möglichen Fehlern wird ausschließlich der zuerst erreichte Fehler gemäß dieser Reihenfolge ausgegeben.

Dadurch bleibt das Ergebnis deterministisch und reproduzierbar.

---

## 4. Fehler- und Abbruchpfade

Jede Prüfung besitzt einen kontrollierten Abbruchpfad.

| Prüfpunkt | Entscheidungscode | Status | Ergebnis |
|---|---|---|---|
| Eingangsobjekt oder Pflichtfeld ungültig | `INVALID_INPUT` | `rejected` | Auftrag ablehnen |
| Betriebsmodus ungültig | `INVALID_MODE` | `rejected` | Auftrag ablehnen |
| Ziel nicht durch die Allowlist freigegeben | `TARGET_NOT_ALLOWED` | `rejected` | Auftrag ablehnen |
| Zielpfad ungültig oder unsicher | `INVALID_PATH` | `rejected` | Auftrag ablehnen |
| Änderungsinhalt oder Commit-Nachricht ungültig | `INVALID_CHANGE` | `rejected` | Auftrag ablehnen |
| Änderungsinhalt überschreitet die zulässige Größe | `CONTENT_TOO_LARGE` | `rejected` | Auftrag ablehnen |
| Reader-Auftrag für WF-0012 ungültig | `READER_REQUEST_INVALID` | `rejected` | Ablauf beenden |
| WF-0012 technisch oder fachlich fehlgeschlagen | `READER_FAILED` | `rejected` | Ablauf beenden |
| Reader-Ergebnis verletzt den erwarteten Vertrag | `READER_RESULT_INVALID` | `rejected` | Ablauf beenden |
| Reader-Ergebnis gehört nicht zum angeforderten Ziel | `TARGET_MISMATCH` | `rejected` | Auftrag ablehnen |
| Erwarteter SHA stimmt nicht mit dem aktuellen SHA überein | `SHA_CONFLICT` | `rejected` | Auftrag ablehnen |
| Vorgeschlagener Inhalt entspricht dem aktuellen Inhalt | `NO_CHANGE` | `rejected` | keinen Writer-Payload vorbereiten |
| Ausdrückliche Freigabe fehlt oder ist ungültig | `INVALID_APPROVAL` | `rejected` | Auftrag ablehnen |
| Writer-Auftrag verletzt den Writer-Vertrag | `WRITER_REQUEST_INVALID` | `rejected` | Ablauf beenden |
| Interner technischer Controller-Fehler | `INTERNAL_CONTROLLER_ERROR` | `failed` | Ablauf kontrolliert beenden |

Bei jeder fachlichen oder validierungsbedingten Ablehnung gilt:

```text
status = rejected
decision.allowed = false
writer_request.prepared = false
execution.writer_called = false
execution.write_executed = false
execution.commit_created = false
execution.push_executed = false
```

Bei einem internen technischen Fehler gilt:

```text
status = failed
decision.allowed = false
decision.code = INTERNAL_CONTROLLER_ERROR
writer_request.prepared = false
execution.writer_called = false
execution.write_executed = false
execution.commit_created = false
execution.push_executed = false
```

WF-0013 darf nach einem fehlgeschlagenen Prüfpunkt keine nachfolgenden Verarbeitungsschritte mehr ausführen.

Insbesondere darf nach einer Ablehnung oder einem Fehler:

- kein Writer-Payload als vorbereitet markiert werden
- WF-0011 nicht aufgerufen werden
- kein Schreibzugriff stattfinden
- kein Commit erzeugt werden
- kein Push ausgeführt werden
- kein erwarteter SHA automatisch ersetzt werden
- keine automatische Konfliktauflösung erfolgen

---

## 5. Reader-Ablauf

WF-0013 besitzt keine eigene direkte GitHub-Leselogik.

Der aktuelle Dateistand darf ausschließlich über WF-0012 ermittelt werden.

Der Reader-Ablauf besteht aus folgenden Schritten:

1. Reader-Auftrag aus dem normalisierten Änderungsauftrag erzeugen
2. Reader-Auftrag gegen den Vertrag von WF-0012 prüfen
3. WF-0012 ausführen oder ein kontrolliert übergebenes Reader-Ergebnis übernehmen
4. Reader-Ergebnis auf Struktur und Pflichtfelder prüfen
5. Zielinformationen des Reader-Ergebnisses mit dem Änderungsauftrag vergleichen
6. aktuellen SHA übernehmen
7. aktuellen Dateiinhalt dekodieren und prüfen
8. Ergebnis ausschließlich intern für die Entscheidung verwenden

Ein ungültiger Reader-Auftrag führt zu:

```text
status = rejected
decision.code = READER_REQUEST_INVALID
writer_request.prepared = false
```

Ein fehlgeschlagenes Reader-Ergebnis führt zu:

```text
status = rejected
decision.code = READER_FAILED
writer_request.prepared = false
```

Ein strukturell ungültiges Reader-Ergebnis führt zu:

```text
status = rejected
decision.code = READER_RESULT_INVALID
writer_request.prepared = false
```

WF-0013 darf kein unbereinigtes Reader-Ergebnis und keine vollständigen technischen HTTP-Antworten nach außen spiegeln.

---

## 6. Vergleichsablauf

Nach erfolgreicher Validierung des Reader-Ergebnisses führt WF-0013 die Vergleiche in fester Reihenfolge aus.

### 6.1 Zielvergleich

Die Zielinformationen des Reader-Ergebnisses müssen vollständig mit dem normalisierten Änderungsauftrag übereinstimmen.

Geprüft werden mindestens:

```text
target.owner
target.repository
target.branch
target.path
```

Bei einer Abweichung gilt:

```text
status = rejected
decision.code = TARGET_MISMATCH
comparison.sha_matches = null
comparison.content_changed = null
writer_request.prepared = false
```

### 6.2 SHA-Vergleich

Der vom Auftrag erwartete SHA muss mit dem aktuellen SHA des Reader-Ergebnisses übereinstimmen.

Bei einer Abweichung gilt:

```text
status = rejected
decision.code = SHA_CONFLICT
comparison.sha_matches = false
comparison.content_changed = null
writer_request.prepared = false
```

WF-0013 darf den erwarteten SHA nicht automatisch aktualisieren oder ersetzen.

### 6.3 Inhaltsvergleich

Erst nach erfolgreichem SHA-Vergleich wird der vorgeschlagene vollständige Dateiinhalt mit dem aktuellen Dateiinhalt verglichen.

Sind beide Inhalte identisch, gilt:

```text
status = rejected
decision.code = NO_CHANGE
comparison.sha_matches = true
comparison.content_changed = false
writer_request.prepared = false
```

Unterscheiden sich beide Inhalte, gilt zunächst:

```text
comparison.sha_matches = true
comparison.content_changed = true
```

Danach wird die ausdrückliche Freigabe geprüft.

---

## 7. Freigabeprüfung

Die Freigabeprüfung erfolgt erst nach erfolgreichem Ziel-, SHA- und Inhaltsvergleich.

Eine gültige Freigabe muss ausdrücklich im Änderungsauftrag enthalten sein.

Zulässig ist ausschließlich:

```text
approval.approved = true
```

Eine fehlende, falsche oder strukturell ungültige Freigabe führt zu:

```text
status = rejected
decision.allowed = false
decision.code = INVALID_APPROVAL
writer_request.prepared = false
```

WF-0013 darf eine Freigabe nicht:

- selbst erzeugen
- aus anderen Feldern ableiten
- aus früheren Aufträgen übernehmen
- automatisch ergänzen
- nachträglich ersetzen

---

## 8. Writer-Vorbereitung

Ein Writer-Payload darf ausschließlich vorbereitet werden, wenn alle vorherigen Prüfungen erfolgreich abgeschlossen wurden.

Dazu müssen gleichzeitig folgende Bedingungen erfüllt sein:

```text
Eingang gültig
Betriebsmodus gültig
Ziel freigegeben
Pfad gültig
Änderungsinhalt gültig
Reader-Auftrag gültig
Reader-Ergebnis gültig
Zielinformationen identisch
SHA identisch
Inhalt geändert
Freigabe ausdrücklich erteilt
Writer-Auftrag gültig
```

Der vorbereitete Writer-Payload:

- richtet sich an WF-0011
- enthält den vollständig validierten Zielzustand
- verwendet den bestätigten erwarteten SHA
- enthält den vollständigen neuen Dateiinhalt
- enthält die validierte Commit-Nachricht
- enthält die zugehörige `request_id`
- nennt WF-0013 als Controller
- setzt den Writer-Modus ausschließlich auf `simulation`

Der Writer-Payload wird nur vorbereitet und im bereinigten Ergebnisobjekt ausgegeben.

WF-0011 wird nicht automatisch aufgerufen.

---

## 9. Erfolgsbedingung

Ein erfolgreich vorbereiteter Ablauf besitzt:

```text
status = prepared
decision.allowed = true
decision.code = WRITER_REQUEST_PREPARED
comparison.sha_matches = true
comparison.content_changed = true
approval.approved = true
writer_request.prepared = true
writer_request.payload.execution.mode = simulation
execution.writer_called = false
execution.write_executed = false
execution.commit_created = false
execution.push_executed = false
error = null
```

`WRITER_REQUEST_PREPARED` bedeutet ausschließlich, dass ein vollständig geprüfter Writer-Auftrag zur weiteren kontrollierten Verarbeitung bereitgestellt wurde.

Der Code bedeutet nicht:

- dass WF-0011 aufgerufen wurde
- dass eine Datei geschrieben wurde
- dass ein Commit erzeugt wurde
- dass ein Push ausgeführt wurde
- dass eine produktive Änderung erfolgt ist

---

## 10. Bereinigte Ausgabe

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

Zulässige Statuswerte:

```text
prepared
rejected
failed
```

Die Ausgabe darf nicht enthalten:

- GitHub-Tokens
- Credentials
- Authorization-Header
- vollständige HTTP-Header
- interne Stack-Traces
- ungefilterte technische Antworten
- vollständige Dateiinhalte innerhalb von Fehlerobjekten

---

## 11. Sicherheitsgrenze

WF-0013:

- besitzt keine direkte GitHub-Schreibberechtigung
- führt keine direkte GitHub-Leseoperation aus
- enthält keine GitHub-Tokens oder Writer-Credentials
- erzeugt keine Authorization-Header
- ergänzt keine Credentials
- umgeht keine Allowlist
- verarbeitet keine unsicheren Zielpfade
- verändert den ursprünglichen Änderungsauftrag nicht
- ersetzt keinen erwarteten SHA automatisch
- löst keine Konflikte automatisch auf
- erzeugt keine Freigabe
- setzt den Writer-Modus ausschließlich auf `simulation`
- führt WF-0011 nicht automatisch aus
- erzeugt keinen Commit
- führt keinen Push aus
- erzeugt im Modus `prepare-only` keine produktiven Seiteneffekte

Diese Sicherheitsgrenze gilt auch für erfolgreich vorbereitete Ergebnisse.

---

## 12. Seiteneffektfreiheit

WF-0013 arbeitet ausschließlich im Modus:

```text
prepare-only
```

Unabhängig vom Ergebnis müssen folgende Werte immer gelten:

```text
execution.writer_called = false
execution.write_executed = false
execution.commit_created = false
execution.push_executed = false
```

Der Workflow darf weder GitHub noch ein lokales Repository verändern.

Ein vorbereitetes Ergebnis ist ausschließlich ein geprüftes Übergabeobjekt. Es ist keine ausgeführte Änderung.

---

## 13. Ablaufende

Der Workflow endet genau in einem der folgenden Zustände:

### `prepared`

Alle Prüfungen waren erfolgreich und ein Writer-Payload wurde im Modus `simulation` vorbereitet.

```text
decision.allowed = true
decision.code = WRITER_REQUEST_PREPARED
writer_request.prepared = true
```

### `rejected`

Eine fachliche, sicherheitsbezogene oder validierungsbedingte Prüfung ist fehlgeschlagen.

```text
decision.allowed = false
writer_request.prepared = false
```

### `failed`

Ein interner technischer Controller-Fehler hat eine kontrollierte Verarbeitung verhindert.

```text
decision.allowed = false
decision.code = INTERNAL_CONTROLLER_ERROR
writer_request.prepared = false
```

Nur der Zustand `prepared` darf einen vollständig validierten Writer-Payload enthalten.

Auch im Zustand `prepared` erfolgt kein Schreibzugriff auf GitHub.

Damit ist der Ablauf von WF-0013 beendet.