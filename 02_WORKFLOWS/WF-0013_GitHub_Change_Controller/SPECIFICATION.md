# WF-0013 – GitHub Change Controller

Version: `v0.1.0`
Status: `draft`
Typ: `Controller`
Betriebsmodus: `prepare-only`

---

## 1. Zweck

WF-0013 koordiniert kontrollierte Änderungen an freigegebenen GitHub-Dateien.

Der Workflow verbindet:

- `WF-0012 – GitHub Reader`
- `WF-0011 – GitHub Writer`

WF-0013 liest und schreibt nicht selbst direkt über die GitHub API.

Er prüft einen Änderungsauftrag gegen den aktuellen Dateistand und erzeugt daraus bei erfolgreicher Prüfung einen kontrollierten, noch nicht ausgeführten Schreibauftrag für WF-0011.

---

## 2. Verantwortlichkeit

WF-0013 ist verantwortlich für:

- Entgegennahme eines strukturierten Änderungsauftrags
- Validierung aller Pflichtfelder
- Prüfung des Ziel-Repositories gegen eine Allowlist
- Prüfung und Normalisierung des Dateipfads
- Vorbereitung eines Leseauftrags für WF-0012
- Prüfung des von WF-0012 gelieferten Dateistands
- Vergleich von erwartetem und tatsächlichem SHA
- Vergleich von aktuellem und vorgeschlagenem Inhalt
- Erkennung unveränderter oder widersprüchlicher Aufträge
- Prüfung einer ausdrücklichen Boolean-Freigabe
- Erzeugung eines kontrollierten Schreibauftrags
- Dokumentation der getroffenen Entscheidung
- sichere und bereinigte Ergebnis- und Fehlerausgabe

WF-0013 ist nicht verantwortlich für:

- direkten Lesezugriff auf GitHub
- direkten Schreibzugriff auf GitHub
- Verwaltung von GitHub-Credentials
- automatische Freigabe eines Schreibvorgangs
- automatische Ausführung von WF-0011
- Zusammenführung konkurrierender Änderungen
- Auflösung von Merge-Konflikten
- Erzeugung produktiver Writer-Modi
- Commit- oder Push-Ausführung

---

## 3. Betriebsmodus

WF-0013 arbeitet in `v0.1.0` ausschließlich im Modus:

```text
prepare-only
```

Dieser Modus bedeutet:

- Der Änderungsauftrag wird vollständig geprüft.
- Der aktuelle Dateistand wird über WF-0012 bezogen.
- Ein Writer-Payload kann vorbereitet werden.
- WF-0011 wird nicht automatisch ausgeführt.
- Es findet kein Schreibzugriff statt.
- Es wird kein Commit erstellt.
- Es wird kein Push ausgeführt.

Andere Betriebsmodi sind in `v0.1.0` unzulässig.

---

## 4. Eingang

WF-0013 akzeptiert genau ein JSON-Objekt pro Ausführung.

Das Eingangsobjekt besitzt folgende Hauptbereiche:

```text
execution
target
change
approval
request_id
```

Beispiel:

```json
{
  "execution": {
    "mode": "prepare-only"
  },
  "target": {
    "owner": "Jamoko112026",
    "repository": "JaMoKo_Automation_OS",
    "branch": "main",
    "path": "02_WORKFLOWS/example.md",
    "expected_sha": "EXPECTED_FILE_SHA"
  },
  "change": {
    "proposed_content": "Vollständiger neuer Dateiinhalt",
    "commit_message": "Update controlled workflow file"
  },
  "approval": {
    "approved": true
  },
  "request_id": "REQ-EXAMPLE-001"
}
```

---

## 5. Pflichtfelder

Folgende Felder sind verpflichtend:

```text
execution.mode
target.owner
target.repository
target.branch
target.path
target.expected_sha
change.proposed_content
change.commit_message
approval.approved
```

`request_id` darf in `v0.1.0` optional sein, sofern die Implementierung kontrolliert eine interne Kennung erzeugt.

Fehlende Pflichtfelder führen zu:

```text
INVALID_INPUT
```

---

## 6. Feldregeln

### 6.1 `execution.mode`

Zulässiger Wert:

```text
prepare-only
```

Datentyp:

```text
string
```

Andere Werte führen zu:

```text
INVALID_MODE
```

### 6.2 `target.owner`

Anforderungen:

- String
- nicht leer
- keine Steuerzeichen
- Bestandteil der Allowlist
- keine URL
- keine führenden oder nachgestellten Leerzeichen nach Normalisierung

### 6.3 `target.repository`

Anforderungen:

- String
- nicht leer
- Bestandteil der Allowlist
- keine URL
- keine Pfadbestandteile
- keine Steuerzeichen

### 6.4 `target.branch`

Anforderungen:

- String
- nicht leer
- Bestandteil der freigegebenen Zielkonfiguration
- keine Steuerzeichen
- kein ungeprüfter Branch-Wechsel

### 6.5 `target.path`

Anforderungen:

- relativer Repository-Pfad
- nicht leer
- keine absoluten Pfade
- keine Traversal-Sequenzen
- keine URL-Bestandteile
- keine Backslashes
- keine Steuerzeichen
- nur freigegebene Pfadbereiche
- nur unterstützte Dateiendungen

### 6.6 `target.expected_sha`

Anforderungen:

- String
- nicht leer
- entspricht dem erwarteten aktuellen Datei-SHA
- wird nicht stillschweigend ersetzt
- muss mit dem von WF-0012 gelieferten SHA übereinstimmen

### 6.7 `change.proposed_content`

Anforderungen:

- String
- vollständiger gewünschter Dateiinhalt
- innerhalb des konfigurierten Größenlimits
- keine Binärdaten
- nicht automatisch aus dem aktuellen Inhalt abgeleitet
- wird vor einer Writer-Vorbereitung mit dem aktuellen Inhalt verglichen

Ein leerer String darf nur zulässig sein, wenn das vollständige Leeren einer Datei durch einen später dokumentierten Standard ausdrücklich freigegeben wurde.

Bis dahin wird ein leerer Inhalt abgelehnt.

### 6.8 `change.commit_message`

Anforderungen:

- String
- nicht leer
- innerhalb der festgelegten Maximallänge
- keine Zeilenumbrüche, sofern WF-0011 diese nicht ausdrücklich unterstützt
- keine Steuerzeichen
- keine Tokens oder Zugangsdaten
- kompatibel mit dem Vertrag von WF-0011

### 6.9 `approval.approved`

Zulässiger Wert:

```json
true
```

Zulässiger Datentyp:

```text
boolean
```

Folgende Werte gelten nicht als Freigabe:

```text
"true"
1
"1"
"yes"
"ja"
null
```

---

### Verbindliche technische Grenzen

Für WF-0013 v0.1.0 gelten folgende Grenzen und Formate:

- `target.expected_sha` muss dem Muster `^[a-fA-F0-9]{40}$` entsprechen.
- `change.proposed_content` darf maximal `100000` UTF-8-Bytes umfassen.
- Ein leerer `proposed_content`-String ist zulässig.
- `proposed_content` darf keine Nullzeichen enthalten.
- `proposed_content` wird weder getrimmt noch inhaltlich normalisiert.
- `change.commit_message` darf maximal 120 Zeichen umfassen.
- `change.commit_message` muss nach kontrollierter Trimmung mindestens ein sichtbares Zeichen enthalten.
- `change.commit_message` muss einzeilig und frei von Null- und Steuerzeichen sein.
- In v0.1.0 werden ausschließlich Dateien mit der Endung `.md` unterstützt.
- Unbekannte Felder sind auf jeder Ebene des Eingangsobjekts unzulässig.

Eine vorhandene externe `request_id` muss dem folgenden Muster entsprechen:

```regex
^REQ-[A-Z0-9][A-Z0-9-]{7,63}$
```

Fehlt die externe `request_id`, darf WF-0013 eine interne Korrelationskennung erzeugen. Diese interne Kennung dient ausschließlich der Verarbeitung und Nachvollziehbarkeit innerhalb von WF-0013. Sie darf nicht als `request_id` an WF-0011 weitergereicht werden.

Die konkrete Kombination aus Owner, Repository, Branch und freigegebenem Pfadbereich wird durch die Allowlist zur Laufzeit geprüft. Sie ist nicht Bestandteil des statischen Eingangsschemas.

---

## 7. Normalisierung

Vor der fachlichen Prüfung darf WF-0013 kontrollierte Normalisierungen durchführen.

Zulässig sind:

- Entfernen äußerer Leerzeichen bei definierten Textfeldern
- Vereinheitlichung eindeutig dokumentierter Feldformen
- Erzeugung einer internen `request_id`, wenn keine vorhanden ist
- kanonische Darstellung des Zielobjekts

Nicht zulässig sind:

- automatische Änderung des Owners
- automatische Änderung des Repositories
- automatische Änderung des Branches
- automatische Korrektur eines unsicheren Pfads
- automatische Ersetzung des erwarteten SHA
- automatische Erteilung einer Freigabe
- automatische Veränderung des vorgeschlagenen Inhalts
- Erfindung einer Commit-Nachricht

Die Normalisierung darf die fachliche Bedeutung des Auftrags nicht verändern.

---

## 8. Allowlist

WF-0013 verarbeitet ausschließlich ausdrücklich freigegebene Ziele.

Die Allowlist muss mindestens folgende Ebenen abbilden können:

```text
owner
repository
branch
path_prefix
file_extension
```

Ein Ziel ist nur zulässig, wenn alle konfigurierten Bedingungen erfüllt sind.

Nicht freigegebene Ziele führen zu:

```text
TARGET_NOT_ALLOWED
```

Die Allowlist darf nicht aus ungeprüften Eingabedaten erweitert werden.

---

## 9. Pfadprüfung

Der Dateipfad wird vor jeder Reader-Verarbeitung geprüft.

Unzulässig sind insbesondere:

```text
..
./
../
\
://
NUL
Steuerzeichen
absolute Pfade
leere Pfadsegmente
```

Der Pfad muss nach der Normalisierung weiterhin innerhalb des freigegebenen Repository-Bereichs liegen.

Ein unsicherer oder ungültiger Pfad führt zu:

```text
INVALID_PATH
```

WF-0013 versucht nicht, einen unsicheren Pfad automatisch zu reparieren.

---

## 10. Reader-Auftrag

WF-0013 erzeugt für WF-0012 einen minimalen Leseauftrag.

Er enthält ausschließlich die erforderlichen Felder:

```json
{
  "request_id": "REQ-EXAMPLE-001",
  "owner": "Jamoko112026",
  "repository": "JaMoKo_Automation_OS",
  "branch": "main",
  "path": "02_WORKFLOWS/example.md"
}
```

Der Reader-Auftrag darf nicht enthalten:

- GitHub-Token
- Authorization-Header
- Writer-Payload
- Freigabedaten
- Commit-Nachricht
- ungeprüfte Zusatzfelder

Ein intern ungültiger Reader-Auftrag führt zu:

```text
READER_REQUEST_INVALID
```

---

## 11. Reader-Ergebnis

WF-0012 muss für einen erfolgreichen Vergleich mindestens liefern:

```text
status
mode
source.owner
source.repository
source.ref
file.sha
file.path
file.content
file.encoding
error
```

WF-0013 prüft:

- erfolgreichen Reader-Status
- interne Zuordnung zum ursprünglichen Änderungsauftrag
- identischen Owner
- identisches Repository
- identischen Branch
- identischen Dateipfad
- vorhandenen Datei-SHA
- vorhandenen Dateiinhalt
- exakt `base64` als ursprüngliche GitHub-Kodierung
- widerspruchsfreie Status- und Fehlerfelder
- Abwesenheit unzulässiger sensibler Daten

WF-0012 v0.1.0 unterstützt keine `request_id`.

WF-0013 muss die `request_id` des ursprünglichen Änderungsauftrags während des Reader-Auftrags intern erhalten und das Reader-Ergebnis eindeutig diesem Auftrag zuordnen. Eine `request_id` darf nicht an WF-0012 übergeben oder von dessen Ergebnis erwartet werden.

Ein technischer Reader-Fehler führt zu:

```text
READER_FAILED
```

Ein unvollständiges oder widersprüchliches Reader-Ergebnis führt zu:

```text
READER_RESULT_INVALID
```

Eine Abweichung des Ziels führt zu:

```text
TARGET_MISMATCH
```

---

## 12. SHA-Prüfung

WF-0013 verwendet optimistische Nebenläufigkeitskontrolle.

Verglichen werden:

```text
target.expected_sha
reader_result.file.sha
```

Beide Werte müssen exakt übereinstimmen.

Bei einer Abweichung gilt:

```text
status = rejected
decision.code = SHA_CONFLICT
writer_request.prepared = false
```

WF-0013 darf den aktuellen SHA nicht automatisch als neuen erwarteten SHA übernehmen.

---

## 13. Inhaltsvergleich

WF-0013 vergleicht:

```text
change.proposed_content
reader_result.file.content
```

Sind beide Inhalte identisch, liegt keine Änderung vor.

Das Ergebnis lautet:

```text
NO_CHANGE
```

In diesem Fall:

- wird kein Writer-Payload vorbereitet
- wird WF-0011 nicht aufgerufen
- bleibt `write_executed` auf `false`

Der Inhaltsvergleich muss deterministisch sein.

Eine mögliche Normalisierung von Zeilenenden muss vor der Implementierung ausdrücklich festgelegt und dokumentiert werden.

---

## 14. Freigabeprüfung

Die Freigabe wird erst ausgewertet, nachdem:

- Eingangsstruktur
- Betriebsmodus
- Ziel
- Pfad
- Änderungsinhalt
- Reader-Auftrag
- Reader-Ergebnis
- Zielübereinstimmung
- SHA
- Inhaltsänderung

erfolgreich geprüft wurden.

Nur folgender Wert gilt als gültige Freigabe:

```json
{
  "approved": true
}
```

Eine fehlende oder ungültige Freigabe führt zu:

```text
INVALID_APPROVAL
```

Eine gültige Freigabe hebt keine andere Ablehnung auf.

---

## 15. Writer-Auftrag

Nur bei vollständigem Erfolg erzeugt WF-0013 einen Writer-Auftrag für WF-0011.

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
    "path": "02_WORKFLOWS/example.md"
  },
  "source": {
    "expected_sha": "CURRENT_FILE_SHA",
    "controller": "WF-0013",
    "controller_version": "v0.1.0"
  },
  "change": {
    "content": "Vollständiger neuer Dateiinhalt",
    "commit_message": "Update controlled workflow file"
  },
  "request_id": "REQ-EXAMPLE-001"
}
```

Der konkrete Wert von `execution.mode` wird in `v0.1.0` fest auf `simulation` gesetzt und muss dem freigegebenen Vertrag von WF-0011 v0.2.0 entsprechen.

WF-0013 darf:

- keinen produktiven Writer-Modus erfinden
- keinen ungeprüften Modus aus dem Eingang übernehmen
- WF-0011 nicht automatisch ausführen
- keine Writer-Credentials ergänzen

Ein ungültiger Writer-Auftrag führt zu:

```text
WRITER_REQUEST_INVALID
```

---

## 16. Entscheidungsreihenfolge

Die Prüfungen erfolgen in dieser Priorität:

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

Bei mehreren möglichen Fehlern wird der zuerst erreichte Fehler gemäß dieser Reihenfolge ausgegeben.

Dadurch bleibt das Ergebnis deterministisch.

---

## 17. Ausgang

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

---

## 18. Erfolgreiches Ergebnis

Ein erfolgreich vorbereitetes Ergebnis besitzt:

```text
status = prepared
decision.allowed = true
decision.code = WRITER_REQUEST_PREPARED
writer_request.prepared = true
execution.writer_called = false
execution.write_executed = false
execution.commit_created = false
execution.push_executed = false
error = null
```

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
    "path": "02_WORKFLOWS/example.md"
  },
  "comparison": {
    "sha_matches": true,
    "content_changed": true
  },
  "approval": {
    "approved": true
  },
  "writer_request": {
    "prepared": true,
    "payload": {
      "execution": {
        "mode": "simulation"
      },
      "target": {
        "owner": "Jamoko112026",
        "repository": "JaMoKo_Automation_OS",
        "branch": "main",
        "path": "02_WORKFLOWS/example.md"
      },
      "source": {
        "expected_sha": "CURRENT_FILE_SHA",
        "controller": "WF-0013",
        "controller_version": "v0.1.0"
      },
      "change": {
        "content": "Vollständiger neuer Dateiinhalt",
        "commit_message": "Update controlled workflow file"
      },
      "request_id": "REQ-EXAMPLE-001"
    }
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

---

## 19. Abgelehntes Ergebnis

Eine fachliche Ablehnung besitzt:

```text
status = rejected
decision.allowed = false
writer_request.prepared = false
execution.writer_called = false
execution.write_executed = false
execution.commit_created = false
execution.push_executed = false
```

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
  "target": {
    "owner": "Jamoko112026",
    "repository": "JaMoKo_Automation_OS",
    "branch": "main",
    "path": "02_WORKFLOWS/example.md"
  },
  "comparison": {
    "sha_matches": false,
    "content_changed": null
  },
  "approval": {
    "approved": true
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

---

## 20. Fehlgeschlagenes Ergebnis

Ein interner technischer Fehler besitzt:

```text
status = failed
decision.allowed = false
decision.code = INTERNAL_CONTROLLER_ERROR
writer_request.prepared = false
```

Interne Stack-Traces, Credentials und ungefilterte technische Antworten dürfen nicht ausgegeben werden.

---

## 21. Fehlercodes

WF-0013 verwendet mindestens folgende stabilen Fehler- und Entscheidungscodes:

```text
INVALID_INPUT
INVALID_MODE
TARGET_NOT_ALLOWED
INVALID_PATH
INVALID_CHANGE
CONTENT_TOO_LARGE
READER_REQUEST_INVALID
READER_FAILED
READER_RESULT_INVALID
TARGET_MISMATCH
SHA_CONFLICT
NO_CHANGE
INVALID_APPROVAL
WRITER_REQUEST_INVALID
INTERNAL_CONTROLLER_ERROR
WRITER_REQUEST_PREPARED
```

Fehlercodes sind maschinenlesbar und stabil zu halten.

Fehlermeldungen sind für Menschen verständlich, enthalten aber keine sensiblen Details.

---

## 22. Sicherheitsanforderungen

WF-0013 muss folgende Sicherheitsanforderungen erfüllen:

1. Keine direkte GitHub-Leseoperation.
2. Keine direkte GitHub-Schreiboperation.
3. Keine automatische Ausführung von WF-0011.
4. Keine Ausgabe von Tokens oder Credentials.
5. Keine Ausgabe vollständiger HTTP-Header.
6. Keine Ausgabe interner Stack-Traces.
7. Keine ungeprüfte Übernahme von Zielangaben.
8. Keine automatische Konfliktauflösung.
9. Keine automatische Freigabe.
10. Keine Umgehung der Allowlist.
11. Keine Verarbeitung unsicherer Pfade.
12. Keine Spiegelung vollständiger Dateiinhalte in Fehlerobjekten.
13. Keine produktiven Seiteneffekte im Modus `prepare-only`.

---

## 23. Seiteneffektfreiheit

Für jede Ausführung von `v0.1.0` müssen gelten:

```text
writer_called = false
write_executed = false
commit_created = false
push_executed = false
```

Diese Werte gelten auch bei:

- erfolgreicher Vorbereitung
- fachlicher Ablehnung
- Reader-Fehler
- internem Controller-Fehler

Ein abweichender Wert ist ein kritischer Sicherheitsfehler.

---

## 24. Protokollierung

Protokolliert werden dürfen:

- Workflow-ID
- Workflow-Version
- Betriebsmodus
- `request_id`
- bereinigtes Ziel
- Entscheidungscode
- Ergebnisstatus
- Vergleichsergebnisse als Boolean-Werte
- Zeitstempel
- kontrollierte technische Metadaten

Nicht protokolliert werden dürfen:

- Tokens
- Zugangsdaten
- Authorization-Header
- vollständige Reader-Rohantworten
- vollständige Datei- oder Änderungsinhalte
- interne Stack-Traces
- n8n-Credentials

---

## 25. Nicht unterstützte Funktionen

In `v0.1.0` werden nicht unterstützt:

- automatische Writer-Ausführung
- produktive Schreibvorgänge
- mehrere Dateien pro Auftrag
- Verzeichnisänderungen
- Datei-Löschung
- Datei-Verschiebung
- Datei-Umbenennung
- Binärdateien
- automatische Zusammenführung von Änderungen
- Merge-Konflikt-Auflösung
- Pull-Request-Erstellung
- Branch-Erstellung
- Tag-Erstellung
- Release-Erstellung
- Repository-übergreifende Sammeländerungen

---

## 26. Abnahmekriterien

WF-0013 gilt auf Spezifikationsebene als abnahmefähig, wenn:

1. genau ein Änderungsauftrag verarbeitet wird,
2. alle Pflichtfelder geprüft werden,
3. ausschließlich `prepare-only` akzeptiert wird,
4. nicht freigegebene Ziele abgelehnt werden,
5. unsichere Pfade abgelehnt werden,
6. der aktuelle Zustand ausschließlich über WF-0012 bezogen wird,
7. Reader-Ergebnisse vollständig validiert werden,
8. Zielabweichungen erkannt werden,
9. SHA-Konflikte erkannt werden,
10. unveränderte Inhalte erkannt werden,
11. ausschließlich Boolean `true` als Freigabe gilt,
12. ein Writer-Payload nur bei vollständigem Erfolg entsteht,
13. WF-0011 niemals automatisch ausgeführt wird,
14. sämtliche Ausgaben bereinigt werden,
15. alle Seiteneffekt-Flags immer `false` bleiben.

---

## 27. Versionsgrenze

Diese Spezifikation gilt ausschließlich für:

```text
WF-0013 v0.1.0
```

Verbindlicher Betriebsmodus:

```text
prepare-only
```

Folgende Änderungen erfordern mindestens eine neue Version und eine Aktualisierung aller betroffenen Dokumente:

- automatische Ausführung von WF-0011
- produktiver Schreibmodus
- veränderter Reader-Vertrag
- veränderter Writer-Vertrag
- Verarbeitung mehrerer Dateien
- Unterstützung weiterer Änderungsarten
- Änderung der Freigabelogik
- Änderung der Fehlerpriorität
- Änderung der Sicherheitsgrenzen

---

## 28. Endbedingung

WF-0013 endet immer mit genau einem bereinigten Ergebnisobjekt.

Das einzige positive Endergebnis von `v0.1.0` ist:

```text
WRITER_REQUEST_PREPARED
```

Dieses Ergebnis bestätigt ausschließlich, dass ein Writer-Auftrag vorbereitet wurde.

Es bestätigt keinen Schreibvorgang, keinen Commit und keinen Push.
