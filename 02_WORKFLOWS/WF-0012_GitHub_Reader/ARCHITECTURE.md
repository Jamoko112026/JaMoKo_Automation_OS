# WF-0012 – GitHub Reader

## Architecture

Version: `v0.1.1`
Status: `released`
Veröffentlichungsdatum: `2026-08-08`
Typ: `Reader`

---

## 1. Architekturziel

WF-0012 stellt einen kontrollierten, ausschließlich lesenden Zugriff auf eine definierte Datei in einem freigegebenen GitHub-Repository bereit.

Die Architektur trennt:

1. Eingabe,
2. Validierung,
3. Zugriffskontrolle,
4. GitHub-Lesezugriff,
5. Dekodierung,
6. Ergebnisvalidierung,
7. Bildung der vertraglich festgelegten JSON-Ausgabe.

Jede Anfrage muss entweder zu einem geprüften Leseergebnis oder zu einer kontrollierten Fehlerausgabe führen.

---

## 2. Architekturprinzipien

WF-0012 folgt diesen Prinzipien:

- Read-only by Design
- Validierung vor externem Zugriff
- Allowlist statt freier Repository-Auswahl
- eindeutige Referenz durch `ref`
- vertraglich strukturierte Erfolgs- und Fehlerausgaben
- keine Zugangsdaten in Workflow-Daten
- keine impliziten Schreiboperationen
- nachvollziehbare Fehlercodes
- explizite Schreibschutzwerte

---

## 3. Systemgrenze

WF-0012 verarbeitet genau eine Datei pro Ausführung.

Innerhalb der Systemgrenze liegen:

- Entgegennahme der Eingabe,
- Prüfung der Pflichtfelder,
- Prüfung von Owner und Repository,
- Prüfung des Dateipfads,
- lesender GitHub-API-Aufruf,
- Dekodierung des Datei­inhalts,
- Übernahme von Datei-SHA und Metadaten,
- Validierung des Leseergebnisses,
- Erzeugung der vertraglich festgelegten JSON-Ausgabe ohne Veränderung von
  Zielwerten, SHA oder Dateiinhalt.

Außerhalb der Systemgrenze liegen:

- Änderung von Repository-Inhalten,
- Erstellung von Commits,
- Push- und Merge-Operationen,
- Branch-Verwaltung,
- Pull Requests,
- Speicherung gelesener Inhalte,
- Verarbeitung mehrerer Dateien,
- automatische Weitergabe an WF-0011.

---

## 4. Beteiligte Systeme

| System | Aufgabe |
|---|---|
| Aufrufender Workflow oder manueller Test | Liefert die definierte Leseanfrage |
| n8n | Führt Validierung, API-Aufruf und Bildung der vertraglichen JSON-Ausgabe aus |
| n8n Credential Store | Verwaltet die GitHub-Zugangsdaten |
| GitHub API | Liefert Dateiinhalt, Datei-SHA und Metadaten |
| WF-0012 | Kontrolliert den Leseprozess und bildet die vertragliche JSON-Ausgabe |
| Nachfolgender Workflow | Kann das geprüfte Ergebnis später verwenden |

---

## 5. Komponenten

### 5.1 Input Receiver

Übernimmt das Eingabeobjekt mit:

- `owner`
- `repository`
- `path`
- `ref`

Der Input Receiver verändert die fachlichen Eingabewerte nicht.

---

### 5.2 Input Validator

Prüft:

- Vorhandensein aller Pflichtfelder,
- Datentypen,
- leere Werte,
- Vorhandensein von `ref`,
- grundlegende Struktur des Dateipfads.

Ungültige Eingaben werden vor dem GitHub-Zugriff abgelehnt.

---

### 5.3 Access Guard

Vergleicht:

- `owner` mit der freigegebenen Owner-Allowlist,
- `repository` mit der freigegebenen Repository-Allowlist.

Nicht freigegebene Ziele führen zu einer kontrollierten Ablehnung.

Der Access Guard verhindert, dass WF-0012 beliebige Repositorys auslesen kann.

---

### 5.4 Path Guard

Prüft den relativen Repository-Pfad.

Abgelehnt werden insbesondere:

- leere Pfade,
- absolute Pfade,
- Pfadüberschreitungen mit `../`,
- Pfade ohne eindeutig definiertes Ziel,
- technisch unzulässige Pfadwerte.

---

### 5.5 GitHub Read Adapter

Führt den ausschließlich lesenden GitHub-API-Aufruf aus.

Verwendete Parameter:

- Owner
- Repository
- Dateipfad
- Referenz

Der Adapter darf nur eine GitHub-Operation zum Abruf von Dateiinhalt und Dateimetadaten enthalten.

Schreibende GitHub-Operationen sind in dieser Komponente nicht zulässig.

---

### 5.6 Content Decoder

Dekodiert den von GitHub gelieferten Dateiinhalt.

Für `v0.1.1` wird eine GitHub-Antwort mit Base64-kodiertem Dateiinhalt erwartet.

Schlägt die Dekodierung fehl, wird der Prozess mit folgendem Fehlercode beendet:

```text
CONTENT_DECODE_FAILED
```

Der Decoder muss den Base64-dekodierten Byteinhalt strikt und verlustfrei als
UTF-8 dekodieren und den resultierenden String vollständig weiterreichen.
Ungültige Bytefolgen oder Ersatzzeichen durch eine verlustbehaftete Dekodierung
führen zu `CONTENT_DECODE_FAILED`. Eine vorhandene UTF-8-BOM, Zeilenenden und alle
anderen gültig dekodierten Zeichen bleiben byteinhaltlich unverändert
repräsentiert. Ein gültiger leerer Inhalt ist zulässig. Der Decoder darf weder
trimmen noch BOM, Unicode-Zeichen, Zeilenenden oder einen finalen Zeilenumbruch
verändern.

---

### 5.7 Read Result Validator

Der Read Result Validator prüft vor der Erfolgsbildung:

- die GitHub-Antwort beschreibt genau eine Datei,
- der zurückgegebene Pfad ist ein String und entspricht exakt dem validierten
  angefragten Pfad,
- der Datei-SHA ist als String vorhanden und entspricht exakt
  `^[a-fA-F0-9]{40}$`,
- der Dateiinhalt ist als Base64-String vorhanden,
- das angegebene Encoding ist exakt `base64`,
- der Byteinhalt wurde strikt, verlustfrei und ohne Ersatzzeichen als
  vollständiger UTF-8-String dekodiert.

Eine strukturell ungültige Antwort oder ein intern unvollständiges Leseergebnis
führt zu `READ_VALIDATION_FAILED`. Dazu gehört insbesondere ein fehlender, leerer
oder nicht dem Muster `^[a-fA-F0-9]{40}$` entsprechender Datei-SHA. Ein ungültiger
SHA darf nicht an WF-0013 gelangen und dort keinen `SHA_CONFLICT` auslösen. Eine
nicht strikt und verlustfrei mögliche UTF-8-Dekodierung führt zu
`CONTENT_DECODE_FAILED`. In beiden Fällen darf kein Erfolgsergebnis entstehen.

---

### 5.8 Result Builder

Der Result Builder erzeugt genau eine von zwei disjunkten Ausgaben:

- ein Erfolgsergebnis mit `status = read`, `source` und `file`, oder
- ein Fehlerergebnis mit `status = rejected` und `error`.

Erfolg und Fehler dürfen nicht miteinander vermischt werden. Der Result Builder
gibt keine Rohantwort der GitHub API aus.

---

### 5.9 Write Protection Guard

Der Write Protection Guard setzt auf jedem Erfolgs- und Fehlerpfad:

```text
writeRequested = false
writeExecuted = false
commitCreated = false
pushExecuted = false
```

Die Werte werden intern erzeugt und niemals aus der Eingabe übernommen.

---

## 6. Verarbeitungsreihenfolge

Die verbindliche Reihenfolge lautet:

```text
Input Receiver
  -> Input Validator
  -> Access Guard
  -> Path Guard
  -> GitHub Read Adapter
  -> Content Decoder
  -> Read Result Validator
  -> Result Builder
  -> Write Protection Guard
```

Eingangs-, Allowlist- und Pfadfehler zweigen vor dem GitHub Read Adapter in den
Fehlerpfad ab. GitHub-, Dekodierungs- und Ergebnisvalidierungsfehler zweigen vor
der Erfolgsbildung ab. Jeder Pfad endet beim Write Protection Guard.

---

## 7. Verbindliches Ausgangsmodell

WF-0012 gibt pro Ausführung genau ein JSON-Objekt zurück.

### 7.1 Erfolg

Ein Erfolg ist ausschließlich durch folgende Kombination gekennzeichnet:

```text
status = read
mode = read-only
source vorhanden
file vorhanden
error nicht vorhanden
```

Das Erfolgsergebnis garantiert:

```text
source.owner
source.repository
source.ref
file.path
file.sha
file.encoding = base64
file.content
writeRequested = false
writeExecuted = false
commitCreated = false
pushExecuted = false
```

Die Zielmetadaten besitzen folgende Herkunft:

| Ausgabefeld | Herkunft und Garantie |
|---|---|
| `source.owner` | validierter angefragter `owner`, unverändert übernommen |
| `source.repository` | validiertes angefragtes `repository`, unverändert übernommen |
| `source.ref` | validierte angefragte `ref`, unverändert übernommen |
| `file.path` | GitHub-Dateipfad, nach exakter Übereinstimmung mit dem validierten angefragten `path` |

Nur diese vier Felder sind vertraglich garantierte Zielmetadaten. `file.sha` ist
der zur validierten GitHub-Datei gehörende SHA. `file.content` ist der bereits
dekodierte vollständige Dateiinhalt derselben Datei.

`file.name` und `file.size` können aus der GitHub-Antwort übernommen werden, sind
aber in `v0.1.1` keine garantierten oder für Verbraucher vertragsrelevanten
Felder. Weitere GitHub-Felder werden nicht Teil des Ausgangsvertrags.

### 7.2 Fehler

Ein Fehler ist ausschließlich durch folgende Kombination gekennzeichnet:

```text
status = rejected
mode = read-only
error.code vorhanden
error.message vorhanden
source nicht vorhanden
file nicht vorhanden
```

Jeder Fehler enthält außerdem alle vier Schreibschutzwerte mit `false`.

Die Architektur unterscheidet:

| Fehlergrenze | Codes |
|---|---|
| Eingangs- oder Zielablehnung | `INPUT_INVALID`, `REF_MISSING`, `OWNER_NOT_ALLOWED`, `REPOSITORY_NOT_ALLOWED`, `PATH_INVALID` |
| Nicht gefundene Ressource | `FILE_NOT_FOUND` |
| Technischer Reader-/GitHub-Fehler | `AUTHENTICATION_FAILED`, `ACCESS_DENIED`, `GITHUB_API_ERROR`, `CONTENT_DECODE_FAILED` |
| Ungültige GitHub-Antwort oder internes Reader-Ergebnis | `READ_VALIDATION_FAILED` |

Kein Fehlerpfad darf `source`, `file`, `file.sha`, `file.content` oder sonstige
Zielmetadaten ausgeben. Ein Fehler bestätigt daher weder die Identität noch den
Zustand einer Datei. Rohe GitHub-Fehler, Header, Credentials und Stack-Traces
bleiben innerhalb der technischen Systemgrenze.

### 7.3 Strukturell ungültige WF-0012-Ausgabe

Ein Objekt, das weder dem Erfolgs- noch dem Fehlerschema vollständig entspricht,
ist kein gültiges WF-0012-Ergebnis. Es erhält nachträglich keinen erfundenen
WF-0012-Fehlercode. Ein Verbraucher muss es an seiner eigenen Vertragsgrenze
ablehnen und darf daraus keine Zielmetadaten, keinen SHA und keinen Inhalt
entnehmen.

---

## 8. Inhaltsintegrität

Zwischen Base64-Dekodierung und Ausgabe von `file.content` darf keine inhaltliche
Transformation stattfinden. Verboten sind insbesondere:

- Trimmen,
- BOM-Ergänzung oder -Entfernung,
- Unicode-Normalisierung,
- Konvertierung von Zeilenenden,
- Ergänzung oder Entfernung eines finalen Zeilenumbruchs,
- Formatierung oder sonstige Inhaltsnormalisierung.

Die strukturelle Normalisierung des Ergebnisobjekts betrifft ausschließlich die
festgelegte JSON-Form. Sie verändert weder Zielwerte noch Dateiinhalt.

---

## 9. Kopplung mit WF-0013

WF-0013 darf ein Erfolgsergebnis erst nach vollständiger Prüfung des in Abschnitt
7.1 definierten Vertrags verarbeiten. Danach kann WF-0013:

- das Ziel über `source.owner`, `source.repository`, `source.ref` und `file.path`
  exakt prüfen,
- den aktuellen SHA eindeutig aus `file.sha` entnehmen,
- den dekodierten Inhalt eindeutig aus `file.content` entnehmen,
- SHA und Inhalt ohne Nachnormalisierung exakt vergleichen.

Fehlt eine der garantierten Zielmetadaten, liegt kein gültiges Erfolgsergebnis
vor. Das Fehlen darf weder als `TARGET_MISMATCH` noch als implizite
Zielübereinstimmung interpretiert werden. Eine Zielabweichung kann nur anhand der
vier ausdrücklich garantierten und tatsächlich gelieferten Zielmetadaten
festgestellt werden.

WF-0012 akzeptiert, erzeugt und transportiert keine interne
WF-0013-`correlation_id`. Eine `request_id` gehört ebenfalls nicht zum
WF-0012-Vertrag. Die Korrelation bleibt vollständig außerhalb der
Reader-Systemgrenze.

---

## 10. Abgrenzung zu WF-0011

WF-0012 trifft keine Annahmen über Eingabe, Ausgabe, Betriebsmodus oder
Feldstruktur von WF-0011. Insbesondere:

- erzeugt WF-0012 keinen Writer-Payload,
- leitet WF-0012 keinen Dateiinhalt automatisch an WF-0011 weiter,
- interpretiert WF-0012 keine Freigabedaten,
- löst WF-0012 keinen Writer-Aufruf aus.

---

## 11. Sicherheitsinvarianten

Für jeden Pfad gelten:

1. Es wird höchstens eine Datei gelesen.
2. Es findet keine schreibende GitHub-Operation statt.
3. Allowlist und Pfadprüfung liegen vor dem GitHub-Zugriff.
4. Ein ungültiges Leseergebnis wird niemals als Erfolg ausgegeben.
5. Ein Fehler enthält keine Zielmetadaten, keinen SHA und keinen Dateiinhalt.
6. Alle Schreibschutzwerte sind exakt `false`.
7. Credentials und ungefilterte technische Antworten verlassen die Systemgrenze
   nicht.

---

## 12. Versionsgrenze

Diese Architektur beschreibt den veröffentlichten Ausgabevertrag von:

```text
WF-0012 v0.1.1
```

Der Dokumentvertrag von WF-0012 v0.1.1 wurde am `2026-08-08` veröffentlicht.
Die technische Export-Implementierung und ihr Konformitätsnachweis gegenüber
diesem Vertrag sind separat offen. Solange dieser Nachweis fehlt, darf der
Vertrag nicht als vollständig operativ umgesetzt bezeichnet werden.

Änderungen an Statuswerten, Pflichtfeldern, garantierten Zielmetadaten,
Inhaltsbehandlung oder Fehlergrenzen erfordern eine versionierte Aktualisierung
von Spezifikation, Architektur, Flow, Tests und Changelog.
