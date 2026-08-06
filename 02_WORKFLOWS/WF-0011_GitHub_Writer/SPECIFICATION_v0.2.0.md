# WF-0011 – Specification v0.2.0

## Version

0.2.0

## Status

draft

## Betriebsmodus

simulation

---

## 1. Zweck

WF-0011 v0.2.0 validiert und simuliert vollständige GitHub-Dateiänderungen, die zuvor durch WF-0013 vorbereitet wurden.

Der Workflow:

1. übernimmt genau einen Writer-Auftrag,
2. validiert dessen Struktur,
3. prüft Ziel, Referenz und Dateipfad,
4. prüft Herkunft und Vorbereitungsstatus,
5. prüft den erwarteten Ausgangs-SHA,
6. validiert Dateiinhalt und Commit-Nachricht,
7. erzeugt eine kanonische Änderungsvorschau,
8. erzeugt einen validierten, nicht angewendeten Patch,
9. gibt genau ein bereinigtes Simulationsergebnis aus.

WF-0011 v0.2.0 führt keine tatsächliche GitHub-Änderung aus.

---

## 2. Verhältnis zu v0.1.0

WF-0011 v0.1.0 verarbeitet einen fachlichen Objektfeld-Auftrag:

```text
objectId
field
currentValue
approvedValue
```

WF-0011 v0.2.0 verarbeitet dagegen einen vollständigen, bereits vorbereiteten Dateiänderungsauftrag:

```text
execution
target
source
change
request_id
```

Die beiden Versionen besitzen unterschiedliche Eingabemodelle.

Ein Auftrag für v0.1.0 ist nicht automatisch mit v0.2.0 kompatibel.

WF-0011 v0.2.0 ersetzt v0.1.0 nicht produktiv. Die Version dient ausschließlich der kontrollierten Entwicklung und Simulation des neuen Writer-Modells.

---

## 3. Systemkontext

WF-0011 v0.2.0 ist Teil der kontrollierten GitHub-Änderungskette:

```text
Fachlicher Änderungsbedarf
        |
        v
WF-0013 – GitHub Change Controller
        |
        | vorbereiteter Writer-Auftrag
        | manuelle Übergabe
        v
WF-0011 v0.2.0 – GitHub Writer Simulation
        |
        v
Bereinigtes Simulationsergebnis
```

WF-0011:

* startet WF-0013 nicht,
* startet keinen anderen Workflow,
* ruft GitHub nicht auf,
* liest keine Datei aus GitHub,
* schreibt keine Datei,
* erstellt keinen Commit,
* führt keinen Push aus.

Die Übergabe eines Auftrags erfolgt im Simulationsbetrieb ausschließlich kontrolliert und manuell.

---

## 4. Verantwortungsgrenze

WF-0011 v0.2.0 ist verantwortlich für:

* Eingangsbegrenzung auf genau einen Auftrag,
* strukturelle Eingangsvalidierung,
* Validierung des Simulationsmodus,
* Prüfung der Controller-Herkunft,
* Prüfung des Vorbereitungsstatus,
* Prüfung des Auditstatus,
* Prüfung des Ziel-Repositorys,
* sichere Validierung und Normalisierung des Dateipfads,
* Validierung der Git-Referenz,
* Validierung des erwarteten Ausgangs-SHA,
* Validierung des vollständigen Zielinhalts,
* Validierung der Commit-Nachricht,
* Erzeugung eines kanonischen internen Auftrags,
* Simulation eines nicht angewendeten Patches,
* Validierung der Patch-Metadaten,
* deterministische Fehlerentscheidung,
* Bereinigung des Endergebnisses,
* Erzwingung sicherer Seiteneffekt-Flags.

WF-0011 v0.2.0 ist nicht verantwortlich für:

* fachliche Freigabe einer Änderung,
* Erzeugung oder Beschaffung realer Zugangsdaten,
* Prüfung des tatsächlichen GitHub-Dateistands,
* Abruf eines aktuellen SHA aus GitHub,
* Anwendung eines Patches,
* Schreiben oder Löschen einer Datei,
* Erstellen eines Branches,
* Erstellen eines Commits,
* Pushen einer Änderung,
* Erstellen eines Pull Requests,
* Aktivierung weiterer Workflows,
* produktive Veröffentlichung.

---

## 5. Verbindliche Betriebsregeln

Für v0.2.0 gelten folgende Regeln:

1. Der Betriebsmodus ist immer `simulation`.
2. Der Workflow bleibt in n8n inaktiv.
3. Automatische Trigger sind verboten.
4. Pro Ausführung ist genau ein Eingangs-Item zulässig.
5. Eingaben gelten grundsätzlich als nicht vertrauenswürdig.
6. Fehlende Werte werden nicht automatisch aus externen Quellen ergänzt.
7. Es werden keine zufälligen oder zeitabhängigen Werte erzeugt.
8. Eine fehlende `request_id` wird nicht automatisch ersetzt.
9. Es dürfen keine Credentials eingebunden sein.
10. Es dürfen keine Netzwerk- oder GitHub-Aufrufe erfolgen.
11. Es dürfen keine Shell- oder Systembefehle ausgeführt werden.
12. Es dürfen keine Dateioperationen erfolgen.
13. Kein anderer Workflow darf automatisch gestartet werden.
14. Alle Erfolgs- und Fehlerwege müssen den Output Sanitizer durchlaufen.
15. Jede Ausführung erzeugt genau ein End-Item.
16. Sicherheitsrelevante Flags werden im Endergebnis immer auf `false` gesetzt.
17. Im Zweifel wird der Auftrag abgelehnt.

---

## 6. Erwartetes Eingangsmodell

Der Workflow erwartet genau ein JSON-Objekt.

Verbindliche Hauptbereiche:

```text
execution
target
source
change
```

Optional:

```text
request_id
```

Der Auftrag muss fachlich diesem Modell entsprechen:

```json
{
  "request_id": "REQ-WF0011-TEST-001",
  "execution": {
    "mode": "simulation"
  },
  "target": {
    "owner": "Jamoko112026",
    "repository": "JaMoKo_Automation_OS",
    "ref": "main",
    "path": "02_WORKFLOWS/WF-0011_GitHub_Writer/README.md"
  },
  "source": {
    "controller_workflow": "WF-0013",
    "controller_status": "prepared",
    "audit_status": "passed",
    "approved_by": "manual_review",
    "expected_sha": "0123456789abcdef0123456789abcdef01234567"
  },
  "change": {
    "content": "# Controlled test content\n",
    "commit_message": "Update controlled test file"
  }
}
```

Die konkreten zulässigen Owner-Repository-Paare werden bei der Implementierung als feste Allowlist dokumentiert.

Unbekannte Eingangsfelder dürfen die Entscheidung nicht unsicher machen und werden nicht in das öffentliche Endergebnis übernommen.

---

## 7. Pflichtfelder

Folgende Felder sind verpflichtend:

```text
execution.mode
target.owner
target.repository
target.ref
target.path
source.controller_workflow
source.controller_status
source.audit_status
source.approved_by
source.expected_sha
change.content
change.commit_message
```

`request_id` ist optional.

`source.approved_by` darf intern vorhanden sein, wird aber nicht öffentlich ausgegeben.

Fehlt ein Pflichtfeld, wird der Auftrag mit folgendem Fehlercode abgelehnt:

```text
MISSING_REQUIRED_FIELD
```

Fehlende Pflichtfelder werden nicht automatisch ergänzt.

---

## 8. Erwartete Datentypen

Folgende Datentypen sind verbindlich:

```text
execution                    object
execution.mode               string
request_id                   string, falls vorhanden
target                       object
target.owner                 string
target.repository            string
target.ref                   string
target.path                  string
source                       object
source.controller_workflow   string
source.controller_status     string
source.audit_status          string
source.approved_by           string
source.expected_sha          string
change                       object
change.content               string
change.commit_message        string
```

Arrays, Nullwerte, Zahlen, boolesche Werte oder Objekte anstelle der erwarteten Zeichenketten sind nicht zulässig.

Ein falscher Datentyp führt zu:

```text
INVALID_FIELD_TYPE
```

---

## 9. Eingangsbegrenzung

WF-0011 verarbeitet genau ein Eingangs-Item.

Folgende Fälle werden abgelehnt:

* kein Eingangs-Item,
* zwei oder mehr Eingangs-Items,
* ein nicht als Objekt verwertbarer Eingang.

Verbindlicher Fehlercode:

```text
INVALID_INPUT_COUNT
```

Mehrere Eingangs-Items dürfen weder zusammengeführt noch einzeln weiterverarbeitet werden.

---

## 10. Modusprüfung

Der einzige zulässige Wert ist:

```text
execution.mode = simulation
```

Andere Werte, Groß-/Kleinschreibungsvarianten oder fehlende Werte sind nicht zulässig.

Beispiele für unzulässige Werte:

```text
live
write
production
apply
Simulation
```

Verbindlicher Fehlercode:

```text
INVALID_MODE
```

Der Workflow darf den Modus nicht automatisch korrigieren.

---

## 11. Herkunfts- und Vorbereitungsprüfung

Der Auftrag muss durch WF-0013 vorbereitet und manuell freigegeben worden sein.

Verbindliche Werte:

```text
source.controller_workflow = WF-0013
source.controller_status = prepared
source.audit_status = passed
source.approved_by = manual_review

---

## 12. Ziel-Allowlist

Nur ausdrücklich freigegebene Kombinationen aus Owner und Repository sind zulässig.

Die Prüfung erfolgt als festes Paar:

```text
target.owner + target.repository
```

Die unabhängige Freigabe eines Owners und eines Repositorynamens reicht nicht aus.

Nicht freigegebene oder mehrdeutige Kombinationen führen zu:

```text
TARGET_NOT_ALLOWED
```

Die Allowlist:

* ist statisch,
* wird nicht aus dem Eingang erweitert,
* wird nicht aus externen Quellen geladen,
* enthält keine Credentials,
* wird in der Implementierungsdokumentation nachvollziehbar festgehalten.

---

## 13. Pfadsicherheit

`target.path` muss einen eindeutigen, relativen Repository-Dateipfad darstellen.

Vor jeder weiteren Verwendung wird der Pfad kontrolliert normalisiert und anschließend erneut validiert.

Unzulässig sind insbesondere:

* absolute Pfade,
* Pfade mit `..`,
* Pfade mit `.` als Segment,
* führende Schrägstriche,
* Backslashes,
* doppelte Schrägstriche,
* leere Segmente,
* Nullbytes,
* URL-Schemata,
* Home-Verweise,
* Laufwerksbuchstaben,
* Query- oder Fragmentbestandteile,
* Pfade mit Steuerzeichen,
* Pfade, deren normalisierte Form vom sicher erwarteten Ziel abweicht,
* leere Pfade,
* reine Verzeichnispfade.

Abzulehnende Beispiele:

```text
../secret.txt
folder/../secret.txt
./file.md
/file.md
folder//file.md
folder\file.md
C:\file.md
~/file.md
https://example.test/file.md
file.md?raw=true
file.md#section
```

Zulässiges Beispiel:

```text
02_WORKFLOWS/WF-0011_GitHub_Writer/README.md
```

Ein unsicherer oder ungültiger Pfad führt zu:

```text
INVALID_PATH
```

Der normalisierte Pfad darf nur intern verwendet werden und muss mit dem späteren Patch-Ziel übereinstimmen.

---

## 14. Referenzprüfung

`target.ref` muss eine ausdrücklich zulässige Git-Referenz enthalten.

Für den ersten Simulationsstand ist ausschließlich die dokumentierte Referenz zulässig, standardmäßig:

```text
main
```

Unzulässig sind insbesondere:

* leere Werte,
* nicht freigegebene Branches,
* Referenzen mit Steuerzeichen,
* mehrdeutige oder zusammengesetzte Refs,
* Werte, die wie Befehle oder URLs aufgebaut sind.

Fehlercode:

```text
INVALID_REF
```

WF-0011 erstellt, verändert oder prüft keinen realen Branch.

---

## 15. Prüfung des erwarteten Ausgangs-SHA

`source.expected_sha` bezeichnet den durch WF-0013 bestätigten erwarteten Ausgangsstand.

Der SHA:

* muss vorhanden sein,
* muss eine Zeichenkette sein,
* muss exakt 40 hexadezimale Zeichen enthalten,
* darf keine Leerzeichen oder Präfixe enthalten,
* wird nicht über GitHub aktualisiert,
* wird nicht als Nachweis eines tatsächlichen Dateistands interpretiert.

Fehlt `source.expected_sha` vollständig, gilt:

```text
MISSING_REQUIRED_FIELD
```

Ist `source.expected_sha` vorhanden, aber leer, gilt:

```text
SOURCE_SHA_MISSING
```

Ist `source.expected_sha` vorhanden, aber entspricht nicht dem vorgeschriebenen Format, gilt:

```text
INVALID_SOURCE_SHA
```

Zulässig ist ausschließlich ein exakt 40-stelliger hexadezimaler SHA:

```text
0123456789abcdef0123456789abcdef01234567
```

Dabei sind ausschließlich folgende Zeichen zulässig:

```text
0–9
a–f
A–F
```

Die SHA-Prüfung darf keinen GitHub-Aufruf auslösen.

WF-0011 vergleicht ausschließlich den im vorbereiteten Auftrag enthaltenen Wert.

Der SHA darf nicht verändert, ergänzt oder automatisch ermittelt werden.

---

## 16. Inhaltsprüfung

`change.content` enthält den vollständigen vorgesehenen Zielinhalt der Datei.

Der Inhalt:

- muss als Zeichenkette vorliegen,
- darf leer sein,
- darf nicht `null` sein,
- wird als UTF-8 verarbeitet,
- darf maximal `100000` UTF-8-Bytes umfassen,
- darf nicht als Binär-, Credential- oder Secret-Transport verwendet werden.

Ein inhaltlich oder technisch unzulässiger Wert führt zu:

```text
INVALID_CONTENT
```

Eine Überschreitung des verbindlichen Byte-Limits führt zu:

```text
CONTENT_TOO_LARGE
```

Die Byte-Länge wird deterministisch auf Grundlage des UTF-8-Inhalts ermittelt.

Das konkrete Größenlimit muss in FLOW, TESTS und Implementierung identisch definiert sein.

Der Dateiinhalt darf nicht in Fehlermeldungen oder im bereinigten Endergebnis erscheinen.

---

## 17. Commit-Nachrichten-Prüfung

`change.commit_message` muss eine kontrollierte Commit-Nachricht enthalten.

Die Nachricht:

- muss eine Zeichenkette sein,
- darf nach Trimmung nicht leer sein,
- darf maximal 120 Zeichen lang sein,
- muss einzeilig sein,
- darf keine Steuerzeichen enthalten,
- darf keine Credentials, Tokens oder geheimen Werte transportieren,
- darf nicht automatisch aus dem Dateiinhalt erzeugt werden.

Ungültige Nachrichten führen zu:

```text
INVALID_COMMIT_MESSAGE
```

WF-0011 simuliert ausschließlich die Verwendung der Commit-Nachricht.

Die Commit-Nachricht wird intern nicht aus dem Dateiinhalt abgeleitet, ergänzt oder automatisch erzeugt.

Es wird kein Commit erstellt.

---

## 18. Request-ID

`request_id` ist optional.

Falls vorhanden, muss sie:

* eine Zeichenkette sein,
* dem dokumentierten Format entsprechen,
* frei von Steuerzeichen sein,
* innerhalb des dokumentierten Längenlimits liegen.

Eine fehlende `request_id` führt nicht zur Ablehnung.

WF-0011 darf keine eigene Request-ID erzeugen.

Bei fehlender `request_id` bleibt das Feld im Endergebnis vollständig aus.

Dadurch bleibt das Ergebnis deterministisch.

---

## 19. Fehlerpriorität

Wenn mehrere Fehler gleichzeitig vorliegen, wird genau ein öffentlicher Fehlercode ausgegeben.

Die verbindliche Fehlerpriorität wird zentral in `FLOW_v0.2.0.md` definiert und muss technisch exakt umgesetzt werden.

Sie beginnt mit der frühesten sicheren Prüfung:

```text
1. INVALID_INPUT_COUNT
2. MISSING_REQUIRED_FIELD
3. INVALID_FIELD_TYPE
4. INVALID_MODE
5. INVALID_CONTROLLER_SOURCE
6. CONTROLLER_NOT_PREPARED
7. AUDIT_NOT_PASSED
8. TARGET_NOT_ALLOWED
9. INVALID_PATH
10. INVALID_REF
11. SOURCE_SHA_MISSING
12. INVALID_SOURCE_SHA
13. INVALID_CONTENT
14. CONTENT_TOO_LARGE
15. INVALID_COMMIT_MESSAGE
16. PATCH_VALIDATION_FAILED
17. INTERNAL_ERROR
```

Bei mehreren Fehlern wird immer der höchstpriorisierte Fehler ausgegeben.

Fehlerlisten, Nebendiagnosen und rohe Prüfobjekte bleiben intern und werden nicht öffentlich ausgegeben.

---

## 20. Kanonischer Auftrag

Nach erfolgreicher Eingangs-, Schema- und Sicherheitsprüfung wird intern ein kanonischer Auftrag erzeugt.

Er enthält ausschließlich normalisierte und für die Simulation erforderliche Felder.

Beispielstruktur:

```json
{
  "request_id": "REQ-WF0011-TEST-001",
  "execution": {
    "mode": "simulation"
  },
  "target": {
    "owner": "Jamoko112026",
    "repository": "JaMoKo_Automation_OS",
    "ref": "main",
    "path": "02_WORKFLOWS/WF-0011_GitHub_Writer/README.md"
  },
  "source": {
    "controller_workflow": "WF-0013",
    "controller_status": "prepared",
    "audit_status": "passed",
    "approved_by": "manual_review",
    "expected_sha": "0123456789abcdef0123456789abcdef01234567"
  },
  "change": {
    "content": "# Controlled test content\n",
    "content_bytes": 26,
    "commit_message": "Update controlled test file"
  }
}
```

Nicht erforderliche und unbekannte Eingangsfelder werden nicht übernommen.

Der kanonische Auftrag ist ein internes Prüfobjekt und darf nicht vollständig im Endergebnis erscheinen.

---

## 21. Patch-Simulation

Aus dem kanonischen Auftrag wird intern ein nicht angewendetes Patch-Objekt erzeugt.

Das Patch-Objekt beschreibt ausschließlich die beabsichtigte Änderung.

Verbindliche Eigenschaften:

```text
target_owner
target_repository
target_ref
target_path
expected_sha
content_encoding
content_bytes
commit_message
applied
```

Dabei gilt zwingend:

```text
content_encoding = utf-8
applied = false
```

`content_bytes` enthält die deterministisch ermittelte Byte-Länge des UTF-8-Inhalts.

Das Patch-Objekt darf keine Felder enthalten, die einen realen Schreibvorgang behaupten oder dokumentieren.

Insbesondere verboten sind:

```text
write_status
commit_id
push_result
credential
credentials
token
secret
authorization
```

Die Patch-Simulation:

- schreibt keine Datei,
- ruft GitHub nicht auf,
- erstellt keinen Commit,
- führt keinen Push aus,
- startet keinen anderen Workflow.

---

## 22. Patch-Validierung

Das simulierte Patch-Objekt wird vor dem Erfolgsweg erneut validiert.

Geprüft werden mindestens:

* Ziel-Owner entspricht dem kanonischen Auftrag,
* Ziel-Repository entspricht dem kanonischen Auftrag,
* Ziel-Referenz entspricht dem kanonischen Auftrag,
* Zielpfad entspricht dem normalisierten kanonischen Pfad,
* erwarteter SHA stimmt überein,
* Inhaltskodierung lautet `utf-8`,
* Byte-Länge stimmt mit dem Inhalt überein,
* Commit-Nachricht stimmt überein,
* `applied` ist exakt `false`,
* keine Schreibstatusfelder sind vorhanden,
* keine Commit-ID ist vorhanden,
* kein Push-Ergebnis ist vorhanden,
* keine Credential- oder Authorization-Daten sind vorhanden.

Jede Abweichung führt zu:

```text
PATCH_VALIDATION_FAILED
```

Ein abgelehnter Patch darf nicht an den Success Builder weitergegeben werden.

---

## 23. Erfolgsergebnis

Ein gültiger Auftrag erzeugt ein bereinigtes Ergebnis mit:

```text
workflow_id
version
mode
status
request_id, falls vorhanden
target
source
simulation
file_changed
commit_created
push_executed
write_executed
```

Verbindliche Werte:

```json
{
  "workflow_id": "WF-0011",
  "version": "0.2.0",
  "mode": "simulation",
  "status": "simulated",
  "file_changed": false,
  "commit_created": false,
  "push_executed": false,
  "write_executed": false
}
```

Die genaue zulässige Unterstruktur von `target` und `simulation` wird in `FLOW_v0.2.0.md` festgelegt.

Das Erfolgsergebnis darf keinen vollständigen Dateiinhalt enthalten.

---

## 24. Ablehnungsergebnis

Ein fachlich oder sicherheitstechnisch ungültiger Auftrag erzeugt ausschließlich:

```text
workflow_id
version
mode
status
request_id, falls sicher vorhanden
error_code
message
file_changed
commit_created
push_executed
write_executed
```

Verbindliche Sicherheitswerte:

```json
{
  "workflow_id": "WF-0011",
  "version": "0.2.0",
  "mode": "simulation",
  "status": "rejected",
  "file_changed": false,
  "commit_created": false,
  "push_executed": false,
  "write_executed": false
}
```

Die Fehlermeldung muss statisch sein.

Sie darf keine Eingangswerte, Pfade, Inhalte, Commit-Nachrichten, Credentials oder technischen Fehlerdetails enthalten.

---

## 25. Verbindliche Fehlercodes

Folgende öffentliche Fehlercodes sind zulässig:

```text
INVALID_INPUT_COUNT
MISSING_REQUIRED_FIELD
INVALID_FIELD_TYPE
INVALID_MODE
INVALID_CONTROLLER_SOURCE
CONTROLLER_NOT_PREPARED
AUDIT_NOT_PASSED
TARGET_NOT_ALLOWED
INVALID_PATH
INVALID_REF
SOURCE_SHA_MISSING
INVALID_SOURCE_SHA
INVALID_CONTENT
CONTENT_TOO_LARGE
INVALID_COMMIT_MESSAGE
PATCH_VALIDATION_FAILED
INTERNAL_ERROR
```

Andere öffentliche Fehlercodes sind in v0.2.0 nicht zulässig.

Die exakten statischen Fehlertexte werden verbindlich aus `FLOW_v0.2.0.md` übernommen.

---

## 26. Technische Fehlerbehandlung

Ein unerwarteter interner Fehler wird sicher abgefangen.

Öffentliches Ergebnis:

```text
status = rejected
error_code = INTERNAL_ERROR
message = Die Simulation konnte nicht sicher abgeschlossen werden.
```

Nicht öffentlich ausgegeben werden:

* Stacktraces,
* rohe Fehlerobjekte,
* Eingangsdumps,
* Node-Konfigurationen,
* interner Programmcode,
* Dateiinhalte,
* Credential- oder Authorization-Daten.

Auch der technische Fehlerweg muss den Output Sanitizer und anschließend den Final Output erreichen.

---

## 27. Output Sanitizer

Alle Ergebniswege müssen zwingend durch den Output Sanitizer geführt werden.

Der Sanitizer arbeitet mit festen Allowlists.

Er entfernt insbesondere:

```text
change.content
source.approved_by
checks
normalized
errors
decision
patch
patch_check
metadata
stack
credentials
credential
token
secret
authorization
```

Zusätzlich entfernt er:

* unbekannte Felder,
* interne Prüfwerte,
* rohe Eingangsobjekte,
* dynamische technische Details,
* nicht ausdrücklich erlaubte verschachtelte Felder.

Der Sanitizer überschreibt folgende Werte unabhängig vom internen Zustand:

```json
{
  "file_changed": false,
  "commit_created": false,
  "push_executed": false,
  "write_executed": false
}
```

Kein Ergebnisweg darf den Sanitizer umgehen.

---

## 28. Final Output

Der Final Output gibt genau ein bereinigtes Item aus.

Zulässig sind ausschließlich:

```text
status = simulated
```

oder:

```text
status = rejected
```

Es dürfen nicht entstehen:

* entstehen:

* null End-Items,

* mehrere End-Items,

* parallele öffentliche Ergebnisse,

* ungefilterte Zwischenobjekte,

* automatische Zusatzmeldungen.

---

## 29. Determinismus

WF-0011 v0.2.0 muss bei identischem Eingang fachlich identische Ergebnisse erzeugen.

Nicht zulässig sind:

* automatisch erzeugte IDs,
* Zeitstempel im Endergebnis,
* Zufallswerte,
* wechselnde Standardwerte,
* wechselnde Fehlerprioritäten,
* umgebungsabhängige öffentliche Ergebnisse.

Eine fehlende `request_id` bleibt fehlend.

Die Reihenfolge der Prüfungen und Fehlerpriorität ist verbindlich.

---

## 30. Seiteneffektfreiheit

Für jede Ausführung gelten zwingend:

```text
file_changed = false
commit_created = false
push_executed = false
write_executed = false
```

Zusätzlich gilt:

```text
GitHub-Aufrufe = 0
Dateioperationen = 0
Commits = 0
Pushes = 0
Shell-Ausführungen = 0
automatisch gestartete Folge-Workflows = 0
```

Diese Regeln gelten für:

* Erfolgswege,
* Schema-Ablehnungen,
* Sicherheitsablehnungen,
* Patch-Ablehnungen,
* interne Fehler.

---

## 31. Verbotene technische Komponenten

WF-0011 v0.2.0 darf keine Komponenten enthalten, die Seiteneffekte ausführen können.

Verboten sind insbesondere:

* GitHub-Nodes,
* HTTP-Request-Nodes,
* Execute-Command-Nodes,
* SSH-Nodes,
* Datei-Schreibnodes,
* Datei-Löschnodes,
* Git-Nodes,
* Datenbank-Schreibnodes,
* E-Mail- oder Messaging-Nodes,
* aktivierende Webhook-Trigger,
* Schedule- oder Cron-Trigger,
* Nodes zum Starten anderer Workflows,
* Community-Nodes mit ungeklärtem Verhalten,
* Code, der Netzwerk-, Datei- oder Systemzugriffe ausführt.

Zulässig sind nur die in `ARCHITECTURE_v0.2.0.md` und `FLOW_v0.2.0.md` freigegebenen, seiteneffektfreien Komponenten.

---

## 32. Credential-Regel

Für WF-0011 v0.2.0 gilt:

```text
eingebundene Credentials = 0
```

Verboten sind:

* Credential-Zuordnungen,
* Credential-IDs,
* Tokens,
* Secrets,
* Passwörter,
* private Schlüssel,
* Authorization-Header,
* produktive Zugangsdaten,
* als Testdaten getarnte reale Zugangsdaten.

Credential-Marker in kontrollierten Sicherheitstests müssen abgelehnt und aus dem Endergebnis entfernt werden.

---

## 33. n8n-Datenhaltung

Vor einer Statusänderung von `draft` zu `testing` muss geprüft und dokumentiert werden, welche Daten n8n speichert.

Zu bewerten sind insbesondere:

* erfolgreiche Ausführungsdaten,
* fehlerhafte Ausführungsdaten,
* manuelle Ausführungsdaten,
* rohe Eingangsdaten,
* vollständige Dateiinhalte,
* kanonische Aufträge,
* Patch-Metadaten,
* interne Prüfobjekte,
* Fehlerobjekte,
* Stacktraces,
* Aufbewahrungsdauer,
* Pruning und automatische Löschung.

Bis zum Abschluss dieser Prüfung dürfen ausschließlich künstliche und nicht vertrauliche Testdaten verwendet werden.

Der Workflow bleibt bis dahin:

```text
inactive
draft
not-approved
```

---

## 34. Abbruchkriterien

Die Entwicklung oder Prüfung wird sofort gestoppt, wenn:

* eine Datei erstellt, verändert, umbenannt oder gelöscht wird,
* ein GitHub-Aufruf erfolgt,
* ein Commit erstellt wird,
* ein Push ausgeführt wird,
* ein Credential eingebunden ist,
* ein Token oder Secret sichtbar wird,
* ein automatischer Trigger aktiv ist,
* ein anderer Workflow automatisch gestartet wird,
* ein Shell- oder Systembefehl ausgeführt wird,
* der Output Sanitizer umgangen wird,
* ein technischer Fehler ungefiltert ausgegeben wird,
* mehr als ein End-Item entsteht.

Nach einem Abbruch bleibt der Workflow:

```text
inactive
draft
not-approved
```

Der Vorfall wird in `KNOWN_ISSUES.md` dokumentiert.

---

## 35. Abnahmekriterien

WF-0011 v0.2.0 darf frühestens von `draft` zu `testing` wechseln, wenn:

* Specification, Architecture, Flow und Tests widerspruchsfrei sind,
* der Workflow vollständig, aber inaktiv angelegt wurde,
* keine verbotenen Nodes enthalten sind,
* keine Credentials eingebunden sind,
* alle Ergebniswege den Sanitizer erreichen,
* die Fehlerpriorität technisch umgesetzt ist,
* genau ein End-Item entsteht,
* die Seiteneffektfreiheit nachvollziehbar ist,
* die n8n-Datenhaltung geprüft wurde,
* ausschließlich kontrollierte Testdaten verwendet werden.

Eine weitere Freigabe setzt voraus, dass alle Pflichtfälle aus `TESTS_v0.2.0.md` bestanden wurden.

---

## 36. Definition of Done

WF-0011 v0.2.0 gilt als spezifikationsgemäß umgesetzt, wenn:

* genau ein vorbereiteter Auftrag verarbeitet wird,
* nur `simulation` akzeptiert wird,
* ausschließlich WF-0013 als Controller-Herkunft akzeptiert wird,
* `prepared` und `passed` verbindlich geprüft werden,
* nur freigegebene Owner-Repository-Paare zulässig sind,
* Pfade sicher normalisiert und validiert werden,
* Referenz und SHA korrekt geprüft werden,
* Inhalt und Commit-Nachricht innerhalb der Grenzwerte liegen,
* ein kanonischer interner Auftrag erzeugt wird,
* ein nicht angewendeter Patch simuliert wird,
* Patch-Manipulationen sicher abgelehnt werden,
* die Fehlerpriorität deterministisch eingehalten wird,
* alle Ergebnisse bereinigt werden,
* genau ein End-Item ausgegeben wird,
* keine internen oder vertraulichen Daten öffentlich erscheinen,
* keine Datei verändert wird,
* kein Commit und kein Push erfolgt,
* kein GitHub- oder Netzwerkaufruf erfolgt,
* keine Credentials eingebunden sind,
* kein anderer Workflow gestartet wird,
* der Workflow inaktiv bleibt,
* alle Tests aus `TESTS_v0.2.0.md` bestanden sind.

---

## 37. Verbindliche Kerndokumente

Für WF-0011 v0.2.0 gelten gemeinsam:

```text
SPECIFICATION_v0.2.0.md
ARCHITECTURE_v0.2.0.md
FLOW_v0.2.0.md
TESTS_v0.2.0.md
```

Dabei beschreibt:

```text
SPECIFICATION_v0.2.0.md
→ fachliche und sicherheitstechnische Anforderungen

ARCHITECTURE_v0.2.0.md
→ technische Komponenten und Verantwortungsgrenzen

FLOW_v0.2.0.md
→ konkrete Node-Reihenfolge, Prüfungen und Fehlertexte

TESTS_v0.2.0.md
→ Nachweis der Umsetzung und Seiteneffektfreiheit
```

Keines dieser Dokumente darf isoliert als vollständige Freigabegrundlage verwendet werden.

Bei einem Widerspruch bleibt der Workflow gesperrt, bis der Widerspruch dokumentiert und behoben wurde.

---

## 38. Aktueller Umsetzungsstatus

```text
Workflow: WF-0011 – GitHub Writer
Version: 0.2.0
Status: draft
Mode: simulation
Specification: defined
Architecture: incomplete
Flow: defined
Tests: defined
Implementation: not-started
Workflow active: false
Credentials: forbidden
GitHub access: forbidden
File changes: forbidden
Commits: forbidden
Pushes: forbidden
External side effects: forbidden
Approval: not-approved
```

---

## 39. Nächster konkreter Schritt

```text
ARCHITECTURE_v0.2.0.md vollständig neu aufbauen.

Anschließend SPECIFICATION_v0.2.0.md,
ARCHITECTURE_v0.2.0.md, FLOW_v0.2.0.md und
TESTS_v0.2.0.md gemeinsam auf Widersprüche prüfen.

Erst nach bestandenem Dokumentationsabgleich darf der
inaktive n8n-Workflow WF-0011 v0.2.0 angelegt werden.
```
