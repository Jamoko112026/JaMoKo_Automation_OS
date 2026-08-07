# WF-0013 – Interface Contract

Version: `v0.1.0`
Status: `draft`
Typ: `Schnittstellenvertrag`
Betriebsmodus: `prepare-only`

---

## 1. Zweck

Diese Datei definiert die verbindlichen Schnittstellen zwischen:

- `WF-0012 – GitHub Reader`
- `WF-0013 – GitHub Change Controller`
- `WF-0011 – GitHub Writer`

Sie dokumentiert:

- bestätigte kompatible Felder
- bestehende Vertragsabweichungen
- notwendige Feldzuordnungen
- den Zielvertrag für WF-0011
- die Bedingungen für eine spätere technische Kopplung

WF-0013 darf erst implementiert und freigegeben werden, wenn die hier beschriebenen Verträge technisch bestätigt sind.

---

## 2. Systemgrenzen

```text
Änderungsauftrag
      │
      ▼
WF-0013 – Change Controller
      │
      ├── Leseauftrag an WF-0012
      │
      ├── Prüfung von Ziel, Pfad, SHA, Inhalt und Freigabe
      │
      └── Vorbereitung eines Writer-Payloads
                              │
                              ▼
                    keine automatische Übergabe
                              │
                              ▼
                  WF-0011 – GitHub Writer
```

WF-0013 führt in `v0.1.0` weder WF-0012 noch WF-0011 automatisch aus.

---

## 3. Vertragsgrundlagen

Der Schnittstellenvertrag basiert auf den folgenden technisch vorhandenen Artefakten:

### WF-0012

```text
02_WORKFLOWS/WF-0012_GitHub_Reader/
└── exports/WF-0012_GitHub_Reader_v0.1.0.json
```

Der n8n-Export ist für den tatsächlich implementierten Ein- und Ausgangsvertrag maßgeblich.

### WF-0011

```text
02_WORKFLOWS/WF-0011_GitHub_Writer/
├── SPECIFICATION_v0.2.0.md
├── ARCHITECTURE_v0.2.0.md
├── FLOW_v0.2.0.md
└── TESTS_v0.2.0.md
```

Die Dokumente der Version `v0.2.0` bilden den Zielvertrag für den vorbereiteten Writer-Auftrag.

---

## 4. WF-0013 zu WF-0012 – Reader-Auftrag

WF-0013 bereitet für WF-0012 folgendes Eingangsobjekt vor:

```json
{
  "owner": "Jamoko112026",
  "repository": "JaMoKo_Automation_OS",
  "path": "01_REGISTRY/workflow_registry.md",
  "ref": "main"
}
```

### 4.1 Feldzuordnung

| WF-0013 | WF-0012 | Behandlung |
|---|---|---|
| `target.owner` | `owner` | direkte Übernahme |
| `target.repository` | `repository` | direkte Übernahme |
| `target.path` | `path` | direkte Übernahme nach Validierung |
| `target.branch` | `ref` | explizites Mapping |
| `request_id` | — | keine Übergabe möglich |

### 4.2 Vertragsabweichung bei `request_id`

WF-0012 v0.1.0 definiert und verarbeitet kein Feld `request_id`.

WF-0013 behält deshalb seine eigene `request_id` während des Reader-Auftrags intern bei und ordnet das Reader-Ergebnis nach der Rückgabe wieder dem ursprünglichen Änderungsauftrag zu.

Die `request_id` darf nicht ungeprüft in den WF-0012-Eingang eingefügt werden.

### 4.3 Reader-Modus

WF-0012 arbeitet ausschließlich mit:

```text
mode: read-only
```

WF-0013 darf diesen Wert weder überschreiben noch als frei wählbaren Eingangsparameter behandeln.

---

## 5. WF-0012 zu WF-0013 – Reader-Ergebnis

WF-0013 darf ausschließlich normalisierte Ergebnisse von WF-0012 weiterverarbeiten.

### 5.1 Erfolgreiches Reader-Ergebnis

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
    "name": "workflow_registry.md",
    "path": "01_REGISTRY/workflow_registry.md",
    "sha": "0123456789abcdef0123456789abcdef01234567",
    "encoding": "base64",
    "size": 1234,
    "content": "# Workflow Registry\n"
  }
}
```

### 5.2 Verbindliche Erfolgsfelder

| Feld | Bedeutung | Behandlung durch WF-0013 |
|---|---|---|
| `status` | Ergebnisstatus | muss exakt `read` sein |
| `mode` | Betriebsmodus | muss exakt `read-only` sein |
| `source.owner`      | gelesener Repository-Owner | mit `target.owner` abgleichen |
| `source.repository` | gelesenes Repository       | mit `target.repository` abgleichen |
| `source.ref`        | gelesene Git-Referenz      | mit `target.branch` abgleichen |
| `file.name` | gelesener Dateiname | als Metadatum übernehmen |
| `file.path` | gelesener Dateipfad | mit `target.path` abgleichen |
| `file.sha` | aktueller GitHub-SHA | als bestätigten Ausgangsstand übernehmen |
| `file.encoding` | ursprüngliche GitHub-Kodierung | muss exakt `base64` sein |
| `file.size` | Dateigröße | als Metadatum übernehmen |
| `file.content`  | von WF-0012 dekodierter Klartext | als bestätigten Ausgangsinhalt validieren |

### 5.3 SHA-Prüfung und Übernahme

Vor der Vorbereitung des Writer-Payloads vergleicht WF-0013:

```text
target.expected_sha
reader_result.file.sha
```

Beide Werte müssen exakt übereinstimmen.

Nur bei erfolgreicher Übereinstimmung darf:

```text
reader_result.file.sha
```

unverändert als:

```text
source.expected_sha
```

in den vorbereiteten Writer-Payload übernommen werden.

Die verbindliche SHA-Kette lautet:

```text
Änderungsauftrag: target.expected_sha
                         ↓ Vergleich
Reader-Ergebnis: reader_result.file.sha
                         ↓ nur bei Gleichheit
Writer-Payload: source.expected_sha
```

Bei einer Abweichung gilt:

```text
status = rejected
decision.code = SHA_CONFLICT
writer_request.prepared = false
```

WF-0013 darf den aktuellen Reader-SHA weder als neuen erwarteten SHA einsetzen noch den ursprünglichen Änderungsauftrag automatisch verändern.


### 5.4 Inhaltsübernahme

`file.content` darf erst verwendet werden, wenn:

1. `status` exakt `read` ist,
2. `mode` exakt `read-only` ist,
3. `source.owner` mit `target.owner` übereinstimmt,
4. `source.repository` mit `target.repository` übereinstimmt,
5. `source.ref` mit `target.branch` übereinstimmt,
6. `file.path` dem validierten Zielpfad entspricht,
7. `file.sha` dem vorgeschriebenen SHA-Format entspricht,
8. `file.encoding` exakt `base64` ist,
9. `file.content` als bereits von WF-0012 dekodierter String vorliegt.

Der von WF-0012 dekodierte Inhalt bildet den bestätigten Ausgangsinhalt für die kontrollierte Änderung.

### 5.5 Kontrollierte Fehlerausgabe

```json
{
  "status": "rejected",
  "error": {
    "code": "ERROR_CODE",
    "message": "Kontrollierte Fehlerbeschreibung"
  }
}
```

Bei `status: rejected` muss WF-0013:

- die Verarbeitung abbrechen,
- keinen Writer-Payload erzeugen,
- `error.code` und `error.message` übernehmen,
- den Änderungsauftrag als abgelehnt kennzeichnen.

### 5.6 Schreibschutz

Die von WF-0012 ausgegebenen Schreibschutzwerte müssen immer `false` sein.

WF-0013 darf ein Reader-Ergebnis mit einem aktiven Schreibindikator nicht akzeptieren. WF-0012 bleibt unabhängig vom späteren Writer-Auftrag strikt read-only.

---

### 5.7 Entscheidungscodes

WF-0013 gibt über `decision.code` genau einen stabilen, maschinenlesbaren Entscheidungscode aus.

| Entscheidungscode | Bedeutung |
|---|---|
| `INVALID_INPUT` | Die Eingangsstruktur oder ein Pflichtfeld ist ungültig. |
| `INVALID_MODE` | Der angeforderte Betriebsmodus ist nicht zulässig. |
| `TARGET_NOT_ALLOWED` | Das angegebene Ziel ist nicht durch die Allowlist freigegeben. |
| `INVALID_PATH` | Der Zielpfad verletzt die verbindlichen Pfadregeln. |
| `INVALID_CHANGE` | Der vorgeschlagene Änderungsinhalt oder die Commit-Nachricht ist ungültig. |
| `CONTENT_TOO_LARGE` | Der vorgeschlagene vollständige Dateiinhalt überschreitet die zulässige Größe. |
| `READER_REQUEST_INVALID` | Der für WF-0012 vorbereitete Reader-Auftrag ist ungültig. |
| `READER_FAILED` | WF-0012 konnte das angeforderte Reader-Ergebnis nicht erfolgreich liefern. |
| `READER_RESULT_INVALID` | Das von WF-0012 gelieferte Ergebnis entspricht nicht dem erwarteten Vertrag. |
| `TARGET_MISMATCH` | Das Reader-Ergebnis gehört nicht vollständig zum angeforderten Ziel. |
| `SHA_CONFLICT` | Der erwartete SHA stimmt nicht mit dem aktuellen Dateistand überein. |
| `NO_CHANGE` | Der vorgeschlagene Inhalt entspricht bereits dem aktuellen Dateiinhalt. |
| `INVALID_APPROVAL` | Es liegt keine gültige ausdrückliche Freigabe für die Vorbereitung des Writer-Auftrags vor. |
| `WRITER_REQUEST_INVALID` | Der für WF-0011 vorbereitete Writer-Auftrag verletzt den Writer-Vertrag. |
| `INTERNAL_CONTROLLER_ERROR` | Ein interner technischer Fehler verhindert eine kontrollierte Verarbeitung. |
| `WRITER_REQUEST_PREPARED` | Der Writer-Auftrag wurde erfolgreich vorbereitet, aber weder ausgeführt noch an WF-0011 übergeben. |

Alle Codes außer `WRITER_REQUEST_PREPARED` verhindern die Vorbereitung eines ausführbaren Writer-Auftrags.

`INTERNAL_CONTROLLER_ERROR` führt zum Status `failed`. Fachliche und validierungsbedingte Ablehnungen führen zum Status `rejected`. `WRITER_REQUEST_PREPARED` führt zum Status `prepared`.

Die Entscheidungscodes müssen über kompatible Patch-Versionen stabil bleiben. Menschlich lesbare Fehlermeldungen dürfen präzisiert werden, ohne die Bedeutung des jeweiligen Codes zu verändern.

WF-0013 darf aus keinem Entscheidungscode eine automatische Ausführung von WF-0011 ableiten.

## 6. WF-0013 zu WF-0011 – Writer-Payload

WF-0013 erzeugt nach erfolgreicher Prüfung einen vorbereiteten Auftrag für den Zielvertrag von `WF-0011 v0.2.0`.

Es findet in `WF-0013 v0.1.0` keine automatische Übergabe und keine Schreiboperation statt.

### 6.1 Vorbereiteter Writer-Payload

```json
{
  "request_id": "REQ-WF0013-TEST-001",
  "target": {
    "owner": "Jamoko112026",
    "repository": "JaMoKo_Automation_OS",
    "branch": "main",
    "path": "01_REGISTRY/workflow_registry.md"
  },
  "source": {
    "expected_sha": "0123456789abcdef0123456789abcdef01234567"
  },
  "change": {
    "content": "vollständiger kontrollierter Zielinhalt",
    "commit_message": "Update workflow registry"
  },
  "execution": {
    "mode": "simulation"
  }
}
```

### 6.2 Feldzuordnung

| WF-0013 | WF-0011 v0.2.0 | Behandlung |
|---|---|---|
| `request_id` | `request_id` | übernehmen, falls vorhanden und gültig |
| `target.owner` | `target.owner` | direkte Übernahme |
| `target.repository` | `target.repository` | direkte Übernahme |
| `target.branch` | `target.branch` | direkte Übernahme |
| `target.path` | `target.path` | direkte Übernahme nach Validierung |
| `reader_result.file.sha` | `source.expected_sha` | nach Reader-Prüfung übernehmen |
| kontrollierter Zielinhalt | `change.content` | vollständigen geprüften Zielinhalt einsetzen |
| freigegebene Commit-Nachricht | `change.commit_message` | nach Validierung übernehmen |
| fester Controller-Modus | `execution.mode` | immer `simulation` |

### 6.3 Herkunft des erwarteten SHA

`source.expected_sha` muss exakt aus dem zuvor validierten Reader-Ergebnis stammen:

```text
WF-0012 file.sha
        ↓
WF-0013 reader_result.file.sha
        ↓
WF-0011 source.expected_sha
```

WF-0013 darf keinen manuell eingegebenen, selbst erzeugten oder nachträglich veränderten SHA an WF-0011 weiterreichen.

### 6.4 Inhaltsvertrag

`change.content` enthält den vollständigen kontrollierten Zielinhalt der Datei.

Vor der Aufnahme in den Writer-Payload muss WF-0013 bestätigen, dass:

1. der bereits von WF-0012 dekodierte Reader-Ausgangsinhalt validiert wurde,
2. die Änderung auf diesem bestätigten Ausgangsinhalt basiert,
3. der Zielinhalt vollständig erzeugt wurde,
4. der Zielinhalt nicht leer ist,
5. die Änderung ausschließlich den freigegebenen Zielpfad betrifft,
6. keine unzulässigen Inhalte oder Geheimnisse enthalten sind.

Ein unvollständiges Fragment darf nicht als `change.content` ausgegeben werden.

### 6.5 Commit-Nachricht

`change.commit_message` muss:

- als nicht leere Zeichenkette vorliegen,
- die kontrollierte Änderung knapp beschreiben,
- frei von Zeilenumbrüchen und Zugangsdaten sein,
- vor Erzeugung des Writer-Payloads validiert werden.

### 6.6 Betriebsmodus

Für den Zielvertrag von WF-0011 gilt verbindlich:

```text
execution.mode = simulation
```

WF-0013 darf in `v0.1.0` weder `write` noch einen anderen Betriebsmodus erzeugen.

Der Writer-Payload ist ausschließlich eine geprüfte Vorbereitung für eine spätere, ausdrücklich freigegebene Übergabe.

### 6.7 Ablehnungsbedingungen

WF-0013 darf keinen Writer-Payload erzeugen, wenn mindestens eine der folgenden Bedingungen vorliegt:

- das Reader-Ergebnis wurde abgelehnt,
- `file.path` stimmt nicht mit `target.path` überein,
- `file.sha` fehlt oder ist ungültig,
- der Dateiinhalt konnte nicht dekodiert werden,
- der Zielinhalt ist leer oder unvollständig,
- die Commit-Nachricht ist ungültig,
- die erforderliche Freigabe fehlt,
- der Betriebsmodus wäre nicht `simulation`.

---

## 7. Kompatibilitätsmatrix und Vertragsabweichungen

Die folgende Matrix bewertet die technische Kompatibilität zwischen `WF-0012 v0.1.0`, `WF-0013 v0.1.0` und dem Zielvertrag von `WF-0011 v0.2.0`.

### 7.1 Kompatibilitätsmatrix

| Schnittstelle | Feld oder Verhalten | Status | Erforderliche Behandlung |
|---|---|---|---|
| WF-0013 → WF-0012 | `target.owner` → `owner` | kompatibel | direkte Übernahme |
| WF-0013 → WF-0012 | `target.repository` → `repository` | kompatibel | direkte Übernahme |
| WF-0013 → WF-0012 | `target.path` → `path` | kompatibel | validieren und direkt übernehmen |
| WF-0013 → WF-0012 | `target.branch` → `ref` | Mapping erforderlich | Feldnamen kontrolliert umwandeln |
| WF-0013 → WF-0012 | `request_id` | nicht unterstützt | intern in WF-0013 behalten |
| WF-0012 → WF-0013 | `status: read` | kompatibel | exakt prüfen |
| WF-0012 → WF-0013 | `mode: read-only` | kompatibel | exakt prüfen |
| WF-0012 → WF-0013 | `file.path` | kompatibel | mit `target.path` abgleichen |
| WF-0012 → WF-0013 | `file.sha` | kompatibel | als bestätigten Ausgangs-SHA übernehmen |
| WF-0012 → WF-0013 | `file.encoding` | kompatibel | muss exakt `base64` sein |
| WF-0012 → WF-0013 | `file.content` | kompatibel | als bereits dekodierten Klartext validieren |
| WF-0012 → WF-0013 | `status: rejected` | kompatibel | Verarbeitung kontrolliert abbrechen |
| WF-0012 → WF-0013 | Schreibschutzwerte | kompatibel | alle Werte müssen `false` sein |
| WF-0013 → WF-0011 | `request_id` | kompatibel | nur übernehmen, falls vorhanden und gültig |
| WF-0013 → WF-0011 | `target.*` | kompatibel | nach Validierung direkt übernehmen |
| WF-0013 → WF-0011 | `reader_result.file.sha` → `source.expected_sha` | Mapping erforderlich | ausschließlich aus dem Reader-Ergebnis übernehmen |
| WF-0013 → WF-0011 | kontrollierter Zielinhalt → `change.content` | kompatibel | vollständigen Zielinhalt einsetzen |
| WF-0013 → WF-0011 | Commit-Nachricht → `change.commit_message` | kompatibel | vor Ausgabe validieren |
| WF-0013 → WF-0011 | `execution.mode` | kompatibel | fest auf `simulation` setzen |
| WF-0013 → WF-0011 | automatische Übergabe | nicht vorgesehen | in v0.1.0 unterbinden |

### 7.2 Bestätigte Vertragsabweichungen

#### Abweichung A – `branch` und `ref`

WF-0013 und WF-0011 verwenden:

```text
target.branch
```

WF-0012 verwendet:

```text
ref
```

WF-0013 muss deshalb beim Erzeugen des Reader-Auftrags ausdrücklich folgendes Mapping anwenden:

```text
target.branch → ref
```

Eine gleichzeitige Übergabe beider Felder ist nicht zulässig.

#### Abweichung B – fehlende `request_id` in WF-0012

WF-0012 v0.1.0 verarbeitet keine `request_id`.

WF-0013 muss die Zuordnung zwischen Änderungsauftrag und Reader-Ergebnis deshalb intern erhalten. Eine Erweiterung des WF-0012-Eingangs ist erst nach einer eigenen Spezifikations- und Versionsänderung zulässig.

#### Abweichung C – unterschiedliche Inhaltsverwendung

WF-0012 liefert in:

```text
file.content

#### Abweichung D – unterschiedliche Betriebsmodi

WF-0012 arbeitet ausschließlich mit:

```text
mode: read-only
```

Der Zielvertrag von WF-0011 arbeitet ausschließlich mit:

```text
execution.mode: simulation
```

Diese Werte beschreiben unterschiedliche Workflow-Grenzen und dürfen nicht ineinander umgedeutet werden.

WF-0013 behandelt beide Betriebsmodi als feste, nicht frei wählbare Vertragswerte.

#### Abweichung E – Dokumentation und technische Quelle

Der vollständige Ausgangsvertrag von WF-0012 ist nicht in dessen Markdown-Dokumentation beschrieben. Er wurde aus folgendem technisch implementierten Artefakt bestätigt:

```text
02_WORKFLOWS/WF-0012_GitHub_Reader/
└── exports/WF-0012_GitHub_Reader_v0.1.0.json
```

Vor einer späteren automatischen Kopplung muss der bestätigte Ausgangsvertrag zusätzlich in der WF-0012-Dokumentation nachgeführt werden.

### 7.3 Kompatibilitätsbewertung

Die drei Workflows sind auf Vertragsebene grundsätzlich koppelbar, sofern WF-0013:

1. `target.branch` kontrolliert zu `ref` abbildet,
2. `request_id` während des Reader-Auftrags intern erhält,
3. das Reader-Ergebnis vollständig validiert,
4. den bereits dekodierten `file.content`-String validiert,
5. `file.sha` unverändert als `source.expected_sha` übernimmt,
6. einen vollständigen kontrollierten Zielinhalt erzeugt,
7. `execution.mode` fest auf `simulation` setzt,
8. keine automatische Übergabe an WF-0011 ausführt.

### 7.4 Noch offene Voraussetzungen

Vor einer technischen Kopplung müssen mindestens folgende Punkte abgeschlossen sein:

- der WF-0012-Ausgangsvertrag wird in dessen Dokumentation ergänzt,
- die interne Zuordnung über `request_id` wird in WF-0013 implementiert und getestet,
- das Mapping `target.branch → ref` wird getestet,
- Klartext- und Inhaltsvalidierung werden getestet,
- die unveränderte SHA-Übernahme wird getestet,
- der Writer-Payload wird gegen den Vertrag von WF-0011 v0.2.0 validiert,
- Fehler- und Ablehnungspfade werden vollständig getestet,
- eine automatische Übergabe bleibt bis zu einer eigenen Freigabe deaktiviert.

---

## 8. Freigabebedingungen

`WF-0013 v0.1.0` darf erst als technisch vorbereitet bewertet werden, wenn alle nachfolgenden Bedingungen nachweislich erfüllt sind.

### 8.1 Reader-Auftrag

WF-0013 muss:

- ausschließlich die Felder `owner`, `repository`, `path` und `ref` an WF-0012 übergeben,
- `target.branch` kontrolliert auf `ref` abbilden,
- die eigene `request_id` intern erhalten,
- unzulässige oder zusätzliche Felder vor der Vorbereitung ablehnen,
- sicherstellen, dass WF-0012 ausschließlich read-only arbeitet.

### 8.2 Reader-Ergebnis

WF-0013 muss vor jeder weiteren Verarbeitung bestätigen:

- `status` ist exakt `read`,
- `mode` ist exakt `read-only`,
- `source.owner` entspricht dem freigegebenen `target.owner`,
- `source.repository` entspricht dem freigegebenen `target.repository`,
- `source.ref` entspricht dem freigegebenen `target.branch`,
- `file.path` entspricht dem freigegebenen `target.path`,
- `file.sha` ist vorhanden und formal gültig,
- `file.encoding` ist exakt `base64`,
- `file.content` liegt als bereits von WF-0012 dekodierter String vor,
- alle Schreibschutzwerte sind `false`.

Bei `status: rejected` oder einer verletzten Bedingung muss die Verarbeitung kontrolliert beendet werden.

### 8.3 Änderungsverarbeitung

Vor der Erzeugung eines Writer-Payloads muss WF-0013 sicherstellen:

- die Änderung basiert auf dem bestätigten Reader-Inhalt,
- der vollständige Zielinhalt wurde reproduzierbar erzeugt,
- der Zielinhalt ist nicht leer,
- ausschließlich der freigegebene Zielpfad wird verändert,
- die Commit-Nachricht ist gültig,
- die erforderliche Freigabe liegt vor,
- keine Zugangsdaten oder unzulässigen Inhalte werden ausgegeben.

### 8.4 Writer-Payload

Der vorbereitete Writer-Payload muss:

- dem Zielvertrag von `WF-0011 v0.2.0` entsprechen,
- den bestätigten Reader-SHA unverändert als `source.expected_sha` enthalten,
- den vollständigen Zielinhalt als `change.content` enthalten,
- `execution.mode` fest auf `simulation` setzen,
- eine gültige `request_id` nur dann enthalten, wenn sie im Änderungsauftrag vorhanden war,
- ohne automatische Übergabe oder Schreiboperation enden.

### 8.5 Erforderliche Tests

Mindestens folgende Testfälle müssen erfolgreich dokumentiert sein:

1. gültiger Reader-Auftrag,
2. korrektes Mapping `target.branch → ref`,
3. interne Erhaltung der `request_id`,
4. gültiges Reader-Ergebnis,
5. abgelehntes Reader-Ergebnis,
6. abweichender Dateipfad,
7. fehlender oder ungültiger SHA,
8. von `base64` abweichende `file.encoding`,
9. fehlender oder ungültiger dekodierter `file.content`-String,
10. leerer oder unvollständiger Zielinhalt,
11. ungültige Commit-Nachricht,
12. fehlende Freigabe,
13. korrekte SHA-Übernahme,
14. gültiger Writer-Payload im Modus `simulation`,
15. Ablehnung eines anderen Betriebsmodus,
16. Nachweis, dass keine automatische Übergabe stattfindet.

### 8.6 Freigabestatus

Die mögliche Bewertung lautet:

| Status | Bedeutung |
|---|---|
| `blocked` | mindestens eine Vertragsbedingung ist ungeklärt oder verletzt |
| `testable` | Verträge und Testfälle sind vollständig beschrieben |
| `verified` | alle verbindlichen Tests wurden erfolgreich ausgeführt |
| `released` | Dokumentation, Implementierung und Freigabe sind abgeschlossen |

Der aktuelle Status von `WF-0013 v0.1.0` bleibt:

```text
draft
```

Die Erstellung dieses Schnittstellenvertrags allein bewirkt keine technische Freigabe.

### 8.7 Ausschluss der Schreibfreigabe

Dieser Vertrag erteilt ausdrücklich keine Berechtigung für:

- den Betriebsmodus `write`,
- automatische Commits,
- automatische Push-Vorgänge,
- automatische Übergaben an WF-0011,
- Änderungen außerhalb des freigegebenen Zielpfads.

Eine spätere Schreibfreigabe benötigt eine eigene Version, dokumentierte Tests und eine ausdrückliche Entscheidung.

---

## 9. Offene Punkte und nächster Entwicklungsschritt

Der Schnittstellenvertrag beschreibt die vorgesehenen Übergänge zwischen WF-0012, WF-0013 und WF-0011. Die technische Kopplung ist damit noch nicht umgesetzt oder freigegeben.

### 9.1 Offene Punkte

Vor der Implementierung von WF-0013 müssen folgende Punkte bearbeitet werden:

1. Der technisch bestätigte Ausgangsvertrag wird in der Dokumentation von WF-0012 ergänzt.
2. Die Eingabe- und Ausgabeobjekte von WF-0013 werden als verbindliche Schemas definiert.
3. Das Mapping `target.branch → ref` wird implementiert und getestet.
4. Die interne Erhaltung und Rückzuordnung der `request_id` wird festgelegt.
5. Die Validierung des Reader-Ergebnisses wird implementiert.
6. Die Validierung des bereits von WF-0012 dekodierten `file.content`-Strings wird implementiert und abgesichert.
7. Die unveränderte Übernahme von `file.sha` nach `source.expected_sha` wird geprüft.
8. Die Erzeugung des vollständigen kontrollierten Zielinhalts wird definiert.
9. Der vorbereitete Writer-Payload wird gegen den Zielvertrag von WF-0011 v0.2.0 validiert.
10. Sämtliche Ablehnungs- und Fehlerpfade werden getestet.
11. Der Ausschluss automatischer Übergaben und Schreiboperationen wird technisch nachgewiesen.

### 9.2 Nächster Entwicklungsschritt

Der nächste konkrete Entwicklungsschritt ist die Erstellung von:

```text
02_WORKFLOWS/WF-0013_GitHub_Change_Controller/
└── TESTS.md
```

`TESTS.md` muss die in Abschnitt 8.5 definierten Testfälle in reproduzierbare Testszenarien überführen.

Für jeden Testfall sind mindestens festzuhalten:

- Test-ID,
- Testziel,
- Ausgangslage,
- Eingabe,
- erwartete Verarbeitung,
- erwartete Ausgabe,
- erwarteter Status,
- Nachweis,
- tatsächliches Ergebnis.

### 9.3 Reihenfolge der weiteren Arbeit

Die weitere Entwicklung erfolgt verbindlich in dieser Reihenfolge:

```text
1. INTERFACE_CONTRACT.md prüfen
2. TESTS.md erstellen
3. Schemas und Validierungsregeln ableiten
4. Workflow technisch implementieren
5. Tests ausführen und dokumentieren
6. Dokumentation mit der Implementierung abgleichen
7. Freigabestatus neu bewerten
```

Eine technische Implementierung vor Abschluss der Vertrags- und Testprüfung ist nicht vorgesehen.

### 9.4 Abschlussbewertung

Der aktuelle Stand lautet:

```text
Schnittstellen beschrieben: ja
Vertragsabweichungen dokumentiert: ja
Freigabebedingungen definiert: ja
Testfälle ausgeführt: nein
WF-0013 implementiert: nein
Automatische Kopplung freigegeben: nein
Schreiboperation freigegeben: nein
Status: draft
```

Damit ist der Schnittstellenvertrag inhaltlich vollständig vorbereitet, aber noch nicht technisch verifiziert.

---

## 10. Änderungsverlauf

| Version | Datum | Änderung | Status |
|---|---|---|---|
| `v0.1.0` | `2026-08-06` | Initialer Schnittstellenvertrag zwischen WF-0012, WF-0013 und WF-0011 v0.2.0 | `draft` |
