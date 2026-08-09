# WF-0011 – GitHub Writer
## Architecture v0.2.0

---

## 1. Dokumentstatus

```text
Workflow: WF-0011 – GitHub Writer
Version: 0.2.0
Dokument: ARCHITECTURE_v0.2.0.md
Status: draft
Mode: simulation
Workflow active: false
Approval: not-approved
```

Dieses Dokument beschreibt die technische Architektur von WF-0011 v0.2.0.

Die Architektur ist ausschließlich für eine seiteneffektfreie Simulation vorgesehen.

Sie erlaubt keine produktiven Schreib-, GitHub-, Git-, Datei-, Netzwerk- oder Systemoperationen.

---

## 2. Zweck der Architektur

WF-0011 v0.2.0 verarbeitet genau einen bereits vorbereiteten Dateiänderungsauftrag.

Der Workflow:

1. nimmt genau ein Eingangsobjekt entgegen,
2. prüft Struktur und Datentypen,
3. prüft Ausführungsmodus und Herkunft,
4. validiert Ziel, Pfad, SHA, Inhalt und Commit-Nachricht,
5. erzeugt einen kanonischen internen Auftrag,
6. erzeugt ein nicht angewendetes Patch-Objekt,
7. prüft das Patch-Objekt auf Manipulationen,
8. erzeugt genau ein bereinigtes Endergebnis.

Der Workflow führt keine reale Änderung aus.

---

## 3. Architekturprinzipien

Für WF-0011 v0.2.0 gelten verbindlich:

```text
Simulation vor Ausführung
Seiteneffektfreiheit
Fail closed
Genau ein Eingangsauftrag
Genau ein End-Item
Deterministische Fehlerpriorität
Explizite Allowlist
Lesender lokaler Repository-Preflight
Harte, dauerhaft geschlossene Schreibgrenze
Keine impliziten Standardwerte
Keine Credentials
Keine externen Aufrufe
Bereinigung aller Endergebnisse
```

Unklare, unvollständige oder widersprüchliche Zustände werden abgelehnt.

---

## 4. Systemgrenze

WF-0011 v0.2.0 beginnt mit der Entgegennahme eines bereits vorbereiteten Auftrags.

WF-0011 ist nicht verantwortlich für:

- die Erzeugung des ursprünglichen Änderungswunsches,
- die fachliche Freigabe des Änderungswunsches,
- das Abrufen einer Datei,
- das Ermitteln eines aktuellen GitHub-SHA,
- das Schreiben oder Löschen einer Datei,
- die Erstellung eines Commits,
- das Pushen eines Commits,
- die Aktivierung anderer Workflows,
- die Benachrichtigung von Personen oder Systemen.

WF-0011 v0.2.0 plant ausschließlich die lesende Prüfung eines
vorkonfigurierten lokalen Repositorys. Diese Prüfung ist keine Freigabe zum
Schreiben und noch nicht technisch implementiert.

Die Controller-Herkunft wird ausschließlich anhand der übergebenen Daten geprüft.

WF-0011 ruft WF-0013 nicht selbst auf.

---

## 5. Zulässiger Betriebszustand

Während der Entwicklung und Prüfung gilt:

```text
workflow active = false
status = draft
mode = simulation
credentials = 0
external side effects = 0
```

Ein anderer Betriebszustand ist für v0.2.0 nicht freigegeben.

Insbesondere existiert kein produktiver Schreibmodus.

---

## 6. Eingangsgrenze

WF-0011 erwartet genau ein n8n-Item mit genau einem JSON-Auftrag.

Das fachliche Eingangsmodell besteht aus:

```text
execution
target
source
change
request_id, optional
```

Verbindliche Eingangsfelder:

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

Das Eingangsschema wird in `SPECIFICATION_v0.2.0.md` fachlich und in `FLOW_v0.2.0.md` operativ festgelegt.

---

## 7. Ausgangsgrenze

WF-0011 erzeugt genau ein End-Item.

Zulässige Endzustände:

```text
simulated
rejected
```

Der Workflow darf niemals mehrere Ergebnis-Items erzeugen.

Der Output darf insbesondere nicht enthalten:

- vollständigen Dateiinhalt,
- unbearbeitete Eingangsdaten,
- interne Prüfobjekte,
- internes Patch-Objekt,
- Stacktraces,
- Node-Namen,
- Credential-Daten,
- Tokens oder Secrets,
- Authorization-Header,
- interne Fehlerdetails.

---

## 8. Technische Gesamtstruktur

Die Architektur besteht aus folgenden logischen Schichten:

```text
1. Input Boundary
2. Input Envelope
3. Schema Validation
4. Policy Validation
5. Repository Preflight Boundary
6. Canonicalization
7. Patch Simulation
8. Patch Guard
9. Closed Write Boundary
10. Result Builder
11. Output Sanitizer
12. Single Output Boundary
```

Jede Schicht besitzt eine klar begrenzte Verantwortung.

---

## 9. Kontrollfluss

```text
Manual Input
→ Input Boundary
→ Validation Pipeline
→ Read-only Repository Preflight
→ Canonical Request Builder
→ Patch Simulation
→ Patch Guard
→ Closed Write Boundary
→ Result Builder
→ Output Sanitizer
→ Single End Item
```

Ablehnungen verlassen die Verarbeitung nicht direkt.

Jede Ablehnung wird als kontrollierter interner Entscheidungszustand weitergeführt und erreicht ebenfalls den Output Sanitizer.

Es darf keinen Bypass um den Sanitizer geben.

---

## 10. Input Boundary

Die Input Boundary stellt sicher:

```text
eingehende Items = 1
```

Folgende Zustände werden abgelehnt:

- kein Eingangs-Item,
- mehr als ein Eingangs-Item,
- ein nicht objektförmiger Eingang,
- ein Array als Hauptobjekt,
- `null` als Hauptobjekt.

Die Input Boundary:

- ergänzt keine Daten,
- normalisiert keine fachlichen Werte,
- führt keine externen Abfragen aus,
- gibt keine rohen Eingangsdaten öffentlich aus.

---

## 11. Internes Verarbeitungs-Envelope

Nach der Input Boundary wird intern ein kontrolliertes Verarbeitungsobjekt geführt.

Konzeptionelle Struktur:

```json
{
  "input": {},
  "decision": {
    "accepted": null,
    "error_code": null,
    "message_key": null,
    "priority": null
  },
  "canonical_request": null,
  "patch": null,
  "public_result": null
}
```

Dieses Envelope ist ausschließlich intern.

Es darf nicht vollständig ausgegeben werden.

Sobald eine höher priorisierte Ablehnung feststeht, dürfen nachfolgende Prüfungen diese Entscheidung nicht überschreiben.

---

## 12. Schema Validation

Die Schema Validation prüft:

- Vorhandensein der Pflichtbereiche,
- Vorhandensein der Pflichtfelder,
- erwartete Datentypen,
- Struktur der verschachtelten Objekte,
- optionales Format von `request_id`.

Fehlende Pflichtfelder führen zu:

```text
MISSING_REQUIRED_FIELD
```

Falsche Datentypen führen zu:

```text
INVALID_FIELD_TYPE
```

Die Schema Validation führt noch keine Ziel-, Pfad-, SHA- oder Inhaltsbewertung durch.

---

## 13. Ausführungsprüfung

Zulässig ist ausschließlich:

```text
execution.mode = simulation
```

Andere Werte werden abgelehnt.

Es existiert keine Architekturkomponente, die einen produktiven Modus umsetzen könnte.

Das öffentliche Ergebnis verwendet weiterhin:

```text
mode = simulation
```

`execution.mode` bezeichnet das Eingangsfeld.

`mode` bezeichnet das bereinigte öffentliche Ausgabefeld.

---

## 14. Herkunftsprüfung

Die Herkunftsprüfung akzeptiert ausschließlich:

```text
source.controller_workflow = WF-0013
source.controller_status = prepared
source.audit_status = passed
source.approved_by = manual_review
```

Die Prüfung basiert ausschließlich auf dem Eingang.

Sie führt keinen Aufruf von WF-0013 aus.

Sie beweist nicht, dass WF-0013 tatsächlich ausgeführt wurde. Sie prüft nur, ob der vorbereitete Auftrag der verbindlichen Schnittstelle entspricht.

Abweichungen werden gemäß `FLOW_v0.2.0.md` deterministisch abgelehnt.

---

## 15. Ziel-Allowlist

WF-0011 v0.2.0 akzeptiert ausschließlich:

```text
target.owner = Jamoko112026
target.repository = JaMoKo_Automation_OS
target.ref = main
```

Owner und Repository werden als festes Paar geprüft.

Eine unabhängige Vermischung freigegebener Einzelwerte ist nicht vorgesehen.

Die Allowlist ist im Workflow statisch definiert.

Zusätzlich gilt eine getrennte, statische Zielpfad-Allowlist. Für den
v0.2.0-Entwurf ist ausschließlich der simulationsbezogene Testpfad

```text
02_WORKFLOWS/WF-0011_GitHub_Writer/README.md
```

vorgesehen. Diese Regel erteilt keine Schreibfreigabe. Syntaktisch sichere,
aber nicht gelistete Pfade werden mit `TARGET_PATH_NOT_ALLOWED` abgelehnt.

Sie wird nicht aus:

- Umgebungsvariablen,
- Credentials,
- externen Dateien,
- Datenbanken,
- APIs,
- anderen Workflows

geladen.

---

## 16. Pfadarchitektur

Die Pfadverarbeitung erfolgt in zwei getrennten Phasen:

```text
1. kontrollierte Normalisierung
2. Sicherheitsvalidierung
```

Der ursprüngliche Pfad darf höchstens einmal URL-dekodiert werden.

Nach der einmaligen Dekodierung verbleibende `%XX`-Kodierungen werden abgelehnt.

Die Architektur muss insbesondere erkennen und ablehnen:

- absolute Pfade,
- führende Schrägstriche,
- Backslashes,
- leere Pfade,
- `.`-Segmente,
- `..`-Segmente,
- Traversal-Versuche,
- doppelte Kodierung,
- Nullbytes,
- Steuerzeichen,
- URL- oder URI-Schemata,
- Query- oder Fragmentbestandteile,
- unzulässige Pfadnormalisierung,
- Pfade außerhalb des erlaubten Repository-Kontexts.

Der normalisierte Pfad wird erst nach bestandener Sicherheitsprüfung in den kanonischen Auftrag übernommen.

Anschließend wird der Pfad exakt gegen die Zielpfad-Allowlist geprüft. Die
Allowlist-Prüfung findet statt, bevor der lokale Repository-Preflight aufgerufen
werden darf.

---

## 16.1 Repository-Preflight-Architektur

Der Repository-Preflight ist eine eigene Vertrauensgrenze. Er erhält weder den
Dateiinhalt noch die Commit-Nachricht noch Credentials. Er prüft ausschließlich
ein vorkonfiguriertes lokales Ziel und liefert intern nur feste boolesche
Prüfwerte beziehungsweise einen kontrollierten Fehlercode.

Verbindlich zu prüfen sind:

```text
is_git_repository = true
repository_identity_matches = true
current_branch = main
working_tree_clean = true
```

`working_tree_clean = true` bedeutet: keine vorgemerkten, nicht vorgemerkten
oder unversionierten Änderungen. Der Repository-Pfad wird nicht aus
`target.path` oder einem anderen Eingangsfeld gebildet.

Der Adapter darf keine Datei verändern, keinen Git-Index aktualisieren, keine
Locks oder Hooks erzeugen, kein Netzwerk verwenden und weder rohe Standardaus-
noch Standardfehlerdaten weitergeben. Der konkrete n8n-kompatible Adapter ist
noch nicht entschieden. Bis zu seinem Konformitätsnachweis bleibt die
Implementierung `not-started`.

---

## 16.2 Geschlossene Write Boundary

Nach erfolgreicher Patch-Validierung folgt eine zentrale Write Boundary. Für
v0.2.0 ist sie konstant geschlossen:

```text
write_allowed = false
commit_allowed = false
push_allowed = false
```

Es existiert kein ausführender Zweig hinter dieser Grenze. Validierung,
kanonische Vorbereitung und In-Memory-Simulation sind zulässig. Dateiänderung,
Commit und Push sind verboten. Jeder Schreibwunsch oder Umgehungsversuch wird
mit `WRITE_NOT_ALLOWED` abgelehnt.

---

## 17. Referenzarchitektur

Für v0.2.0 gilt ausschließlich:

```text
target.ref = main
```

Branches, Tags, Commit-Referenzen oder dynamische Refs werden nicht unterstützt.

Die Eingangsprüfung erfolgt zunächst exakt gegen den übergebenen String. Der
spätere lokale Repository-Preflight bestätigt zusätzlich ausschließlich
lesend, dass der aktuelle lokale Branch `main` ist. Es findet kein GitHub-
Abgleich und keine Änderung des Repositorys statt.

---

## 18. SHA-Architektur

`source.expected_sha` ist ein Pflichtfeld.

Die technische Unterscheidung lautet:

```text
Feld fehlt
→ MISSING_REQUIRED_FIELD

Feld vorhanden, aber leer
→ SOURCE_SHA_MISSING

Feld vorhanden, aber nicht exakt 40-stellig hexadezimal
→ INVALID_SOURCE_SHA
```

Zulässiges Format:

```text
^[0-9a-fA-F]{40}$
```

Der SHA wird:

- nicht über GitHub geprüft,
- nicht über Git ermittelt,
- nicht automatisch ergänzt,
- nicht verändert,
- nicht als Existenzbeweis interpretiert.

Er dient ausschließlich als kontrollierter Bestandteil der Simulation.

---

## 19. Inhaltsarchitektur

`change.content` wird als vollständiger vorgesehener Dateiinhalt behandelt.

Der Inhalt:

- muss eine Zeichenkette sein,
- darf leer sein,
- wird als UTF-8 verarbeitet,
- darf maximal `100000` UTF-8-Bytes umfassen.

Die Byte-Länge wird deterministisch ermittelt.

Die Architektur darf den vollständigen Inhalt nur so lange intern halten, wie dies für Validierung, Byte-Ermittlung und Patch-Simulation notwendig ist.

Der vollständige Inhalt darf nicht erscheinen in:

- Erfolgsergebnissen,
- Ablehnungsergebnissen,
- öffentlichen Fehlermeldungen,
- Debug-Ausgaben,
- manuell ergänzten Diagnosefeldern.

Bis zur abgeschlossenen Prüfung der n8n-Datenhaltung dürfen nur künstliche, nicht vertrauliche Testinhalte verwendet werden.

---

## 20. Commit-Nachrichten-Architektur

`change.commit_message`:

- muss eine Zeichenkette sein,
- darf nach Trimmung nicht leer sein,
- darf maximal 120 Zeichen lang sein,
- muss einzeilig sein,
- darf keine Steuerzeichen enthalten,
- darf keine Credentials oder Secrets transportieren.

Die Nachricht wird nicht automatisch erzeugt oder ergänzt.

Sie wird nur in das interne Simulationsobjekt übernommen.

Es existiert keine Komponente zur Erstellung eines Commits.

---

## 21. Secret- und Credential-Schutz

WF-0011 v0.2.0 besitzt keine eingebundenen Credentials.

```text
credential count = 0
```

Kontrollierte Prüfungen müssen verdächtige Credential- oder Secret-Marker erkennen und ablehnen.

Zu den zu schützenden Mustern gehören insbesondere:

- Tokens,
- Passwörter,
- private Schlüssel,
- Authorization-Header,
- GitHub-Zugangsdaten,
- Credential-IDs,
- Secret-Felder.

Verdächtige Werte dürfen nicht in das öffentliche Ergebnis kopiert werden.

Die Ablehnung darf den gefundenen Geheimwert nicht wiederholen.

Preflight-Adapter und Fehlerbehandlung dürfen lokale Repository-Pfade, rohe
Git-Ausgaben, Standardausgaben oder Standardfehler nicht in das interne
Workflow-Envelope übernehmen. Der Output Sanitizer baut öffentliche Ergebnisse
und Exporte ausschließlich aus festen Feld-Allowlists neu auf.

---

## 22. Kanonischer Auftrag

Nach allen bestandenen Eingangsprüfungen wird ein internes kanonisches Auftragsobjekt erzeugt.

Es enthält ausschließlich normalisierte und freigegebene Werte.

Konzeptionelle Struktur:

```json
{
  "request_id": "optional",
  "mode": "simulation",
  "target": {
    "owner": "Jamoko112026",
    "repository": "JaMoKo_Automation_OS",
    "ref": "main",
    "path": "normalized/path.md"
  },
  "source": {
    "controller_workflow": "WF-0013",
    "controller_status": "prepared",
    "audit_status": "passed",
    "approved_by": "manual_review",
    "expected_sha": "0123456789abcdef0123456789abcdef01234567"
  },
  "change": {
    "content": "internal only",
    "content_bytes": 0,
    "commit_message": "Controlled message"
  }
}
```

Der kanonische Auftrag ist kein öffentliches Ergebnis.

Unbekannte Eingangsfelder werden nicht übernommen.

---

## 23. Patch-Simulation

Aus dem kanonischen Auftrag wird intern ein Patch-Objekt erzeugt.

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

Verbindliche Zustände:

```text
content_encoding = utf-8
applied = false
```

Das Patch-Objekt beschreibt ausschließlich eine beabsichtigte Änderung.

Es besitzt keine Methode und keine technische Verbindung zur realen Anwendung eines Patches.

---

## 24. Verbotene Patch-Eigenschaften

Das Patch-Objekt darf insbesondere nicht enthalten:

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

Es darf außerdem keinen Wert enthalten, der fälschlich behauptet:

- eine Datei sei geschrieben worden,
- ein Commit sei erstellt worden,
- ein Push sei erfolgt,
- GitHub sei aufgerufen worden,
- eine Freigabe sei produktiv ausgeführt worden.

---

## 25. Patch Guard

Der Patch Guard prüft das intern erzeugte Patch-Objekt vor dem Success Builder.

Er kontrolliert mindestens:

- vollständige erlaubte Patch-Struktur,
- keine zusätzlichen verbotenen Eigenschaften,
- `content_encoding = utf-8`,
- korrekte `content_bytes`,
- Übereinstimmung der Zielwerte,
- Übereinstimmung des SHA,
- Übereinstimmung der Commit-Nachricht,
- `applied = false`.

Eine Abweichung führt zu einer kontrollierten Patch-Ablehnung.

Ein abgelehnter Patch darf den Success Builder nicht erreichen.

Der Patch Guard darf keine Reparatur eines manipulierten Patch-Objekts durchführen.

---

## 26. Fehlerarchitektur

Alle fachlichen und sicherheitstechnischen Fehler werden als kontrollierte interne Ablehnungsentscheidung behandelt.

Fehler werden nicht durch rohe technische Ausnahmen öffentlich dargestellt.

Die Architektur trennt:

```text
fachlicher Fehler
Sicherheitsfehler
Patch-Fehler
interner technischer Fehler
```

Jeder Fehlerweg führt zum Rejection Builder und anschließend zum Output Sanitizer.

---

## 27. Deterministische Fehlerpriorität

Wenn mehrere Fehler gleichzeitig vorliegen, darf nur der laut `FLOW_v0.2.0.md` höchstpriorisierte Fehler öffentlich erscheinen.

Die technische Umsetzung verwendet einen internen Entscheidungszustand mit einer festen Priorität.

Grundregeln:

1. Eine gesetzte Ablehnung wird nicht durch einen niedriger priorisierten Fehler überschrieben.
2. Pro Auftrag wird genau ein öffentlicher Fehlercode ausgegeben.
3. Die Reihenfolge der Node-Ausführung darf das Ergebnis nicht zufällig verändern.
4. Unbekannte technische Fehler erhalten ein kontrolliertes, bereinigtes Fehlerergebnis.
5. Technische Details dürfen nicht öffentlich erscheinen.

Die vollständige Reihenfolge der Fehlerpriorität wird ausschließlich in `FLOW_v0.2.0.md` festgelegt.

---

## 28. Success Builder

Der Success Builder darf nur erreicht werden, wenn:

- alle Pflichtprüfungen bestanden sind,
- kein Ablehnungszustand gesetzt ist,
- der kanonische Auftrag gültig ist,
- das Patch-Objekt gültig ist,
- `applied = false` bestätigt wurde.

Er erzeugt ein kontrolliertes Erfolgsergebnis mit:

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

```text
workflow_id = WF-0011
version = 0.2.0
mode = simulation
status = simulated
file_changed = false
commit_created = false
push_executed = false
write_executed = false
```

Der Success Builder gibt keinen vollständigen Dateiinhalt aus.

---

## 29. Rejection Builder

Der Rejection Builder erzeugt ausschließlich kontrollierte Ablehnungsergebnisse.

Zulässige öffentliche Felder:

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

Verbindliche Werte:

```text
workflow_id = WF-0011
version = 0.2.0
mode = simulation
status = rejected
file_changed = false
commit_created = false
push_executed = false
write_executed = false
```

Die öffentliche Nachricht stammt aus einer festen Zuordnung erlaubter Fehlercodes zu bereinigten Meldungen.

Rohe Fehlermeldungen werden nicht übernommen.

---

## 30. Output Sanitizer

Der Output Sanitizer ist die letzte verarbeitende Komponente vor dem Workflow-Ausgang.

Er wird von jedem Ergebnisweg erreicht.

Der Sanitizer:

- verwendet eine explizite Allowlist öffentlicher Felder,
- entfernt unbekannte Felder,
- entfernt interne Objekte,
- entfernt den vollständigen Dateiinhalt,
- entfernt Patch-Details, die nicht öffentlich vorgesehen sind,
- entfernt technische Fehlerdetails,
- entfernt Credential- und Secret-Marker,
- erzwingt die vier `false`-Werte für Seiteneffekte,
- erzeugt genau ein End-Item.

Kein Ergebnis darf den Sanitizer umgehen.

---

## 31. Single Output Boundary

Nach dem Output Sanitizer existiert genau ein gemeinsamer Ausgang.

Zulässig:

```text
output items = 1
```

Unzulässig:

```text
output items = 0
output items > 1
```

Mehrere parallele End-Nodes sind nicht erlaubt.

---

## 32. Zulässige n8n-Komponenten

Für WF-0011 v0.2.0 sind ausschließlich seiteneffektfreie Core-Komponenten zulässig.

Grundsätzlich zulässig:

- Manual Trigger für kontrollierte manuelle Tests,
- Code-Nodes mit rein lokaler Datenverarbeitung,
- seiteneffektfreie Kontrollfluss-Nodes,
- seiteneffektfreie Merge-Funktionen, falls der Flow sie eindeutig benötigt.

Zusätzlich geplant, aber noch nicht technisch freigegeben, ist genau ein
gekapselter Repository-Preflight-Adapter. Er darf ausschließlich die in
Abschnitt 16.1 definierten lesenden Prüfwerte liefern. Seine konkrete n8n-
Umsetzung bleibt bis zu einem gesonderten Sicherheitsnachweis offen.

Code-Nodes dürfen ausschließlich:

- JSON-Daten lesen,
- Werte vergleichen,
- Zeichenketten prüfen,
- Pfade normalisieren,
- UTF-8-Byte-Längen bestimmen,
- interne Objekte erzeugen,
- bereinigte Ergebnisobjekte erzeugen.

Die konkrete Node-Reihenfolge wird in `FLOW_v0.2.0.md` verbindlich festgelegt.

---

## 33. Verbotene n8n-Komponenten

Nicht zulässig sind insbesondere:

- GitHub-Nodes,
- HTTP-Request-Nodes,
- nicht ausdrücklich als konformer lesender Preflight-Adapter freigegebene
  Execute-Command-Nodes,
- SSH-Nodes,
- Git-Nodes mit Netzwerk- oder Mutationsfähigkeit,
- Datei-Schreibnodes,
- Datei-Löschnodes,
- Datenbank-Schreibnodes,
- E-Mail-Nodes,
- Messaging-Nodes,
- aktivierte Webhook-Trigger,
- Schedule- oder Cron-Trigger,
- Nodes zum Starten anderer Workflows,
- Community-Nodes mit ungeklärtem Verhalten,
- AI-Nodes mit externem Modellaufruf,
- Code mit Netzwerkzugriff,
- Code mit schreibendem Dateisystemzugriff,
- Code mit nicht freigegebenem Prozess- oder Systemzugriff.

---

## 34. Verbotene Code-Funktionen

Code innerhalb von WF-0011 darf insbesondere nicht verwenden:

```text
fetch
XMLHttpRequest
axios
http
https
net
tls
dns
schreibende fs-Funktionen
mutierende child_process-Aufrufe
mutierende process execution
mutierende shell commands
database clients
credential APIs
workflow execution APIs
```

Auch indirekte oder dynamisch geladene Varianten dieser verbotenen Zugriffe sind verboten.

Code darf keine Seiteneffekte außerhalb des aktuellen In-Memory-Verarbeitungszustands erzeugen.

---

## 35. Credential-Architektur

WF-0011 besitzt:

```text
credentials = 0
```

Keine Node darf eine Credential-Zuordnung enthalten.

Workflow-Export und technische Prüfung müssen bestätigen:

- keine Credential-ID,
- kein Credential-Name,
- kein Token,
- kein Secret,
- kein Passwort,
- kein privater Schlüssel,
- kein Authorization-Header.

Auch Test-Credentials sind nicht zulässig.

---

## 36. Trigger-Architektur

Der Workflow bleibt inaktiv.

Für kontrollierte Tests ist ausschließlich ein manueller Start vorgesehen.

Nicht zulässig sind:

- aktive Webhooks,
- Zeitpläne,
- Cron-Ausführung,
- automatische Workflow-Aufrufe,
- Event-Trigger,
- externe produktive Trigger.

Die Inaktivität des Workflows ist Bestandteil der Sicherheitsarchitektur.

---

## 37. Technische Fehlerbehandlung

Unerwartete interne Fehler dürfen nicht ungefiltert ausgegeben werden.

Insbesondere verboten im öffentlichen Ergebnis:

- Stacktraces,
- JavaScript-Fehlermeldungen,
- Quellcodezeilen,
- Node-Konfigurationen,
- interne Eingangsdaten,
- Dateiinhalte,
- Tokens oder Secrets.

Ein interner Fehler wird in ein fest definiertes, bereinigtes Ablehnungsergebnis überführt.

Der genaue öffentliche Fehlercode und die Meldung werden in `FLOW_v0.2.0.md` festgelegt.

---

## 38. Logging und n8n-Datenhaltung

Die Workflow-Architektur kann nicht allein verhindern, dass n8n Ausführungsdaten speichert.

Vor dem Statuswechsel zu `testing` müssen deshalb geprüft werden:

- Speicherung erfolgreicher Ausführungen,
- Speicherung fehlerhafter Ausführungen,
- Speicherung manueller Ausführungen,
- Speicherung von Node-Eingaben,
- Speicherung von Node-Ausgaben,
- Speicherung vollständiger Dateiinhalte,
- Aufbewahrungsdauer,
- Pruning,
- automatische Löschung,
- Zugriffsrechte auf Ausführungsdaten.

Zusätzlich muss der Preflight-Nachweis bestätigen, dass lokale Pfade, rohe
Git-Ausgaben sowie Standardaus- und Standardfehler weder in n8n-Logs noch in
Ausführungsdaten oder Exporte gelangen. Der Output Sanitizer allein reicht für
diese Plattformgrenze nicht aus.

Bis zur dokumentierten Klärung sind ausschließlich künstliche und nicht vertrauliche Testdaten zulässig.

---

## 39. Seiteneffekt-Nachweis

Die Seiteneffektfreiheit wird auf vier Ebenen nachgewiesen:

### 39.1 Komponentenebene

Es sind keine Nodes mit externen Schreib- oder Netzwerkfähigkeiten enthalten.
Der einzige geplante lokale Zugriff ist der gekapselte lesende
Repository-Preflight.

### 39.2 Codeebene

Code greift nicht auf Netzwerk oder externe Systeme zu und führt keine
Dateiänderung oder mutierende Git-, Prozess- oder Systemoperation aus. Der
Repository-Preflight darf ausschließlich nach separatem Konformitätsnachweis
lesend auf den vorkonfigurierten lokalen Repository-Zustand zugreifen.

### 39.3 Datenebene

Alle Ergebnisobjekte bestätigen:

```text
file_changed = false
commit_created = false
push_executed = false
write_executed = false
```

### 39.4 Workflowebene

Der Workflow bleibt:

```text
inactive
draft
not-approved
```

---

## 40. Vertrauensgrenzen

Die Architektur unterscheidet vier Vertrauenszonen:

```text
Zone 1: ungeprüfter Eingang
Zone 2: geprüfte Einzelwerte
Zone 3: bereinigtes Ergebnis des lokalen Repository-Preflights
Zone 4: kanonischer interner Auftrag und simuliertes Patch
Zone 5: dauerhaft geschlossene Write Boundary
Zone 6: bereinigtes öffentliches Ergebnis
```

Daten dürfen nur nach bestandener Prüfung in die nächste Zone übernommen werden.

Unbekannte Eingangsfelder erreichen Zone 4 nicht.

Der vollständige Dateiinhalt darf Zone 4 niemals erreichen.

---

## 41. Verantwortungsgrenzen der Kerndokumente

```text
SPECIFICATION_v0.2.0.md
→ Anforderungen, Regeln und Sicherheitsbedingungen

ARCHITECTURE_v0.2.0.md
→ Komponenten, Grenzen und technische Verantwortungen

FLOW_v0.2.0.md
→ konkrete Node-Reihenfolge und Fehlerpriorität

TESTS_v0.2.0.md
→ Nachweis der korrekten und seiteneffektfreien Umsetzung
```

Keines der Dokumente ersetzt die anderen.

Bei einem Widerspruch bleibt die Umsetzung gesperrt.

---

## 42. Architekturentscheidungen v0.2.0

Für diese Version sind verbindlich entschieden:

```text
Eingangsmodus: execution.mode
Öffentlicher Modus: mode
Controller: WF-0013
Controller-Status: prepared
Audit-Status: passed
Freigabe: manual_review
Owner: Jamoko112026
Repository: JaMoKo_Automation_OS
Ref: main
Test-Zielpfad-Allowlist: 02_WORKFLOWS/WF-0011_GitHub_Writer/README.md
Repository-Preflight: read-only, planned, not implemented
Working Tree: clean erforderlich
Write Boundary: dauerhaft geschlossen
Content-Encoding intern: utf-8
Maximale Inhaltsgröße: 100000 UTF-8-Bytes
Maximale Commit-Nachricht: 120 Zeichen
Patch angewendet: false
Workflow aktiv: false
Credentials: 0
End-Items: 1
```

---

## 43. Nichtziele von v0.2.0

WF-0011 v0.2.0 ist ausdrücklich kein:

- produktiver GitHub Writer,
- Datei-Writer,
- Commit-Service,
- Push-Service,
- GitHub-Client,
- Workflow-Orchestrator,
- Credential-Verwalter,
- Freigabesystem,
- Rollback-System,
- Deployment-System.

Die Version beweist ausschließlich die sichere Vorbereitung und Simulation eines Writer-Auftrags.

---

## 44. Abbruchkriterien

Die technische Prüfung wird sofort abgebrochen, wenn:

- eine Datei verändert wird,
- ein GitHub-Aufruf stattfindet,
- ein Netzwerkaufruf stattfindet,
- ein Commit erstellt wird,
- ein Push stattfindet,
- ein Credential eingebunden ist,
- ein automatischer Trigger aktiv ist,
- ein anderer Workflow gestartet wird,
- Code auf nicht freigegebene oder mutierende Datei-, Git-, Prozess- oder Systemfunktionen zugreift,
- der Sanitizer umgangen wird,
- ein roher technischer Fehler ausgegeben wird,
- nicht genau ein End-Item entsteht.

Der Workflow bleibt danach:

```text
inactive
draft
not-approved
```

Der Vorfall wird in `KNOWN_ISSUES.md` dokumentiert.

---

## 45. Architektur-Abnahmekriterien

Die Architektur gilt als umsetzungsbereit, wenn:

- sie der Specification entspricht,
- sie dem Flow nicht widerspricht,
- sie alle Pflichtfälle der Tests unterstützt,
- alle Ergebniswege den Sanitizer erreichen,
- genau ein Ausgang existiert,
- keine verbotenen Komponenten vorgesehen sind,
- keine Credentials vorgesehen sind,
- die Fehlerpriorität deterministisch umsetzbar ist,
- Patch-Simulation und Patch Guard getrennt sind,
- ein konformer lesender Repository-Preflight separat nachgewiesen ist,
- die Write Boundary dauerhaft geschlossen und nicht umgehbar ist,
- die vier Seiteneffektwerte immer `false` bleiben,
- die Prüfung der n8n-Datenhaltung dokumentiert ist.

---

## 46. Aktueller Architekturstatus

```text
Workflow: WF-0011 – GitHub Writer
Version: 0.2.0
Status: draft
Mode: simulation
Architecture: defined
Implementation: not-started
Workflow active: false
Credentials: forbidden
GitHub access: forbidden
Network access: forbidden
Read-only local repository preflight: planned, not implemented
File changes: forbidden
Commits: forbidden
Pushes: forbidden
External side effects: forbidden
Approval: not-approved
```

---

## 47. Nächster konkreter Schritt

```text
SPECIFICATION_v0.2.0.md,
ARCHITECTURE_v0.2.0.md,
FLOW_v0.2.0.md und
TESTS_v0.2.0.md gemeinsam auf Widersprüche prüfen.

Anschließend muss der ausschließlich lesende Repository-Preflight technisch
evaluiert werden. Erst nach seinem dokumentierten Konformitätsnachweis darf der
inaktive n8n-Workflow WF-0011 v0.2.0 angelegt werden.
```
