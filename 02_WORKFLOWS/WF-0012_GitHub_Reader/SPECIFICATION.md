# WF-0012 – GitHub Reader

## Specification

Version: `v0.1.1`
Status: `released`
Veröffentlichungsdatum: `2026-08-08`
Typ: `Reader`

---

## 1. Zweck

WF-0012 liest eine definierte Datei aus einem freigegebenen GitHub-Repository.

Der Workflow stellt den aktuellen Dateiinhalt sowie den zugehörigen GitHub-SHA für nachfolgende Prüf- und Simulationsprozesse bereit.

WF-0012 bildet damit die lesende Grundlage der Repository Automation.

---

## 2. Ziel

Der Workflow soll:

1. eine eindeutig definierte Repository-Datei abrufen,
2. den aktuellen Dateiinhalt bereitstellen,
3. den aktuellen Datei-SHA ermitteln,
4. das Ergebnis in die vertraglich festgelegte JSON-Struktur abbilden, ohne
   Zielwerte, SHA oder Dateiinhalt zu verändern,
5. Fehler kontrolliert und nachvollziehbar zurückgeben,
6. keinerlei Änderungen am Repository ausführen.

---

## 3. Einsatzbereich

WF-0012 wird eingesetzt, wenn ein nachfolgender Workflow den tatsächlichen Stand einer Repository-Datei benötigt.

Typische Verbraucher sind:

- WF-0010 – Object Auditor
- WF-0011 – GitHub Writer
- zukünftige Vergleichs-, Prüf- und Synchronisationsworkflows

---

## 4. Abgrenzung

WF-0012:

- erstellt keine Dateien,
- verändert keine Dateien,
- löscht keine Dateien,
- erzeugt keine Commits,
- führt keinen Push aus,
- erstellt keine Branches,
- führt keinen Merge aus,
- schreibt keine Daten nach GitHub zurück.

Der Workflow besitzt ausschließlich lesenden Zugriff.

---

## 5. Eingabe

Die Eingabe erfolgt als JSON-Objekt.

### Pflichtfelder

```json
{
  "owner": "Jamoko112026",
  "repository": "JaMoKo_Automation_OS",
  "path": "01_REGISTRY/workflow_registry.md",
  "ref": "main"
}
```

Die vier Pflichtfelder sind Strings und dürfen nach der Eingangsprüfung nicht leer
sein.

| Feld | Bedeutung |
|---|---|
| `owner` | GitHub-Owner des freigegebenen Ziel-Repositories |
| `repository` | Name des freigegebenen Ziel-Repositories |
| `path` | relativer Pfad genau einer Datei im Repository |
| `ref` | auszulesende Git-Referenz |

Eine `request_id` oder `correlation_id` gehört nicht zum Eingangsvertrag von
WF-0012. Insbesondere darf WF-0012 eine interne `correlation_id` von WF-0013 weder
erfassen noch speichern, weiterreichen oder ausgeben. Die Zuordnung zwischen
Auftrag und Ergebnis bleibt Verantwortung des aufrufenden Workflows.

---

## 6. Eingangs- und Zielprüfung

WF-0012 prüft vor dem GitHub-Zugriff:

- Vorhandensein und String-Datentyp aller Pflichtfelder,
- nicht leere Werte,
- `owner` und `repository` gegen die konfigurierte Allowlist,
- `path` als sicheren relativen Repository-Pfad,
- das Vorhandensein einer nicht leeren `ref`.

Ungültige Eingaben oder nicht freigegebene Ziele erreichen den GitHub Read
Adapter nicht.

---

## 7. Lese- und Dekodierungsregeln

WF-0012 liest genau eine Datei über die GitHub Contents API. Eine erfolgreiche
GitHub-Antwort muss eine Datei, einen Dateipfad, einen Datei-SHA, Base64-kodierten
Inhalt und das Encoding `base64` liefern. Verzeichnisantworten oder strukturell
unvollständige Antworten sind kein erfolgreiches Leseergebnis.

Der Base64-dekodierte Byteinhalt muss strikt und verlustfrei als UTF-8 dekodierbar
sein. Ungültige Bytefolgen oder Ersatzzeichen, die durch eine verlustbehaftete
Dekodierung entstehen würden, sind unzulässig und führen zu
`CONTENT_DECODE_FAILED`. `file.content` enthält anschließend den vollständigen
dekodierten Dateiinhalt. Ein gültiger leerer Dateiinhalt ist zulässig.

WF-0012 verändert den dekodierten Inhalt nicht. Insbesondere sind verboten:

- Trimmen,
- BOM-Ergänzung oder -Entfernung,
- Unicode-Normalisierung,
- Änderung von Zeilenenden,
- Ergänzung oder Entfernung eines finalen Zeilenumbruchs,
- sonstige inhaltliche Normalisierung.

Eine vorhandene UTF-8-BOM, Zeilenenden und alle anderen gültig dekodierten Zeichen
bleiben byteinhaltlich unverändert repräsentiert. Binärdateien und Byteinhalte,
die nicht strikt und verlustfrei als UTF-8 dekodierbar sind, liegen außerhalb des
Erfolgsvertrags von `v0.1.1` und werden mit `CONTENT_DECODE_FAILED` abgelehnt.

---

## 8. Verbindlicher Ausgabevertrag

WF-0012 liefert pro Ausführung genau ein JSON-Objekt. Erfolg und Fehler sind
anhand von `status` eindeutig und disjunkt:

| Ergebnisart | `status` | `source` | `file` | `error` |
|---|---|---|---|---|
| Erfolg | `read` | verpflichtend | verpflichtend | unzulässig |
| Fehler | `rejected` | unzulässig | unzulässig | verpflichtend |

Für beide Ergebnisarten gelten außerdem:

```text
mode = read-only
writeRequested = false
writeExecuted = false
commitCreated = false
pushExecuted = false
```

Andere Status- oder Moduswerte gehören nicht zu diesem Vertrag. Ein Objekt, das
die Bedingungen seiner Ergebnisart nicht vollständig erfüllt oder Erfolg und
Fehler vermischt, ist keine gültige WF-0012-Ausgabe.

### 8.1 Erfolgsergebnis

Das verbindliche Erfolgsschema lautet:

```json
{
  "status": "read",
  "mode": "read-only",
  "source": {
    "owner": "Jamoko112026",
    "repository": "JaMoKo_Automation_OS",
    "ref": "main"
  },
  "file": {
    "path": "01_REGISTRY/workflow_registry.md",
    "sha": "0123456789abcdef0123456789abcdef01234567",
    "encoding": "base64",
    "content": "Vollständiger dekodierter Dateiinhalt"
  },
  "writeRequested": false,
  "writeExecuted": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

Folgende Felder sind bei `status = read` garantiert:

| Feld | Garantie |
|---|---|
| `source.owner` | entspricht exakt dem validierten angefragten `owner` |
| `source.repository` | entspricht exakt dem validierten angefragten `repository` |
| `source.ref` | entspricht exakt der validierten angefragten `ref` |
| `file.path` | stammt aus der GitHub-Dateiantwort und entspricht exakt dem validierten angefragten `path` |
| `file.sha` | zur gelesenen Datei gehörender GitHub-Datei-SHA als String, der exakt `^[a-fA-F0-9]{40}$` erfüllt |
| `file.encoding` | exakt `base64`; bezeichnet das Encoding der gelesenen GitHub-Antwort, nicht den Zustand von `file.content` |
| `file.content` | vollständiger, bereits dekodierter und danach unveränderter UTF-8-Dateiinhalt als String |

`source.owner`, `source.repository`, `source.ref` und `file.path` bilden die
garantierten Zielmetadaten. Nur diese Felder dürfen Verbraucher für einen
Zielvergleich verwenden. `file.sha` und `file.content` gehören immer zu genau
diesem erfolgreich validierten Ziel und Dateistand.

Die GitHub-Antwort kann zusätzlich `file.name` und `file.size` bereitstellen.
Diese Felder werden in `v0.1.1` nicht als für Verbraucher verpflichtende oder
vertragsrelevante Felder garantiert. Insbesondere darf WF-0013 seine
Reader-Validierung, Zielprüfung, SHA-Prüfung oder Inhaltsprüfung nicht von ihnen
abhängig machen.

Ein fehlender, leerer oder nicht dem Muster `^[a-fA-F0-9]{40}$` entsprechender
`file.sha` verhindert die Erfolgsbildung und führt zu
`READ_VALIDATION_FAILED`. Ein solcher Wert darf WF-0013 nicht als aktueller SHA
übergeben werden und dort keinen `SHA_CONFLICT` auslösen.

### 8.2 Fehlerergebnis

Das verbindliche Fehlerschema lautet:

```json
{
  "status": "rejected",
  "mode": "read-only",
  "error": {
    "code": "GITHUB_API_ERROR",
    "message": "Bereinigte Fehlermeldung"
  },
  "writeRequested": false,
  "writeExecuted": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

`error.code` und `error.message` sind verpflichtende, nicht leere Strings. Die
Meldung muss bereinigt sein und darf keine Credentials, Header, Stack-Traces oder
ungefilterten GitHub-Antworten enthalten.

Für Fehler gelten folgende Klassen:

| Klasse | Fehlercodes | Bedeutung |
|---|---|---|
| Eingangs- und Zielablehnung | `INPUT_INVALID`, `REF_MISSING`, `OWNER_NOT_ALLOWED`, `REPOSITORY_NOT_ALLOWED`, `PATH_INVALID` | Auftrag ist formal ungültig oder nicht freigegeben; kein GitHub-Leseerfolg wird behauptet |
| Nicht gefunden | `FILE_NOT_FOUND` | angeforderte Datei oder Referenz wurde von GitHub nicht gefunden |
| Technischer Reader-/GitHub-Fehler | `AUTHENTICATION_FAILED`, `ACCESS_DENIED`, `GITHUB_API_ERROR`, `CONTENT_DECODE_FAILED` | Authentifizierung, Zugriff, GitHub-Aufruf oder Dekodierung ist technisch fehlgeschlagen |
| Ungültige GitHub-/Reader-Antwort | `READ_VALIDATION_FAILED` | die GitHub-Antwort oder das intern erzeugte Leseergebnis erfüllt den erwarteten Reader-Erfolgsvertrag nicht |

Jedes Fehlerergebnis muss `source` und `file` vollständig auslassen. Damit
behauptet kein Fehler eine bestätigte Zielidentität, einen Datei-SHA oder einen
Dateiinhalt. Das gilt insbesondere für `FILE_NOT_FOUND`,
`READ_VALIDATION_FAILED`, `CONTENT_DECODE_FAILED` und alle technischen
GitHub-Fehler.

Ein formal ungültiges WF-0012-Ausgabeobjekt ist selbst kein neuer WF-0012-
Fehlercode. Ein Verbraucher muss es an seiner Eingangsgrenze als ungültiges
Reader-Ergebnis behandeln und darf daraus weder Ziel, SHA noch Inhalt ableiten.

---

## 9. Inhalts- und Metadatengrenzen

WF-0012 garantiert ausschließlich die in Abschnitt 8 genannten Felder. Der
Vertrag behauptet insbesondere keine weiteren GitHub-Felder wie URLs, Node-IDs,
Download-Links, Commit-Daten, Branch-Metadaten oder Benutzerinformationen.

Der Reader:

- verändert keine fachlichen Zielwerte, keinen SHA und keinen Dateiinhalt für
  nachgelagerte Vergleiche; ausschließlich die JSON-Struktur wird vertraglich
  gebildet,
- erzeugt keine Writer-Felder und trifft keine Annahmen über einen
  WF-0011-Eingangsvertrag,
- übernimmt keine Freigabeentscheidung,
- erzeugt keinen Writer-Payload,
- gibt keine externe oder interne Korrelationskennung aus.

---

## 10. Kompatibilität mit WF-0013

Ein zu diesem Vertrag kompatibles WF-0012-Erfolgsergebnis erlaubt WF-0013:

1. die Ergebnisart eindeutig über `status = read` festzustellen,
2. das Ziel ausschließlich über `source.owner`, `source.repository`,
   `source.ref` und `file.path` exakt mit dem angefragten Ziel zu vergleichen,
3. den aktuellen GitHub-Datei-SHA eindeutig aus `file.sha` zu entnehmen,
4. den vollständigen dekodierten und unveränderten Inhalt eindeutig aus
   `file.content` zu entnehmen,
5. SHA und Inhalt erst nach erfolgreicher Validierung des gesamten
   Erfolgsergebnisses zu vergleichen.

Fehlt eines der garantierten Erfolgsfelder oder besitzt es nicht den festgelegten
Typ beziehungsweise Wert, liegt kein kompatibles Erfolgsergebnis vor. Fehlende
Zielmetadaten dürfen weder als Zielübereinstimmung noch als Zielabweichung
interpretiert werden.

WF-0012 kennt die interne `correlation_id` von WF-0013 nicht. WF-0013 muss die
Zuordnung zum Reader-Auftrag ausschließlich in seinem eigenen Ausführungskontext
führen.

---

## 11. Sicherheitsanforderungen

Für jeden Ausführungspfad gelten:

1. keine Schreiboperation auf GitHub,
2. kein Commit und kein Push,
3. keine automatische Übergabe an WF-0011,
4. keine Ausgabe von Credentials oder vollständigen HTTP-Headern,
5. keine ungefilterte Ausgabe technischer GitHub- oder n8n-Fehler,
6. alle vier Schreibschutzwerte sind exakt `false`.

---

## 12. Versionsgrenze

Der veröffentlichte Dokumentvertrag lautet:

```text
WF-0012 v0.1.1
```

Der Dokumentvertrag von WF-0012 v0.1.1 wurde am `2026-08-08` veröffentlicht.
Die technische Export-Implementierung und ihr Konformitätsnachweis gegenüber
diesem Vertrag sind separat offen. Solange dieser Nachweis fehlt, darf der
Vertrag nicht als vollständig operativ umgesetzt bezeichnet werden.

Neue Statuswerte, andere Pflichtfelder, eine Änderung der garantierten
Zielmetadaten, eine Inhaltsumformung oder eine Writer-Kopplung erfordern eine
versionierte Vertragsänderung und einen Changelog-Eintrag.
