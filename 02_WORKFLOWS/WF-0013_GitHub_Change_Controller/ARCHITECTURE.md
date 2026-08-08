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
- prüft die veröffentlichte Vertragsgrenze vor einer späteren Übergabe an WF-0011
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
- das Writer-Vertragsgate prüfen; mit den aktuell veröffentlichten Verträgen keinen
  Writer-Auftrag erzeugen
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
   WF-0012 liest, WF-0013 kontrolliert und WF-0011 verarbeitet ausschließlich
   Aufträge, die seinem veröffentlichten Vertrag entsprechen.

2. **Kein implizites Schreiben**
   Das Writer-Vertragsgate darf keinen Schreibvorgang auslösen. Mit den aktuell
   veröffentlichten Verträgen erzeugt WF-0013 keinen Writer-Auftrag.

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
   Jede Entscheidung muss über Status, Fehlercode und entweder eine vorhandene
   externe `request_id` oder eine klar intern bezeichnete `correlation_id`
   zuordenbar sein.

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
Writer-Vertragsgate
```

WF-0011 wird in `v0.1.0` nicht automatisch ausgeführt.

Mit den aktuell veröffentlichten Verträgen endet die Verarbeitung am
Writer-Vertragsgate, ohne einen Writer-Auftrag bereitzustellen.

---

## 4. Komponentenübersicht

WF-0013 besteht logisch aus den folgenden Komponenten:

| Komponente | Aufgabe |
|---|---|
| Input Receiver | Nimmt genau einen Änderungsauftrag entgegen |
| Schema Validator | Prüft Pflichtfelder, Datentypen und Struktur |
| Input Normalizer | Vereinheitlicht ausschließlich typgeprüfte Eingangswerte nach der Spezifikation |
| Target Guard | Prüft Owner, Repository, Branch und Dateipfad |
| Allowlist Gate | Prüft das formal gültige Ziel gegen die konfigurierte Allowlist |
| Content Gate | Prüft Inhalt und Commit-Nachricht, ohne `proposed_content` zu verändern |
| Reader Request Builder | Erstellt den Leseauftrag für WF-0012 |
| Reader Result Validator | Prüft die Antwort von WF-0012 |
| State Comparator | Vergleicht erwarteten und aktuellen Zustand |
| Approval Guard | Prüft die ausdrückliche Freigabe |
| Decision Engine | Ermittelt das deterministische Gesamtergebnis |
| Writer Contract Validator | Prüft die veröffentlichte Vertragsgrenze zu WF-0011 |
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

`commit_message` muss ein nicht leerer String sein und wird lokal validiert. Da
WF-0011 v0.1.0 dieses Feld nicht als Eingang definiert, begründet die Prüfung
keine Writer-Kompatibilität.

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

Eine externe `request_id` dient der Zuordnung und Nachvollziehbarkeit. Sie wird nur
weitergeführt, wenn sie im Eingang vorhanden und formal gültig ist.

Fehlt die externe `request_id`, erzeugt WF-0013 eine interne `correlation_id`.
Diese ist niemals eine `request_id` und bleibt eindeutig als internes Feld
gekennzeichnet.

Die `correlation_id` stellt keine fachliche Freigabe dar und wird weder an WF-0012
noch an WF-0011 übergeben.

---

## 6. Validierungsschicht

Die Validierung erfolgt in einer festen Reihenfolge. Kein Gate darf Daten an das
nächste Gate weitergeben, wenn seine Prüfung fehlgeschlagen ist.

### 6.1 Strukturprüfung

Geprüft werden:

- genau ein Eingangsobjekt
- zulässige Hauptbereiche
- vorhandene Pflichtfelder
- korrekte Datentypen
- keine unerwarteten Nullwerte
- keine unzulässigen Mehrfachobjekte

Diese Prüfung arbeitet auf dem unveränderten Eingang und findet vor der
Normalisierung statt. Sie darf keine Typkonvertierung durchführen.

### 6.2 Normalisierung

Der Input Normalizer übernimmt ausschließlich den struktur- und typgeprüften
Eingang. Er wendet nur die in Abschnitt 7 der Spezifikation festgelegten Regeln an
und bildet daraus genau einen normalisierten Auftrag. `proposed_content` wird dabei
nicht verändert.

### 6.3 Prüfung des normalisierten Auftrags

`execution.mode` muss exakt `prepare-only` entsprechen.

Ein anderer oder fehlender Modus beendet die Verarbeitung.

Anschließend werden die formalen Regeln für eine gegebenenfalls vorhandene externe
`request_id` und den SHA geprüft. Ziel und Pfad folgen im nächsten Gate; die
Inhaltsprüfung folgt erst nach der Allowlist.

### 6.4 Ziel- und Pfadprüfung

Der Target Guard prüft:

- formal gültigen Owner
- formal gültiges Repository
- formal gültigen Branch
- zulässigen Dateipfad
- zulässige Dateiendung
- Traversal-Versuche
- absolute Pfade
- leere Pfadsegmente
- Steuerzeichen
- unerwartete URL-Bestandteile

`target.path` wird ausschließlich validiert. Der Wert wird niemals getrimmt,
Unicode-normalisiert, kanonisiert oder anderweitig verändert. Eine Pfadreparatur
oder erneute Normalisierung findet nicht statt.

### 6.5 Allowlist-Prüfung

Erst nach der formalen Ziel- und Pfadprüfung vergleicht der Target Guard die
vollständige normalisierte Zielkombination mit der Allowlist. Ein nicht
freigegebenes Ziel erreicht den Reader Request Builder nicht.

### 6.6 Inhaltsprüfung

Geprüft werden:

- Datentyp von `proposed_content`
- konfiguriertes Größenlimit
- Datentyp und Länge der Commit-Nachricht
- unzulässige Steuerzeichen
- verbotene oder nicht unterstützte Änderungsarten

Binärdateien werden in `v0.1.0` nicht unterstützt.

Ein leerer `proposed_content`-String ist zulässig und beschreibt das kontrollierte
Leeren der vollständigen Datei. Alle Prüfungen behandeln `proposed_content`
unverändert.

### 6.7 Freigabeprüfung

Die Freigabe wird erst nach gültigem Reader-Ergebnis, SHA-Vergleich und
Inhaltsvergleich ausgewertet.

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

Eine externe `request_id` und eine interne `correlation_id` werden nicht an
WF-0012 übergeben. WF-0013 erhält die Zuordnung im internen Ausführungskontext.
Die `correlation_id` wird auch nicht an WF-0011 übergeben.

Der veröffentlichte WF-0012-Vertrag benennt noch kein vollständiges
Ausgabeschema. Er sagt lediglich den aktuellen Dateiinhalt, den Datei-SHA sowie
normalisierte Erfolgs- und Fehlerausgaben zu. Der Reader Result Validator darf
daher keine konkreten Ausgabe-Feldnamen, Statuswerte oder ein ausgegebenes
Encoding voraussetzen.

`READER_FAILED` gilt ausschließlich für einen von WF-0012 eindeutig als technisch
gemeldeten Fehler. Ein uneindeutiges oder nicht validierbares Ergebnis ist kein
`READER_FAILED`.

`READER_RESULT_INVALID` gilt, wenn das Ergebnis nicht anhand eines veröffentlichten
kompatiblen WF-0012-Ausgabevertrags validiert werden kann oder wenn daraus der
bereits dekodierte Inhalt und der zugehörige Datei-SHA nicht eindeutig entnommen
werden können. In diesem Fall findet kein Zustandsvergleich statt.

`TARGET_MISMATCH` gilt ausschließlich, wenn ein veröffentlichter Ausgabevertrag
Zielmetadaten liefert und diese nachweisbar vom angefragten Ziel abweichen.
Fehlende, nicht vertraglich zugesicherte Zielmetadaten lösen niemals
`TARGET_MISMATCH` aus.

---

## 8. Zustandsvergleich

Der State Comparator vergleicht den erwarteten, aktuellen und vorgeschlagenen Zustand.

### 8.1 SHA-Vergleich

Es gilt:

```text
target.expected_sha == validierter aktueller Reader-SHA
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
change.proposed_content == validierter aktueller Reader-Inhalt
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

kann die Verarbeitung zur Freigabeprüfung und anschließend zum
Writer-Vertragsgate fortgesetzt werden.

---

## 9. Entscheidungslogik

Die Decision Engine arbeitet deterministisch.

Die Fehlerpriorität lautet:

1. ungültige Eingangsstruktur
2. Normalisierung des typgeprüften Eingangs
3. ungültiger normalisierter Auftrag oder Betriebsmodus
4. formal ungültiges Ziel oder ungültiger Pfad
5. nicht erlaubtes Ziel
6. ungültiger Änderungsinhalt
7. ungültiger Leseauftrag
8. Reader-Fehlergrenzen in der Reihenfolge `READER_FAILED`,
   `READER_RESULT_INVALID`, `TARGET_MISMATCH`
9. SHA-Konflikt
10. unveränderter Inhalt
11. fehlende oder ungültige Freigabe
12. nicht kompatibler Writer-Auftrag

Die Reader-Fehlercodes werden dabei ausschließlich unter den in Abschnitt 7
definierten Voraussetzungen verwendet. Insbesondere lösen fehlende, nicht
vertraglich zugesicherte Zielmetadaten niemals `TARGET_MISMATCH` aus.

Mit den aktuell veröffentlichten Verträgen endet das Writer-Vertragsgate mit
`WRITER_REQUEST_INVALID`. Der Status `prepared` und
`WRITER_REQUEST_PREPARED` bleiben in `v0.1.0` unerreichbar.

Der Zustandsvergleich beginnt ausschließlich nach erfolgreichem Reader Result
Validator. Er vergleicht zuerst den SHA und nur bei Übereinstimmung den Inhalt.

---

## 10. Writer-Vertragsgate für WF-0011

Der Writer Contract Validator darf erst nach erfolgreicher Freigabeprüfung und nur
gegen einen kompatiblen, veröffentlichten Vertrag von WF-0011 arbeiten.

WF-0011 v0.1.0 erwartet einen feldbezogenen Änderungsvorschlag mit Objekt-ID,
Feld, aktuellem und freigegebenem Wert sowie dokumentierter Freigabe und Audit.
Der WF-0013-Auftrag beschreibt dagegen einen vollständigen Dateiinhalt und enthält
diese Pflichtangaben nicht.

Der Writer Contract Validator darf diese Angaben nicht aus `proposed_content`
ableiten. Daher endet die Verarbeitung mit den aktuell veröffentlichten Verträgen
an diesem Gate. Ein Writer-Auftrag kann erst nach Veröffentlichung eines
kompatiblen Vertrags validiert und als `prepared` ausgegeben werden. WF-0011 wird
in `v0.1.0` in keinem Fall automatisch aufgerufen.

---

## 11. Ausgangsmodell

WF-0013 liefert genau ein bereinigtes Ergebnisobjekt.

Verbindliche Hauptfelder:

```text
workflow
version
mode
status
decision
target
comparison
approval
writer_request
execution
error
```

Die Kennung ist bedingt: `request_id` darf nur bei einer formal gültigen externen
`request_id` im Eingang ausgegeben werden. Andernfalls darf die Ausgabe
ausschließlich die klar als intern bezeichnete `correlation_id` für die lokale
Nachvollziehbarkeit enthalten. `correlation_id` ist niemals eine `request_id` und
wird weder an WF-0012 noch an WF-0011 übergeben.

### 11.1 Unerreichbarer Vorbereitungsstatus

Ein Ergebnis mit `status = prepared`, `WRITER_REQUEST_PREPARED` oder einem
konkreten Vollinhalt-Writer-Payload ist mit den aktuell veröffentlichten Verträgen
in `v0.1.0` nicht erreichbar. Diese Werte bleiben für eine spätere Version mit
kompatiblem veröffentlichtem WF-0011-Vertrag reserviert.

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

`writer_request` enthält in `v0.1.0` mit den aktuell veröffentlichten Verträgen
keinen vollständigen Writer-Payload.

---

## 12. Sicherheitsarchitektur

### 12.1 Allowlist

Zulässige Kombinationen aus Owner, Repository, Branch und Pfadbereich werden zentral definiert.

Nicht gelistete Ziele werden abgelehnt.

### 12.2 Pfadsicherheit

Der unverändert validierte `target.path` darf nicht enthalten:

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
Validate Input Structure and Types
Normalize Typed Input
Validate Mode
Validate Normalized Request
Validate Target and Path
Validate Allowlist
Validate Change
Build Reader Request
Execute or Receive WF-0012 Result
Validate Reader Result
Compare Guaranteed Target Metadata If Present
Compare SHA
Compare Content
Validate Approval
Validate Published Writer Contract
Reject Incompatible Writer Transition
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
failed
```

`prepared` ist als möglicher Status einer späteren kompatiblen Vertragsversion
reserviert, gehört mit den aktuell veröffentlichten Verträgen aber nicht zu den
erreichbaren Zuständen von `v0.1.0`.

Zulässige Übergänge:

```text
received
  -> validated
  -> reader-request-prepared
  -> reader-result-received
  -> compared
  -> rejected
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

Nach `rejected` oder `failed` erfolgt kein weiterer Verarbeitungsschritt.

---

## 16. Invarianten

Folgende Bedingungen müssen immer gelten:

1. WF-0013 schreibt niemals selbst auf GitHub.
2. WF-0013 führt WF-0011 in `v0.1.0` niemals automatisch aus.
3. Mit den aktuell veröffentlichten Verträgen entsteht kein Writer-Auftrag.
4. Eine SHA-Abweichung führt vor dem Writer-Vertragsgate zur Ablehnung.
5. Ein identischer Inhalt führt vor dem Writer-Vertragsgate zur Ablehnung.
6. Ohne Boolean-Freigabe `true` wird das Writer-Vertragsgate nicht erreicht.
7. Ein unzulässiges Ziel wird vor der Reader-Verarbeitung abgelehnt.
8. Jede Ausgabe enthält `workflow`, `version`, `mode` und `status` sowie entweder
   die vorhandene formal gültige externe `request_id` oder die klar intern
   bezeichnete `correlation_id`.
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

WF-0013 prüft Zielmetadaten der Reader-Antwort nur, wenn ein veröffentlichter
kompatibler WF-0012-Ausgabevertrag diese tatsächlich garantiert liefert. Nur eine
nachweisbare Abweichung löst `TARGET_MISMATCH` aus. Fehlende, nicht zugesicherte
Zielmetadaten lösen weder `TARGET_MISMATCH` noch eine implizite Freigabe aus.

---

## 18. Abgrenzung zu WF-0011

WF-0011 ist verantwortlich für:

- Validierung des in v0.1.0 veröffentlichten feldbezogenen Writer-Auftrags
- Erzeugung einer Änderungsvorschau und eines validierten, nicht angewendeten
  Patches im Modus `simulation`
- Rückgabe eines bereinigten Simulationsergebnisses ohne Dateiänderung, Commit
  oder Push

WF-0011 v0.1.0 veröffentlicht keinen Vollinhalt-Writer-Vertrag. WF-0013 erzeugt
bis zu einem kompatiblen veröffentlichten Vollinhalt-Writer-Vertrag keinen
Writer-Payload.

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
7. das Writer-Vertragsgate WF-0011 v0.1.0 als nicht kompatibel erkennt und keinen
   Writer-Payload erzeugt,
8. kein Pfad WF-0011 automatisch ausführt,
9. alle Ausgaben zentral bereinigt werden,
10. sämtliche Seiteneffekt-Flags nachweislich `false` bleiben.

---

## 21. Architekturentscheidung

WF-0013 wird in `v0.1.0` als kontrollierender, seiteneffektfreier Orchestrierungsworkflow aufgebaut.

Er liest nicht selbst, schreibt nicht selbst und führt keinen Writer aus.

Mit den aktuell veröffentlichten Verträgen besitzt `v0.1.0` kein positives
Writer-Ergebnis. Ein validierter Writer-Auftrag setzt einen späteren kompatiblen,
veröffentlichten WF-0011-Vertrag voraus.

Damit bildet WF-0013 die Sicherheits- und Entscheidungsschicht zwischen Änderungsabsicht, aktuellem GitHub-Zustand und einer möglichen späteren Schreibverarbeitung.
