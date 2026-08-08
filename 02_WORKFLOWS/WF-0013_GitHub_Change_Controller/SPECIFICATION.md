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

Er prüft einen Änderungsauftrag gegen den aktuellen Dateistand und kontrolliert
anschließend, ob ein kompatibler veröffentlichter Vertrag für eine spätere
Übergabe an WF-0011 vorliegt. Mit den aktuell veröffentlichten Verträgen wird kein
Writer-Auftrag erzeugt.

---

## 2. Verantwortlichkeit

WF-0013 ist verantwortlich für:

- Entgegennahme eines strukturierten Änderungsauftrags
- Validierung aller Pflichtfelder
- Prüfung des Ziel-Repositories gegen eine Allowlist
- unveränderte Validierung des Dateipfads
- Vorbereitung eines Leseauftrags für WF-0012
- Prüfung des von WF-0012 gelieferten Dateistands
- Vergleich von erwartetem und tatsächlichem SHA
- Vergleich von aktuellem und vorgeschlagenem Inhalt
- Erkennung unveränderter oder widersprüchlicher Aufträge
- Prüfung einer ausdrücklichen Boolean-Freigabe
- Prüfung der vertraglichen Grenze vor einer späteren Writer-Übergabe
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
- Das Writer-Vertragsgate wird geprüft; ein Writer-Payload ist mit den aktuell
  veröffentlichten Verträgen nicht erreichbar.
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

Eine externe `request_id` ist in `v0.1.0` optional. Fehlt sie, erzeugt WF-0013
eine interne `correlation_id`.

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
- wird im Zustandsvergleich mit dem aktuellen Inhalt verglichen

Ein leerer String ist zulässig. Er beschreibt den vollständigen gewünschten
Dateiinhalt und ermöglicht damit das kontrollierte Leeren einer Datei. Auch in
diesem Fall bleibt `proposed_content` unverändert und unterliegt den übrigen
Prüfungen.

### 6.8 `change.commit_message`

Anforderungen:

- String
- nicht leer
- innerhalb der festgelegten Maximallänge
- keine Zeilenumbrüche, sofern WF-0011 diese nicht ausdrücklich unterstützt
- keine Steuerzeichen
- keine Tokens oder Zugangsdaten
- wird lokal validiert, begründet aber ohne kompatiblen veröffentlichten
  WF-0011-Vertrag keine Writer-Kompatibilität

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

Fehlt die externe `request_id`, erzeugt WF-0013 eine interne `correlation_id`.
Sie ist niemals eine `request_id`, dient ausschließlich der lokalen Verarbeitung
und Nachvollziehbarkeit innerhalb von WF-0013 und darf weder an WF-0012 noch an
WF-0011 übergeben werden.

Die konkrete Kombination aus Owner, Repository, Branch und freigegebenem Pfadbereich wird durch die Allowlist zur Laufzeit geprüft. Sie ist nicht Bestandteil des statischen Eingangsschemas.

---

## 7. Normalisierung

Die Normalisierung beginnt erst, nachdem Eingangsstruktur und String-Datentypen
geprüft wurden. Sie darf weder Typen konvertieren noch fehlende Pflichtfelder
ergänzen. Dadurch können beispielsweise Zahlen, Boolean-Werte oder `null` nicht
durch Normalisierung zu gültigen Textwerten werden.

Für `v0.1.0` gelten ausschließlich die folgenden Normalisierungsregeln:

1. Bei `target.owner`, `target.repository` und `target.branch` werden nur führende
   und nachgestellte ASCII-Leerzeichen `U+0020` entfernt. Andere Leerraumzeichen,
   Steuerzeichen, Groß-/Kleinschreibung und der innere Wert bleiben unverändert.
   **Begründung:** Die drei Werte werden als technische Bezeichner an WF-0012
   weitergegeben; eine eng begrenzte Randbereinigung verhindert mehrdeutige
   Zielwerte, ohne einen anderen Owner, ein anderes Repository oder einen anderen
   Branch abzuleiten.
2. Bei `change.commit_message` werden nur führende und nachgestellte
   ASCII-Leerzeichen `U+0020` entfernt. Die Nachricht wird anschließend gegen die
   Längen- und Zeichenregeln aus Abschnitt 6 geprüft; ihr innerer Inhalt wird nicht
   verändert. **Begründung:** So ist die lokale Leerheits- und Längenprüfung
   eindeutig, ohne eine Commit-Nachricht zu erfinden oder inhaltlich umzuschreiben.
3. Aus den nach Regel 1 validierten Zielwerten wird einmalig ein kanonisches
   Zielobjekt gebildet. Genau dessen Werte werden für Allowlist, Reader-Auftrag,
   Prüfung des Reader-Ergebnisses und eine spätere Writer-Übergabe verwendet; eine
   erneute Normalisierung zwischen diesen Stufen ist unzulässig. **Begründung:**
   Alle Prüfungen beziehen sich damit auf dasselbe Ziel und können nicht durch
   stufenabhängige Umformungen auseinanderlaufen.
4. Für den Reader-Auftrag wird ausschließlich der Feldname abgebildet:
   `target.branch` wird mit unverändertem Wert zu `ref`; `owner`, `repository` und
   `path` werden unter ihren gleichnamigen Feldern übernommen. Diese Projektion
   fügt keine `request_id` hinzu. **Begründung:** Dies entspricht den vier in
   WF-0012 v0.1.0 dokumentierten Pflichtfeldern und hält die interne Korrelation
   außerhalb des Reader-Vertrags.
5. Fehlt die optionale externe `request_id`, wird eine davon unterscheidbare
   interne `correlation_id` erzeugt. Sie ist niemals eine `request_id`, muss als
   internes Feld eindeutig von einer externen `request_id` getrennt sein und darf
   weder an WF-0012 noch an WF-0011 übergeben werden. Eine vorhandene externe
   `request_id` wird nicht normalisiert, sondern unverändert gegen das Muster aus
   Abschnitt 6 geprüft.
   **Begründung:** Die Zuordnung des Reader-Ergebnisses bleibt möglich, ohne
   Herkunft oder Identität einer externen Kennung vorzutäuschen oder die Verträge
   von WF-0012 und WF-0011 zu erweitern.

Alle übrigen Werte bleiben exakt erhalten. Insbesondere gelten folgende Grenzen:

- `execution.mode`, `target.path`, `target.expected_sha`, `approval.approved` und
  eine vorhandene externe `request_id` werden nicht getrimmt, umkodiert oder in
  einen anderen Typ überführt.
- Groß-/Kleinschreibung wird nirgends vereinheitlicht. Insbesondere werden SHA,
  Owner, Repository, Branch beziehungsweise `ref` und `request_id` nicht in Groß-
  oder Kleinbuchstaben umgeschrieben.
- `target.path` wird weder URL-dekodiert noch werden Backslashes ersetzt,
  Punktsegmente aufgelöst, Schrägstriche zusammengezogen oder Unicode-Zeichen
  normalisiert. Ein nur durch eine solche Korrektur gültiger Pfad wird abgelehnt.
- `change.proposed_content` wird nach der Typprüfung als unveränderter Stringwert
  durch alle Stufen geführt. Verboten sind insbesondere Trimmen,
  Zeilenendenkonvertierung, Unicode-Normalisierung, BOM-Ergänzung oder -Entfernung,
  Einrückung, Ergänzung eines finalen Zeilenumbruchs und sonstige inhaltliche
  Änderungen. Größenprüfung und Vergleich dürfen den Wert nur lesen. Der Vergleich
  mit dem aus einem gültigen Reader-Ergebnis entnommenen aktuellen Inhalt erfolgt
  exakt; bei einer später vertraglich zulässigen Übergabe muss derselbe
  unveränderte Wert weitergereicht werden.
- Werte aus dem Reader-Ergebnis werden von WF-0013 nicht nachnormalisiert. Ziel,
  SHA und Inhalt müssen in der gelieferten Form validiert und exakt verglichen
  werden. Eine abweichende Form darf nicht passend gemacht werden.
- Eine Normalisierung darf weder eine Freigabe ableiten noch den erwarteten SHA
  ersetzen oder eine Commit-Nachricht erzeugen.

Die vorliegenden Verträge von WF-0012 v0.1.0 legen dessen vollständige
Erfolgsausgabe nicht fest. Ebenso akzeptiert der veröffentlichte Vertrag von
WF-0011 v0.1.0 keinen Vollinhalt-Writer-Auftrag mit `change.content` und
`commit_message`. WF-0013 darf diese Vertragslücken nicht durch weitere
Normalisierung, Feldableitung oder Umformung überbrücken. Bis zu einem kompatiblen,
dokumentierten Schnittstellenvertrag endet die sichere Verarbeitung an der jeweils
betroffenen Übergabegrenze.

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

`target.path` wird unverändert geprüft und muss innerhalb des freigegebenen
Repository-Bereichs liegen. Der Wert wird weder getrimmt, Unicode-normalisiert,
kanonisiert noch anderweitig verändert.

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
  "owner": "Jamoko112026",
  "repository": "JaMoKo_Automation_OS",
  "path": "02_WORKFLOWS/example.md",
  "ref": "main"
}
```

Die Werte werden ausschließlich aus dem einmalig normalisierten und anschließend
validierten Zielobjekt projiziert. Der Auftrag muss genau die von WF-0012 v0.1.0
veröffentlichten Pflichtfelder `owner`, `repository`, `path` und `ref` enthalten.

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

Der veröffentlichte Vertrag von WF-0012 v0.1.0 sagt zu, den aktuellen
Dateiinhalt und den zugehörigen Datei-SHA in einer normalisierten Erfolgs- oder
Fehlerausgabe bereitzustellen. Er legt jedoch weder das vollständige JSON-Schema
noch konkrete Feldnamen und Statuswerte dieser Ausgabe fest. WF-0013 darf daher
keine Felder wie `status`, `mode`, `source.*`, `file.*`, `encoding` oder `error`
als veröffentlichten WF-0012-Vertrag voraussetzen.

Die Reader-Prüfung erfolgt in dieser Reihenfolge:

1. Das Ergebnis wird über den internen Ausführungskontext genau dem zuvor
   erzeugten Reader-Auftrag zugeordnet. Eine dafür erzeugte `correlation_id`
   bleibt innerhalb von WF-0013; weder `correlation_id` noch `request_id` werden
   an WF-0012 übergeben oder von dessen Ergebnis erwartet.
2. `READER_FAILED` wird ausschließlich ausgegeben, wenn WF-0012 einen Fehler
   eindeutig als technischen Fehler meldet. Ein nicht validierbares oder nur
   uneindeutiges Ergebnis genügt dafür nicht.
3. `READER_RESULT_INVALID` wird ausgegeben, wenn das Ergebnis nicht anhand eines
   veröffentlichten kompatiblen WF-0012-Ausgabevertrags validiert werden kann oder
   wenn daraus der aktuelle Datei-SHA und der bereits dekodierte
   Dateiinhalt-String nicht eindeutig entnommen werden können. Es wird kein Wert
   abgeleitet oder nachnormalisiert.
4. Eine Zielübereinstimmung darf nur anhand solcher Zielmetadaten geprüft werden,
   die ein veröffentlichter Ausgabevertrag tatsächlich liefert. `TARGET_MISMATCH`
   wird ausschließlich bei einer nachweisbaren Abweichung dieser gelieferten
   Zielmetadaten vom angefragten Ziel ausgegeben. Fehlende, nicht vertraglich
   zugesicherte Zielmetadaten lösen niemals `TARGET_MISMATCH` aus.

Erst nach erfolgreichem Abschluss dieser Reader-Prüfung darf WF-0013 den
Reader-Vergleich in den Abschnitten 12 und 13 beginnen.

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
aus dem gültigen Reader-Ergebnis entnommener aktueller Datei-SHA
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
aus dem gültigen Reader-Ergebnis entnommener aktueller Dateiinhalt
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

Der Inhaltsvergleich muss deterministisch und exakt sein. Weder
`change.proposed_content` noch der gelieferte aktuelle Inhalt werden für den
Vergleich getrimmt, in ihren Zeilenenden verändert oder anderweitig
nachnormalisiert.

---

## 14. Freigabeprüfung

Die Freigabe wird erst ausgewertet, nachdem:

- Eingangsstruktur
- Betriebsmodus
- Ziel
- Pfad
- Allowlist
- Änderungsinhalt
- Reader-Auftrag
- Reader-Ergebnis
- Prüfung vertraglich garantierter Zielmetadaten, sofern solche geliefert werden
- SHA
- Inhaltsänderung

erfolgreich geprüft wurden. Fehlende, nicht vertraglich zugesicherte
Zielmetadaten lösen weder `TARGET_MISMATCH` aus noch gelten sie als implizite
Zielbestätigung oder Freigabe.

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

Nach erfolgreichem Zustandsvergleich und erfolgreicher Freigabeprüfung muss vor
jeder Writer-Vorbereitung ein kompatibler, veröffentlichter Vertrag von WF-0011
vorliegen.

Der veröffentlichte Vertrag von WF-0011 v0.1.0 erwartet einen feldbezogenen,
freigegebenen und auditierten Änderungsvorschlag mit unter anderem `objectId`,
`field`, `currentValue`, `approvedValue`, `approvalStatus`, `approvedBy` und
`auditStatus`. Der WF-0013-Eingang enthält diese Angaben nicht und beschreibt
stattdessen einen vollständigen neuen Dateiinhalt. WF-0013 darf die fehlenden
Writer-Felder nicht aus `proposed_content` ableiten oder erfinden.

Damit kann WF-0013 aus dem hier definierten Auftrag derzeit keinen zu WF-0011
v0.1.0 kompatiblen Writer-Auftrag erzeugen oder validieren. Bis ein kompatibler
Writer-Vertrag veröffentlicht ist, endet die Verarbeitung an dieser
Übergabegrenze; WF-0011 wird nicht aufgerufen und `proposed_content` bleibt
unverändert.

Ein ungültiger Writer-Auftrag führt zu:

```text
WRITER_REQUEST_INVALID
```

---

## 16. Entscheidungsreihenfolge

Die Prüfungen erfolgen als getrennte Gates in dieser Reihenfolge:

1. **Eingangsvalidierung:** WF-0013 prüft genau ein JSON-Objekt, die zulässigen
   Haupt- und Unterfelder, alle Pflichtfelder sowie deren ursprüngliche
   Datentypen. Fehlende Felder, unbekannte Felder, `null` an unzulässiger Stelle
   oder falsche Datentypen führen zu `INVALID_INPUT`. Insbesondere wird
   `proposed_content` dabei nur als String bestätigt und nicht verändert.
2. **Normalisierung:** Ausschließlich die Regeln aus Abschnitt 7 werden auf den
   typgeprüften Eingang angewendet. Die Normalisierung konvertiert keine Typen und
   repariert keine ungültigen Werte.
3. **Validierung des normalisierten Auftrags:** Zuerst wird
   `execution.mode` exakt geprüft; eine Abweichung führt zu `INVALID_MODE`.
   Anschließend werden die unveränderte externe `request_id` und
   `target.expected_sha` formal geprüft. Ungültige Werte führen zu
   `INVALID_INPUT`.
4. **Ziel- und Pfadprüfung:** Owner, Repository und Branch werden formal geprüft.
   Ein unsicherer oder formal ungültiger Pfad führt zu `INVALID_PATH`; sonstige
   formal ungültige Zielwerte führen zu `INVALID_INPUT`.
5. **Allowlist-Prüfung:** Erst der formal gültige, normalisierte Zielwert wird als
   vollständige Kombination aus Owner, Repository, Branch, Pfadbereich und
   Dateiendung gegen die konfigurierte Allowlist geprüft. Eine Abweichung führt zu
   `TARGET_NOT_ALLOWED`. Vor erfolgreichem Abschluss dieses Gates wird kein
   Inhalt geprüft und kein Reader-Auftrag erzeugt.
6. **Inhaltsprüfung:** Der unveränderte `proposed_content`-String und die
   normalisierte Commit-Nachricht werden gegen die Regeln aus Abschnitt 6 geprüft.
   Ein leerer `proposed_content`-String ist gültig. Sonstige ungültige Inhalte
   führen zu `INVALID_CHANGE`, eine Überschreitung des Größenlimits zu
   `CONTENT_TOO_LARGE`.
7. **Reader-Auftrag:** Aus dem geprüften Ziel werden exakt `owner`, `repository`,
   `path` und `ref` gebildet und gegen den veröffentlichten Eingang von WF-0012
   geprüft. Ein intern abweichender Auftrag führt zu `READER_REQUEST_INVALID`.
8. **Reader-Prüfung:** Die Fehlergrenzen aus Abschnitt 11 gelten exakt:
   `READER_FAILED` nur bei einem eindeutig technisch gemeldeten WF-0012-Fehler,
   `READER_RESULT_INVALID` bei fehlender vertraglicher Validierbarkeit oder nicht
   eindeutig entnehmbarem SHA beziehungsweise Inhalt und `TARGET_MISMATCH` nur bei
   nachweisbar abweichenden, vertraglich gelieferten Zielmetadaten. Fehlende, nicht
   zugesicherte Zielmetadaten lösen niemals `TARGET_MISMATCH` aus.
9. **SHA-Vergleich:** Erst nach erfolgreicher Reader-Prüfung wird
   `target.expected_sha` exakt mit dem aktuellen Reader-SHA verglichen. Eine
   Abweichung führt zu `SHA_CONFLICT`.
10. **Inhaltsvergleich:** Nur bei übereinstimmendem SHA wird der unveränderte
    `change.proposed_content`-String exakt mit dem aktuellen Reader-Inhalt
    verglichen. Identische Inhalte führen zu `NO_CHANGE`.
11. **Freigabeprüfung:** Nur nach einer tatsächlichen Inhaltsänderung wird
   `approval.approved` ausgewertet. Ausschließlich Boolean `true` ist gültig;
   andernfalls folgt `INVALID_APPROVAL`.
12. **Writer-Vertragsgate:** Erst nach allen vorherigen Gates darf die
    Kompatibilität mit einem veröffentlichten WF-0011-Vertrag geprüft werden. Für
    WF-0011 v0.1.0 besteht die in Abschnitt 15 dokumentierte Übergabegrenze;
    fehlende Writer-Felder dürfen nicht abgeleitet werden. Der Auftrag endet mit
    `WRITER_REQUEST_INVALID`. `WRITER_REQUEST_PREPARED` und der Status `prepared`
    bleiben bis zu einem kompatiblen veröffentlichten Vertrag unerreichbar.

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
decision
target
comparison
approval
writer_request
execution
error
```

Die Kennungsfelder sind bedingt:

- `request_id` darf nur enthalten sein, wenn der Eingang eine formal gültige
  externe `request_id` enthielt; ihr Wert bleibt unverändert.
- Fehlt eine externe `request_id`, darf die Ausgabe ausschließlich die intern
  bezeichnete `correlation_id` zur lokalen Nachvollziehbarkeit enthalten.
- `correlation_id` ist niemals eine `request_id` und wird weder an WF-0012 noch an
  WF-0011 übergeben.

Zulässige Statuswerte:

```text
rejected
failed
```

Der Status `prepared` ist für eine spätere kompatible Vertragsversion reserviert,
bleibt in `v0.1.0` mit den aktuell veröffentlichten Verträgen jedoch unerreichbar.

---

## 18. Unerreichbarer Vorbereitungsstatus

`WRITER_REQUEST_PREPARED`, `status = prepared` und ein konkreter
Vollinhalt-Writer-Payload sind in `v0.1.0` mit den aktuell veröffentlichten
Verträgen nicht zulässige Laufzeitergebnisse. Sie bleiben für eine spätere
Version mit kompatiblem veröffentlichtem WF-0011-Vertrag reserviert.

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

`WRITER_REQUEST_PREPARED` ist reserviert und mit den aktuell veröffentlichten
Verträgen in `v0.1.0` nicht erreichbar.

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

- Erreichen des Writer-Vertragsgates
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
- eine vorhandene externe `request_id` oder andernfalls die klar intern
  bezeichnete `correlation_id`
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
12. das Writer-Vertragsgate WF-0011 v0.1.0 als nicht kompatibel erkennt und keinen
    Writer-Payload erzeugt,
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

Mit den aktuell veröffentlichten Verträgen endet ein bis zum Writer-Vertragsgate
gelangter Auftrag mit `WRITER_REQUEST_INVALID`. `WRITER_REQUEST_PREPARED` und der
Status `prepared` bleiben in `v0.1.0` unerreichbar. Kein Ergebnis bestätigt einen
Schreibvorgang, Commit oder Push.
