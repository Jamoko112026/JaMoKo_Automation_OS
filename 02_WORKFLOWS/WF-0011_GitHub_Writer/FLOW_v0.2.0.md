# WF-0011 – Flow v0.2.0

## Version

0.2.0

## Status

draft

## Betriebsmodus

simulation

---

## 1. Zweck

Dieses Dokument beschreibt den konkreten Ablauf von WF-0011 – GitHub Writer v0.2.0.

Der Flow überführt die in `ARCHITECTURE_v0.2.0.md` definierten Komponenten in eine nachvollziehbare n8n-Node-Abfolge.

WF-0011 v0.2.0:

- nimmt genau einen manuell übergebenen Writer-Auftrag entgegen,
- validiert Struktur und Sicherheitsbedingungen,
- trifft eine deterministische Entscheidung,
- simuliert einen vollständigen Dateiersatz,
- validiert den Simulationsplan,
- bereinigt die Ausgabe,
- verändert keine Datei,
- erstellt keinen Commit,
- führt keinen Push aus,
- verwendet keine GitHub-Schreibzugänge.

---

## 2. Flow-Grundsatz

Jede Ausführung endet mit genau einem bereinigten Ergebnis:

```text
simulated
```

oder:

```text
rejected
```

Für jeden Ausgang gelten:

```text
file_changed = false
commit_created = false
push_executed = false
write_executed = false
```

Kein Node darf diese Werte aus der Eingabe übernehmen oder auf `true` setzen.

---

## 3. Gesamtablauf

```text
00_MANUAL_TRIGGER
        |
        v
01_TEST_PAYLOAD
        |
        v
10_INPUT_GATE
        |
        v
20_SCHEMA_VALIDATOR
        |
        v
30_SECURITY_VALIDATOR
        |
        v
40_DECISION_ENGINE_INITIAL
        |
        +---------------- rejected ----------------+
        |                                          |
        v                                          v
50_CANONICAL_REQUEST                      70_REJECTION_BUILDER
        |                                          |
        v                                          |
51_PATCH_SIMULATOR                                 |
        |                                          |
        v                                          |
60_PATCH_VALIDATOR                                 |
        |                                          |
        v                                          |
61_DECISION_ENGINE_PATCH                           |
        |                                          |
        +----------- rejected ---------------------+
        |
        v
71_SUCCESS_BUILDER
        |
        +-------------------+
                            |
                            v
                  80_OUTPUT_SANITIZER
                            |
                            v
                    90_FINAL_OUTPUT
```

Die technische Umsetzung darf mehrere n8n-Verbindungen verwenden. Die logische Reihenfolge bleibt verbindlich.

---

## 4. Node-Übersicht

| Reihenfolge | Node-Name | n8n-Typ | Aufgabe |
|---:|---|---|---|
| 1 | `00_MANUAL_TRIGGER` | Manual Trigger | kontrollierte manuelle Ausführung starten |
| 2 | `01_TEST_PAYLOAD` | Edit Fields / Code | genau einen Testauftrag bereitstellen |
| 3 | `10_INPUT_GATE` | Code | Eingabeanzahl und Grundform prüfen |
| 4 | `20_SCHEMA_VALIDATOR` | Code | Pflichtfelder und Datentypen prüfen |
| 5 | `30_SECURITY_VALIDATOR` | Code | Sicherheitsregeln und Grenzwerte prüfen |
| 6 | `40_DECISION_ENGINE_INITIAL` | Code | ersten priorisierten Fehler bestimmen |
| 7 | `41_ROUTE_INITIAL_DECISION` | IF | zwischen Simulation und Ablehnung verzweigen |
| 8 | `50_CANONICAL_REQUEST` | Code | internes normalisiertes Auftragsobjekt erzeugen |
| 9 | `51_PATCH_SIMULATOR` | Code | Simulationsplan für Dateiersatz erzeugen |
| 10 | `60_PATCH_VALIDATOR` | Code | Simulationsplan validieren |
| 11 | `61_DECISION_ENGINE_PATCH` | Code | Patch-Ergebnis entscheiden |
| 12 | `62_ROUTE_PATCH_DECISION` | IF | zwischen Erfolg und Ablehnung verzweigen |
| 13 | `70_REJECTION_BUILDER` | Code | standardisierte Ablehnung erzeugen |
| 14 | `71_SUCCESS_BUILDER` | Code | standardisiertes Simulationsergebnis erzeugen |
| 15 | `80_OUTPUT_SANITIZER` | Code | Ausgabe auf erlaubte Felder reduzieren |
| 16 | `90_FINAL_OUTPUT` | Edit Fields / Code | genau ein Endergebnis ausgeben |

---

## 5. Node-Namensstandard

Für v0.2.0 gelten:

```text
00–09  Eingang
10–19  Input Gate
20–29  Schema
30–39  Sicherheit
40–49  erste Entscheidung
50–59  Simulation
60–69  Patch-Prüfung
70–79  Ergebnisaufbau
80–89  Bereinigung
90–99  Endausgabe
```

Node-Namen werden:

- eindeutig,
- stabil,
- ohne Versionsnummer,
- in Großbuchstaben,
- mit numerischem Präfix

angelegt.

Die Workflow-Version wird in den Workflow-Metadaten und Ergebnisobjekten geführt.

---

## 6. `00_MANUAL_TRIGGER`

### Typ

```text
Manual Trigger
```

### Aufgabe

Startet WF-0011 v0.2.0 ausschließlich manuell.

### Regeln

Der Workflow besitzt in v0.2.0 keinen:

- Webhook Trigger,
- Schedule Trigger,
- Form Trigger,
- Execute Workflow Trigger,
- automatischen Eingang aus WF-0013.

### Ausgang

Der Node erzeugt ausschließlich das n8n-Startsignal.

---

## 7. `01_TEST_PAYLOAD`

### Typ

```text
Edit Fields
```

oder, falls für die genaue Objektstruktur erforderlich:

```text
Code
```

### Aufgabe

Stellt genau einen kontrollierten Testauftrag bereit.

### Zulässiger Testauftrag

```json
{
  "request_id": "REQ-TEST-001",
  "target": {
    "owner": "Jamoko112026",
    "repository": "JaMoKo_Automation_OS",
    "path": "test-fixtures/controller-target.md",
    "ref": "main"
  },
  "source": {
    "expected_sha": "0123456789abcdef0123456789abcdef01234567",
    "controller_workflow": "WF-0013",
    "controller_status": "prepared",
    "audit_status": "passed",
    "approved_by": "manual_review"
  },
  "change": {
    "content": "# Simulierter Inhalt\n",
    "commit_message": "Update controller test fixture"
  },
  "execution": {
    "mode": "simulation"
  }
}
```

### Regeln

- Es werden keine realen Credentials eingetragen.
- Es werden keine vertraulichen Kundendaten verwendet.
- Es wird kein GitHub-Token eingetragen.
- Der Testinhalt bleibt klein und nachvollziehbar.
- Pro Ausführung wird genau ein Auftrag erzeugt.

---

## 8. `10_INPUT_GATE`

### Typ

```text
Code
```

### Aufgabe

Prüft Anzahl und Grundform der eingegangenen Items.

### Prüfungen

1. Anzahl der Items ist exakt `1`.
2. Das Item enthält ein JSON-Objekt.
3. Das Objekt ist nicht `null`.
4. Das Objekt ist kein Array.
5. Das Objekt ist nicht leer.
6. Es liegt keine Liste mehrerer Änderungen vor.

### Interner Ausgang

```json
{
  "stage": "input_gate",
  "checks": {
    "input_count_valid": true,
    "input_object_valid": true
  },
  "errors": []
}
```

### Fehler

| Bedingung | Fehlercode |
|---|---|
| null oder mehr als ein Item | `INVALID_INPUT_COUNT` |
| Eingang ist kein Objekt | `INVALID_FIELD_TYPE` |
| leeres Objekt | `MISSING_REQUIRED_FIELD` |
| mehrere Dateiänderungen | `INVALID_INPUT_COUNT` |

Der Node beendet den Workflow nicht selbst. Er dokumentiert Prüfergebnisse für die Decision Engine.

---

## 9. `20_SCHEMA_VALIDATOR`

### Typ

```text
Code
```

### Aufgabe

Prüft Pflichtfelder und Datentypen, ohne fachliche Sicherheitswerte zu bewerten.

### Pflichtfelder

```text
target.owner
target.repository
target.path
target.ref
source.expected_sha
source.controller_workflow
source.controller_status
source.audit_status
source.approved_by
change.content
change.commit_message
execution.mode
```

`request_id` ist optional.

### Erwartete Datentypen

| Feld | Typ |
|---|---|
| `request_id` | String, falls vorhanden |
| `target` | Objekt |
| `target.owner` | String |
| `target.repository` | String |
| `target.path` | String |
| `target.ref` | String |
| `source` | Objekt |
| `source.expected_sha` | String |
| `source.controller_workflow` | String |
| `source.controller_status` | String |
| `source.audit_status` | String |
| `source.approved_by` | String |
| `change` | Objekt |
| `change.content` | String |
| `change.commit_message` | String |
| `execution` | Objekt |
| `execution.mode` | String |

### Interner Ausgang

Der Node ergänzt:

```json
{
  "checks": {
    "required_fields_valid": true,
    "field_types_valid": true
  },
  "errors": []
}
```

### Fehler

- Fehlendes Pflichtfeld: `MISSING_REQUIRED_FIELD`
- Falscher Datentyp: `INVALID_FIELD_TYPE`

Leere oder fachlich ungültige Strings werden in den nachfolgenden Sicherheitsprüfungen bewertet, sofern das Feld vorhanden und vom richtigen Datentyp ist.

---

## 10. `30_SECURITY_VALIDATOR`

### Typ

```text
Code
```

### Aufgabe

Führt alle fachlichen Sicherheitsprüfungen aus und erzeugt strukturierte Prüfergebnisse.

Der Node trifft noch nicht die öffentliche Gesamtentscheidung.

### Prüfreihenfolge

1. Betriebsmodus
2. Controller-Herkunft
3. Controller-Status
4. Auditstatus
5. Ziel-Allowlist
6. Pfad
7. Referenz
8. SHA
9. Inhalt
10. Commit-Nachricht
11. optionale Request-ID

---

## 11. Betriebsmodus-Prüfung

Zulässiger Wert:

```text
simulation
```

Alle anderen Werte führen zu:

```text
INVALID_MODE
```

Der Wert wird exakt und case-sensitiv geprüft.

---

## 12. Controller-Prüfung

Zulässige Werte:

```text
source.controller_workflow = WF-0013
source.controller_status = prepared
source.audit_status = passed
source.approved_by = manual_review
```

Zuordnung:

| Abweichung | Fehlercode |
|---|---|
| falscher `controller_workflow` | `INVALID_CONTROLLER_SOURCE` |
| falscher `controller_status` | `CONTROLLER_NOT_PREPARED` |
| falscher `audit_status` | `AUDIT_NOT_PASSED` |
| falscher oder leerer `approved_by` | `AUDIT_NOT_PASSED` |

Diese Eingabewerte gelten nicht als kryptografischer Nachweis. Sie bilden in v0.2.0 den kontrollierten manuellen Übergabevertrag ab.

---

## 13. Ziel-Allowlist-Prüfung

Zulässig ist ausschließlich das exakte Paar:

```json
{
  "owner": "Jamoko112026",
  "repository": "JaMoKo_Automation_OS"
}
```

Die Prüfung erfolgt:

- ohne Teilstring,
- ohne Präfix,
- ohne Wildcard,
- ohne Groß-/Kleinschreibungsnormalisierung,
- ohne aus der Eingabe abgeleitete Freigabe.

Eine Abweichung führt zu:

```text
TARGET_NOT_ALLOWED
```

---

## 14. Pfadprüfung

### Ablauf

1. Originalwert übernehmen.
2. Nullzeichen ablehnen.
3. Kontrollzeichen ablehnen.
4. Zeilenumbrüche und Tabs ablehnen.
5. Backslashes ablehnen.
6. absoluten Unix-Pfad ablehnen.
7. absoluten Windows-Pfad ablehnen.
8. Wert höchstens einmal URL-dekodieren.
9. Dekodierungsfehler ablehnen.
10. verbleibende `%XX`-Kodierung ablehnen.
11. leere Pfadsegmente ablehnen.
12. `.`-Segmente ablehnen.
13. `..`-Segmente ablehnen.
14. Pfad normalisieren.
15. normalisierten Wert erneut vollständig prüfen.
16. relativen Zielpfad bestätigen.

### Abzulehnende Beispiele

```text
../secret.txt
folder/../secret.txt
/absolute/file.md
C:\secret.txt
folder\secret.txt
folder/%2e%2e/secret.txt
folder/%252e%252e/secret.txt
folder//file.md
folder/./file.md
```

### Zulässiges Beispiel

```text
test-fixtures/controller-target.md
```

Ein Fehler führt zu:

```text
INVALID_PATH
```

Der normalisierte Pfad wird separat als interner Prüfwert gespeichert.

---

## 15. Referenzprüfung

Für v0.2.0 ist ausschließlich zulässig:

```text
main
```

Jeder andere Wert führt zu:

```text
INVALID_REF
```

Es findet keine Abfrage bei GitHub statt.

---

## 16. SHA-Prüfung

### Fehlender SHA

Ein fehlender oder leerer Wert führt zu:

```text
SOURCE_SHA_MISSING
```

### Ungültiges Format

Zulässiges Format:

```regex
^[a-fA-F0-9]{40}$
```

Eine Abweichung führt zu:

```text
INVALID_SOURCE_SHA
```

Der SHA wird nicht gegen GitHub geprüft.

---

## 17. Inhaltsprüfung

`change.content` muss:

- ein String sein,
- gültig als UTF-8 verarbeitet werden können,
- frei von Nullzeichen sein,
- höchstens `100000` UTF-8-Bytes besitzen.

### Fehlerzuordnung

| Bedingung | Fehlercode |
|---|---|
| kein String oder technisch nicht verarbeitbar | `INVALID_CONTENT` |
| Nullzeichen enthalten | `INVALID_CONTENT` |
| größer als 100000 UTF-8-Bytes | `CONTENT_TOO_LARGE` |

Ein leerer String ist in v0.2.0 zulässig, weil er einen vollständigen Dateiersatz durch eine leere Datei simulieren kann.

Das Löschen einer Datei ist weiterhin nicht zulässig.

---

## 18. Commit-Nachrichten-Prüfung

`change.commit_message` muss:

- ein String sein,
- nach dem Trimmen mindestens ein sichtbares Zeichen enthalten,
- höchstens `120` Zeichen besitzen,
- genau eine Zeile umfassen,
- frei von Nullzeichen und Kontrollzeichen sein,
- keine erkennbare Credential- oder Authorization-Zeichenfolge enthalten.

Ein Fehler führt zu:

```text
INVALID_COMMIT_MESSAGE
```

Die validierte Nachricht wird intern getrimmt gespeichert.

---

## 19. Request-ID-Prüfung

`request_id` ist optional.

Ist sie vorhanden, gilt:

```regex
^REQ-[A-Z0-9][A-Z0-9-]{7,63}$
```

Ungültige optionale Request-IDs werden als:

```text
INVALID_FIELD_TYPE
```

behandelt.

Die Request-ID darf nicht automatisch generiert oder korrigiert werden.

---

## 20. Prüfergebnis des Security Validators

Beispiel:

```json
{
  "checks": {
    "mode_valid": true,
    "controller_source_valid": true,
    "controller_status_valid": true,
    "audit_status_valid": true,
    "target_allowed": true,
    "path_valid": true,
    "ref_valid": true,
    "sha_present": true,
    "sha_valid": true,
    "content_valid": true,
    "content_size_valid": true,
    "commit_message_valid": true,
    "request_id_valid": true
  },
  "normalized": {
    "path": "test-fixtures/controller-target.md",
    "commit_message": "Update controller test fixture",
    "content_bytes": 22
  },
  "errors": []
}
```

Der vollständige Inhalt bleibt für die interne Simulation verfügbar, wird aber nicht in das öffentliche Prüfergebnis kopiert.

---

## 21. `40_DECISION_ENGINE_INITIAL`

### Typ

```text
Code
```

### Aufgabe

Bestimmt anhand aller bisherigen Prüfergebnisse genau einen Gesamtstatus.

### Mögliche Ergebnisse

```text
continue
rejected
```

### Verbindliche Fehlerpriorität

1. `INVALID_INPUT_COUNT`
2. `MISSING_REQUIRED_FIELD`
3. `INVALID_FIELD_TYPE`
4. `INVALID_MODE`
5. `INVALID_CONTROLLER_SOURCE`
6. `CONTROLLER_NOT_PREPARED`
7. `AUDIT_NOT_PASSED`
8. `TARGET_NOT_ALLOWED`
9. `INVALID_PATH`
10. `INVALID_REF`
11. `SOURCE_SHA_MISSING`
12. `INVALID_SOURCE_SHA`
13. `INVALID_CONTENT`
14. `CONTENT_TOO_LARGE`
15. `INVALID_COMMIT_MESSAGE`
16. `PATCH_VALIDATION_FAILED`
17. `INTERNAL_ERROR`

### Entscheidungsobjekt bei Annahme

```json
{
  "decision": {
    "status": "continue",
    "error_code": null
  }
}
```

### Entscheidungsobjekt bei Ablehnung

```json
{
  "decision": {
    "status": "rejected",
    "error_code": "INVALID_PATH"
  }
}
```

Es wird ausschließlich der höchstpriorisierte zutreffende Fehler übernommen.

---

## 22. `41_ROUTE_INITIAL_DECISION`

### Typ

```text
IF
```

### Bedingung

```text
decision.status equals continue
```

### True-Pfad

Weiter zu:

```text
50_CANONICAL_REQUEST
```

### False-Pfad

Weiter zu:

```text
70_REJECTION_BUILDER
```

Der IF-Node verändert keine Daten.

---

## 23. `50_CANONICAL_REQUEST`

### Typ

```text
Code
```

### Aufgabe

Erzeugt ein neues internes, normalisiertes Auftragsobjekt.

### Internes Ergebnis

```json
{
  "request_id": "REQ-TEST-001",
  "target": {
    "owner": "Jamoko112026",
    "repository": "JaMoKo_Automation_OS",
    "path": "test-fixtures/controller-target.md",
    "ref": "main"
  },
  "source": {
    "expected_sha": "0123456789abcdef0123456789abcdef01234567",
    "controller_workflow": "WF-0013",
    "controller_status": "prepared",
    "audit_status": "passed",
    "approved_by": "manual_review"
  },
  "change": {
    "content": "# Simulierter Inhalt\n",
    "commit_message": "Update controller test fixture"
  },
  "execution": {
    "workflow_id": "WF-0011",
    "version": "0.2.0",
    "mode": "simulation"
  },
  "metadata": {
    "content_bytes": 22
  }
}
```

### Regeln

- Das Objekt wird neu aufgebaut.
- Unbekannte Eingangsfelder werden nicht kopiert.
- Der normalisierte Pfad wird verwendet.
- Die getrimmte Commit-Nachricht wird verwendet.
- Sicherheitsflags werden nicht aus der Eingabe übernommen.
- Das Objekt bleibt intern.
- Es wird keine Datei gelesen oder geschrieben.

---

## 24. `51_PATCH_SIMULATOR`

### Typ

```text
Code
```

### Aufgabe

Erzeugt einen nicht angewendeten Simulationsplan für einen vollständigen Dateiersatz.

### Interner Ausgang

```json
{
  "patch": {
    "patch_type": "full-file-replacement",
    "target_path": "test-fixtures/controller-target.md",
    "expected_sha": "0123456789abcdef0123456789abcdef01234567",
    "content_encoding": "utf-8",
    "content_bytes": 22,
    "applied": false
  }
}
```

### Regeln

Der Node:

- erzeugt keinen Unified Diff,
- liest keinen alten Dateiinhalt,
- verändert keine Datei,
- verwendet kein GitHub-Credential,
- ruft keine GitHub-API auf,
- erstellt keinen Commit,
- führt keinen Push aus.

Der vollständige neue Inhalt muss nicht in das Patch-Metadatenobjekt kopiert werden.

---

## 25. `60_PATCH_VALIDATOR`

### Typ

```text
Code
```

### Aufgabe

Validiert den erzeugten Simulationsplan gegen den kanonischen Auftrag.

### Prüfungen

1. `patch_type` entspricht `full-file-replacement`.
2. `target_path` entspricht exakt dem normalisierten Zielpfad.
3. `expected_sha` entspricht exakt dem validierten SHA.
4. `content_encoding` entspricht `utf-8`.
5. `content_bytes` entspricht der berechneten UTF-8-Byte-Länge.
6. `applied` ist exakt `false`.
7. Es existiert kein Schreibstatus.
8. Es existiert keine Commit-ID.
9. Es existiert kein Push-Ergebnis.
10. Es existieren keine Credential-Daten.

### Ausgang bei Erfolg

```json
{
  "patch_check": {
    "patch_valid": true,
    "error_code": null
  }
}
```

### Ausgang bei Fehler

```json
{
  "patch_check": {
    "patch_valid": false,
    "error_code": "PATCH_VALIDATION_FAILED"
  }
}
```

---

## 26. `61_DECISION_ENGINE_PATCH`

### Typ

```text
Code
```

### Aufgabe

Überführt das Patch-Prüfergebnis in die abschließende fachliche Entscheidung.

### Mögliche Ergebnisse

Bei gültigem Patch:

```json
{
  "decision": {
    "status": "simulated",
    "error_code": null
  }
}
```

Bei ungültigem Patch:

```json
{
  "decision": {
    "status": "rejected",
    "error_code": "PATCH_VALIDATION_FAILED"
  }
}
```

Bei einem sicher abgefangenen technischen Fehler:

```json
{
  "decision": {
    "status": "rejected",
    "error_code": "INTERNAL_ERROR"
  }
}
```

---

## 27. `62_ROUTE_PATCH_DECISION`

### Typ

```text
IF
```

### Bedingung

```text
decision.status equals simulated
```

### True-Pfad

Weiter zu:

```text
71_SUCCESS_BUILDER
```

### False-Pfad

Weiter zu:

```text
70_REJECTION_BUILDER
```

---

## 28. `70_REJECTION_BUILDER`

### Typ

```text
Code
```

### Aufgabe

Erzeugt für jeden Fehlercode eine einheitliche, sichere Ablehnung.

### Statische Fehlertexte

| Fehlercode | Meldung |
|---|---|
| `INVALID_INPUT_COUNT` | Es liegt nicht genau ein Writer-Auftrag vor. |
| `MISSING_REQUIRED_FIELD` | Mindestens ein erforderliches Feld fehlt. |
| `INVALID_FIELD_TYPE` | Mindestens ein Feld besitzt einen ungültigen Datentyp oder ein ungültiges Format. |
| `INVALID_MODE` | Der Writer darf ausschließlich im Simulationsmodus ausgeführt werden. |
| `INVALID_CONTROLLER_SOURCE` | Der Auftrag stammt nicht aus der vorgesehenen Controller-Verarbeitung. |
| `CONTROLLER_NOT_PREPARED` | Der Writer-Auftrag wurde nicht freigegeben vorbereitet. |
| `AUDIT_NOT_PASSED` | Die vorgelagerte Prüfung wurde nicht bestanden. |
| `TARGET_NOT_ALLOWED` | Owner und Repository sind nicht freigegeben. |
| `INVALID_PATH` | Der Zielpfad ist ungültig oder unsicher. |
| `INVALID_REF` | Die Zielreferenz ist nicht zulässig. |
| `SOURCE_SHA_MISSING` | Der erwartete Ausgangs-SHA fehlt. |
| `INVALID_SOURCE_SHA` | Der erwartete Ausgangs-SHA ist syntaktisch ungültig. |
| `INVALID_CONTENT` | Der vorgesehene Dateiinhalt ist ungültig. |
| `CONTENT_TOO_LARGE` | Der vorgesehene Dateiinhalt überschreitet das Größenlimit. |
| `INVALID_COMMIT_MESSAGE` | Die vorgesehene Commit-Nachricht ist ungültig. |
| `PATCH_VALIDATION_FAILED` | Der simulierte Änderungspatch konnte nicht validiert werden. |
| `INTERNAL_ERROR` | Die Simulation konnte nicht sicher abgeschlossen werden. |

### Internes Ergebnis

```json
{
  "workflow_id": "WF-0011",
  "version": "0.2.0",
  "mode": "simulation",
  "status": "rejected",
  "request_id": "REQ-TEST-001",
  "error_code": "INVALID_PATH",
  "message": "Der Zielpfad ist ungültig oder unsicher.",
  "file_changed": false,
  "commit_created": false,
  "push_executed": false,
  "write_executed": false
}
```

`request_id` wird nur aufgenommen, wenn sie vorhanden und bereits gültig ist.

---

## 29. `71_SUCCESS_BUILDER`

### Typ

```text
Code
```

### Aufgabe

Erzeugt das standardisierte Simulationsergebnis.

### Ergebnisstruktur

```json
{
  "workflow_id": "WF-0011",
  "version": "0.2.0",
  "mode": "simulation",
  "status": "simulated",
  "request_id": "REQ-TEST-001",
  "target": {
    "owner": "Jamoko112026",
    "repository": "JaMoKo_Automation_OS",
    "path": "test-fixtures/controller-target.md",
    "ref": "main"
  },
  "source": {
    "expected_sha": "0123456789abcdef0123456789abcdef01234567",
    "controller_workflow": "WF-0013",
    "controller_status": "prepared",
    "audit_status": "passed"
  },
  "simulation": {
    "content_valid": true,
    "commit_message_valid": true,
    "patch_valid": true,
    "patch_type": "full-file-replacement",
    "patch_applied": false
  },
  "file_changed": false,
  "commit_created": false,
  "push_executed": false,
  "write_executed": false
}
```

Nicht in das Ergebnis übernommen werden:

- `change.content`,
- `source.approved_by`,
- interne Prüfobjekte,
- vollständige Patch-Daten,
- unbekannte Eingangsfelder.

---

## 30. `80_OUTPUT_SANITIZER`

### Typ

```text
Code
```

### Aufgabe

Baut aus dem Ergebnis ein neues öffentliches Objekt anhand einer expliziten Feld-Allowlist.

### Erfolgs-Allowlist

```text
workflow_id
version
mode
status
request_id
target.owner
target.repository
target.path
target.ref
source.expected_sha
source.controller_workflow
source.controller_status
source.audit_status
simulation.content_valid
simulation.commit_message_valid
simulation.patch_valid
simulation.patch_type
simulation.patch_applied
file_changed
commit_created
push_executed
write_executed
```

### Fehler-Allowlist

```text
workflow_id
version
mode
status
request_id
error_code
message
file_changed
commit_created
push_executed
write_executed
```

### Verbindliche Sicherheitswerte

Der Sanitizer setzt unabhängig von der Eingabe:

```json
{
  "file_changed": false,
  "commit_created": false,
  "push_executed": false,
  "write_executed": false
}
```

### Zu entfernende Daten

- vollständiger Dateiinhalt,
- vollständiger Patch,
- Credentials,
- Tokens,
- Authorization-Header,
- n8n-Credential-Referenzen,
- interne Prüfergebnisse,
- rohe Fehlerobjekte,
- Stacktraces,
- Node-Metadaten,
- unbekannte Felder.

Der Sanitizer kopiert nicht das gesamte vorherige Objekt und löscht anschließend Felder. Er baut die Ausgabe ausschließlich aus erlaubten Feldern neu auf.

---

## 31. `90_FINAL_OUTPUT`

### Typ

```text
Edit Fields
```

oder:

```text
Code
```

### Aufgabe

Gibt genau ein bereits bereinigtes Ergebnis aus.

### Regeln

- Keine weitere Fachlogik.
- Keine weitere Normalisierung.
- Keine Fehlerdetails ergänzen.
- Keine Eingangsdaten erneut zusammenführen.
- Keine Verbindung zu einem weiteren Workflow.
- Keine Schreiboperation.
- Genau ein n8n-Item als Endausgabe.

---

## 32. Fehlerpfade

Alle fachlichen Fehler laufen über:

```text
Decision Engine
      |
      v
Rejection Builder
      |
      v
Output Sanitizer
      |
      v
Final Output
```

Kein Validator darf direkt mit dem finalen Ausgang verbunden werden.

Kein technischer Fehler darf ungefiltert den Output Sanitizer umgehen.

---

## 33. Technische Fehlerbehandlung

Code-Nodes müssen erwartbare ungültige Eingaben als strukturierte Prüfergebnisse behandeln.

Unerwartete Fehler werden intern abgefangen und in:

```text
INTERNAL_ERROR
```

überführt.

Nicht zulässig sind öffentliche Ausgaben mit:

- JavaScript-Stacktrace,
- n8n-Fehlerobjekt,
- Node-Konfiguration,
- Eingabedump,
- vollständigem Inhalt,
- Credential-Referenz.

Der Workflow darf keinen `Continue On Fail`-Pfad verwenden, der rohe Fehlerdaten direkt zur Endausgabe führt.

---

## 34. Merge-Verhalten

Die beiden fachlichen Ergebniswege:

```text
70_REJECTION_BUILDER
71_SUCCESS_BUILDER
```

dürfen beide mit:

```text
80_OUTPUT_SANITIZER
```

verbunden werden.

Ein Merge-Node ist nur dann zulässig, wenn er:

- keine beiden Pfade abwartet,
- keine Items kombiniert,
- keine Ergebnisduplikate erzeugt,
- keine internen Daten wieder zusammenführt.

Bevorzugt werden direkte Verbindungen beider Builder zum Sanitizer.

---

## 35. Verbotene Verbindungen

Nicht zulässig sind Verbindungen von:

- `01_TEST_PAYLOAD` direkt zum Patch Simulator,
- Validatoren direkt zum Final Output,
- Patch Simulator direkt zum Success Builder,
- Rejection Builder direkt zum Final Output,
- internen Datenpfaden an externe Workflows,
- WF-0013 automatisch zu WF-0011,
- WF-0011 zu einem GitHub-Schreibworkflow.

Der Output Sanitizer darf niemals umgangen werden.

---

## 36. Verbotene Nodes

WF-0011 v0.2.0 darf nicht enthalten:

- GitHub Write Nodes,
- GitHub Create-or-Update-File Nodes,
- HTTP Request Nodes mit `POST`, `PUT`, `PATCH` oder `DELETE` an GitHub,
- Execute Command,
- Read/Write Files from Disk,
- FTP-, SFTP- oder SSH-Schreiboperationen,
- Execute Workflow zu schreibenden Workflows,
- Webhook Trigger,
- Schedule Trigger,
- GitHub Trigger,
- Credential-Test-Nodes mit Schreibzugriff.

Erlaubt sind ausschließlich Nodes, die für manuellen Eingang, interne Verarbeitung und bereinigte Ausgabe erforderlich sind.

---

## 37. Credential-Regel

WF-0011 v0.2.0 benötigt keine Credentials.

Es dürfen insbesondere keine folgenden Credentials eingebunden werden:

- GitHub,
- Git,
- SSH,
- HTTP Header Auth,
- OAuth2,
- Dateisystem- oder Cloud-Speicher-Zugänge.

Die Anzahl eingebundener Credentials muss im bereinigten Export betragen:

```text
0
```

---

## 38. Datenhaltung in n8n

Vor der Freigabe muss geprüft werden:

1. Welche Eingangsdaten n8n speichert.
2. Welche Zwischenergebnisse gespeichert werden.
3. Ob `change.content` in Ausführungsdaten sichtbar bleibt.
4. Ob Fehlerausführungen rohe Node-Daten enthalten.
5. Ob erfolgreiche Ausführungen interne Patch-Daten enthalten.
6. Welche Einstellungen zur Speicherung erfolgreicher und fehlerhafter Ausführungen gelten.

Bis diese Prüfung abgeschlossen ist, bleibt der Workflow:

```text
draft
```

Testinhalte dürfen deshalb keine vertraulichen Daten enthalten.

---

## 39. Determinismus

Bei identischem Eingang muss WF-0011 v0.2.0 dasselbe fachliche Ergebnis erzeugen.

Nicht zulässig sind:

- Zufallswerte,
- aktuelle Zeit als Entscheidungsgrundlage,
- automatisch generierte Request-IDs,
- wechselnde Allowlist-Werte,
- externe API-Abfragen,
- nicht festgelegte Standardwerte,
- von der Node-Reihenfolge abhängige Fehlerausgaben.

Die Fehlerpriorität wird ausschließlich durch die verbindliche Fehlerliste bestimmt.

---

## 40. Seiteneffektfreiheit

Die Simulation darf ausschließlich interne n8n-Datenobjekte erzeugen.

Zulässige Wirkung:

```text
bereinigtes Simulationsergebnis erzeugen
```

Nicht zulässige Wirkungen:

```text
Datei verändern
Datei erstellen
Datei löschen
Commit erstellen
Push ausführen
Branch erstellen
Pull Request erstellen
GitHub aufrufen
anderen Workflow automatisch starten
Shell-Befehl ausführen
```

---

## 41. Erfolgsweg

Der vollständige Erfolgsweg lautet:

```text
00_MANUAL_TRIGGER
-> 01_TEST_PAYLOAD
-> 10_INPUT_GATE
-> 20_SCHEMA_VALIDATOR
-> 30_SECURITY_VALIDATOR
-> 40_DECISION_ENGINE_INITIAL
-> 41_ROUTE_INITIAL_DECISION
-> 50_CANONICAL_REQUEST
-> 51_PATCH_SIMULATOR
-> 60_PATCH_VALIDATOR
-> 61_DECISION_ENGINE_PATCH
-> 62_ROUTE_PATCH_DECISION
-> 71_SUCCESS_BUILDER
-> 80_OUTPUT_SANITIZER
-> 90_FINAL_OUTPUT
```

Erwarteter Endstatus:

```text
simulated
```

---

## 42. Ablehnungsweg vor der Simulation

Beispiel für einen ungültigen Pfad:

```text
00_MANUAL_TRIGGER
-> 01_TEST_PAYLOAD
-> 10_INPUT_GATE
-> 20_SCHEMA_VALIDATOR
-> 30_SECURITY_VALIDATOR
-> 40_DECISION_ENGINE_INITIAL
-> 41_ROUTE_INITIAL_DECISION
-> 70_REJECTION_BUILDER
-> 80_OUTPUT_SANITIZER
-> 90_FINAL_OUTPUT
```

Erwarteter Endstatus:

```text
rejected
```

Erwarteter Fehlercode:

```text
INVALID_PATH
```

Der Patch Simulator wird nicht erreicht.

---

## 43. Ablehnungsweg nach Patch-Simulation

Bei manipulierten oder inkonsistenten Patch-Metadaten:

```text
51_PATCH_SIMULATOR
-> 60_PATCH_VALIDATOR
-> 61_DECISION_ENGINE_PATCH
-> 62_ROUTE_PATCH_DECISION
-> 70_REJECTION_BUILDER
-> 80_OUTPUT_SANITIZER
-> 90_FINAL_OUTPUT
```

Erwarteter Fehlercode:

```text
PATCH_VALIDATION_FAILED
```

Auch in diesem Fall wurde kein Patch angewendet.

---

## 44. Kontrollpunkte beim n8n-Aufbau

Nach jeder Node-Gruppe wird geprüft:

### Eingang

```text
Manual Trigger vorhanden
Kein öffentlicher Trigger vorhanden
Genau ein Testauftrag
```

### Validierung

```text
Pflichtfelder geprüft
Datentypen geprüft
Fehler gesammelt
Keine vorzeitige öffentliche Ausgabe
```

### Sicherheit

```text
Allowlist exakt
Pfad normalisiert
Grenzwerte umgesetzt
Keine externe Abfrage
```

### Entscheidung

```text
Fehlerpriorität deterministisch
Genau ein Fehlercode
Kein Validator entscheidet allein
```

### Simulation

```text
Nur Patch-Metadaten
applied = false
Kein Schreibnode
```

### Ausgabe

```text
Erfolg und Fehler durch Sanitizer
Keine Inhalte in öffentlicher Ausgabe
Sicherheitsflags immer false
Genau ein Endergebnis
```

---

## 45. Implementierungsreihenfolge

Die n8n-Implementierung erfolgt in dieser Reihenfolge:

1. neuen Workflow als inaktiven Entwurf anlegen,
2. Manual Trigger anlegen,
3. sicheren Test-Payload anlegen,
4. Input Gate implementieren,
5. Schema Validator implementieren,
6. Security Validator implementieren,
7. erste Decision Engine implementieren,
8. initiale Verzweigung anlegen,
9. Canonical Request Builder implementieren,
10. Patch Simulator implementieren,
11. Patch Validator implementieren,
12. Patch Decision Engine implementieren,
13. zweite Verzweigung anlegen,
14. Rejection Builder implementieren,
15. Success Builder implementieren,
16. Output Sanitizer implementieren,
17. Final Output anlegen,
18. Verbindungen prüfen,
19. Credentials und verbotene Nodes prüfen,
20. Workflow inaktiv speichern,
21. Testfälle aus `TESTS_v0.2.0.md` ausführen,
22. bereinigten Export erzeugen und prüfen.

Es wird nicht parallel an einer schreibenden Writer-Version gearbeitet.

---

## 46. Definition of Done für den Flow

`FLOW_v0.2.0.md` gilt als technisch umgesetzt, wenn:

- alle dokumentierten Node-Gruppen vorhanden sind,
- alle Node-Namen dem Standard entsprechen,
- genau ein manueller Eingang existiert,
- genau ein bereinigtes Endergebnis entsteht,
- die Fehlerpriorität reproduzierbar ist,
- der Erfolgsweg vollständig funktioniert,
- jeder Ablehnungsweg den Sanitizer durchläuft,
- Patch-Metadaten validiert werden,
- keine Datei verändert werden kann,
- keine Credentials eingebunden sind,
- keine automatische Verbindung zu WF-0013 besteht,
- keine verbotenen Nodes enthalten sind,
- alle Tests bestanden sind,
- der n8n-Export sicher geprüft wurde.

---

## 47. Abgleich mit den Kerndokumenten

Vor dem Status `testing` muss der Flow übereinstimmen mit:

```text
SPECIFICATION_v0.2.0.md
ARCHITECTURE_v0.2.0.md
TESTS_v0.2.0.md
WF-0013 – Interface Contract
DEC-0009 – Writer Safety Modes
STD-0003 – Workflow Architecture Standard
```

Bei einem Widerspruch wird die Implementierung gestoppt und zuerst die Dokumentation geklärt.

---

## 48. Aktueller Flow-Status

```text
Version: 0.2.0
Status: draft
Mode: simulation
Flow: defined
Implementation: not-started
Nodes created: 0
Tests: not-run
Credentials: forbidden
GitHub write nodes: forbidden
Filesystem write nodes: forbidden
Automatic controller connection: forbidden
```

---

## 49. Nächster konkreter Schritt

```text
SPECIFICATION_v0.2.0.md,
ARCHITECTURE_v0.2.0.md,
FLOW_v0.2.0.md und
TESTS_v0.2.0.md gemeinsam auf Widersprüche prüfen.

Erst nach bestandenem Dokumentationsabgleich darf der
inaktive n8n-Workflow WF-0011 v0.2.0 angelegt werden.
```
