# WF-0011 – Flow

## Version

0.1.0

## Status

draft

---

## Zweck

Dieses Dokument beschreibt den konkreten Ablauf von WF-0011 – GitHub Writer.

Version 0.1.0 arbeitet ausschließlich im Betriebsmodus `simulation`.

Der Workflow prüft einen freigegebenen Änderungsvorschlag, erzeugt einen sicheren Änderungsplan und simuliert einen Patch. Er verändert keine Datei, erstellt keinen Commit und führt keinen Push aus.

---

## Startbedingung

Der Workflow startet mit genau einem Änderungsvorschlag, der:

- aus WF-0009 – Object Repair Engine stammt,
- durch WF-0010 – Object Auditor geprüft wurde,
- den Auditstatus `passed` besitzt,
- manuell freigegeben wurde,
- einen dokumentierten `sourceSha` enthält.

---

## Eingangsdaten

```json
{
  "objectId": "FIN-0001",
  "path": "01_Objects/Finance/FIN-0001/object.md",
  "field": "status",
  "currentValue": null,
  "approvedValue": "active",
  "approvalStatus": "approved",
  "sourceSha": "abc123",
  "approvedBy": "manual_review",
  "auditStatus": "passed"
}
```

---

## Gesamtfluss

```text
Manual Trigger
      │
      ▼
Set Simulation Input
      │
      ▼
Normalize Input
      │
      ▼
Validate Required Fields
      │
      ▼
Required Fields Valid?
      │
      ├── Nein ──► Build Rejected Result
      │
      └── Ja
            │
            ▼
      Validate Approval
            │
            ▼
      Approval Valid?
            │
            ├── Nein ──► Build Rejected Result
            │
            └── Ja
                  │
                  ▼
            Validate Audit
                  │
                  ▼
            Audit Passed?
                  │
                  ├── Nein ──► Build Rejected Result
                  │
                  └── Ja
                        │
                        ▼
                  Validate Safety
                        │
                        ▼
                  Safety Valid?
                        │
                        ├── Nein ──► Build Rejected Result
                        │
                        └── Ja
                              │
                              ▼
                        Build Change Plan
                              │
                              ▼
                        Simulate Patch
                              │
                              ▼
                        Validate Patch
                              │
                              ▼
                        Patch Valid?
                              │
                              ├── Nein ──► Build Rejected Result
                              │
                              └── Ja
                                    │
                                    ▼
                              Build Simulated Result
```

---

## Node-Übersicht

| Nr. | Node-Name | Typ | Aufgabe |
|---:|---|---|---|
| 1 | Manual Trigger | Manual Trigger | Startet den Testlauf |
| 2 | Set Simulation Input | Set | Stellt Testdaten bereit |
| 3 | Normalize Input | Code | Normalisiert die Eingangsdaten |
| 4 | Validate Required Fields | Code | Prüft Pflichtfelder und Grundstruktur |
| 5 | Required Fields Valid? | IF | Trennt gültige und ungültige Eingaben |
| 6 | Validate Approval | Code | Prüft Freigabe und Freigabeinstanz |
| 7 | Approval Valid? | IF | Trennt freigegebene und abgewiesene Vorschläge |
| 8 | Validate Audit | Code | Prüft den Auditstatus |
| 9 | Audit Passed? | IF | Trennt bestandene und nicht bestandene Audits |
| 10 | Validate Safety | Code | Prüft Modus, Objekt-ID, Pfad und SHA |
| 11 | Safety Valid? | IF | Trennt sichere und unsichere Vorschläge |
| 12 | Build Change Plan | Code | Erzeugt den strukturierten Änderungsplan |
| 13 | Simulate Patch | Code | Erzeugt den simulierten Patch |
| 14 | Validate Patch | Code | Prüft das Simulationsergebnis |
| 15 | Patch Valid? | IF | Trennt gültige und ungültige Patches |
| 16 | Build Simulated Result | Code | Erzeugt das erfolgreiche Endergebnis |
| 17 | Build Rejected Result | Code | Erzeugt ein strukturiertes Fehlerergebnis |

---

## Node 1 – Manual Trigger

### Typ

`Manual Trigger`

### Aufgabe

Startet WF-0011 während Entwicklung und Test manuell.

### Sicherheitswirkung

Der Workflow besitzt in Version 0.1.0 keinen produktiven Webhook und keinen automatischen Repository-Trigger.

---

## Node 2 – Set Simulation Input

### Typ

`Set`

### Aufgabe

Stellt einen einzelnen freigegebenen Änderungsvorschlag als Testeingabe bereit.

### Felder

```json
{
  "objectId": "FIN-0001",
  "path": "01_Objects/Finance/FIN-0001/object.md",
  "field": "status",
  "currentValue": null,
  "approvedValue": "active",
  "approvalStatus": "approved",
  "sourceSha": "abc123",
  "approvedBy": "manual_review",
  "auditStatus": "passed",
  "mode": "simulation"
}
```

---

## Node 3 – Normalize Input

### Typ

`Code`

### Aufgabe

Übernimmt die Eingangsdaten ohne fachliche Veränderung und ergänzt die Workflow-Metadaten.

### Ausgabe

```json
{
  "workflowId": "WF-0011",
  "version": "0.1.0",
  "mode": "simulation",
  "input": {
    "objectId": "FIN-0001",
    "path": "01_Objects/Finance/FIN-0001/object.md",
    "field": "status",
    "currentValue": null,
    "approvedValue": "active",
    "approvalStatus": "approved",
    "sourceSha": "abc123",
    "approvedBy": "manual_review",
    "auditStatus": "passed"
  }
}
```

---

## Node 4 – Validate Required Fields

### Typ

`Code`

### Aufgabe

Prüft:

- ob genau ein Änderungsvorschlag vorliegt,
- ob alle Pflichtfelder vorhanden sind,
- ob Pflichtwerte nicht leer sind,
- ob die Grundstruktur verarbeitet werden kann.

### Pflichtfelder

```text
objectId
path
field
currentValue
approvedValue
approvalStatus
sourceSha
approvedBy
auditStatus
```

`currentValue` darf ausdrücklich `null` sein.

### Erfolgreiche Ausgabe

```json
{
  "validationPassed": true,
  "errorCode": null,
  "message": null
}
```

### Fehlerausgabe

```json
{
  "validationPassed": false,
  "errorCode": "MISSING_REQUIRED_FIELD",
  "message": "Ein oder mehrere Pflichtfelder fehlen."
}
```

---

## Node 5 – Required Fields Valid?

### Typ

`IF`

### Bedingung

```text
validationPassed ist true
```

### Verzweigung

- `true` → Validate Approval
- `false` → Build Rejected Result

---

## Node 6 – Validate Approval

### Typ

`Code`

### Aufgabe

Prüft:

- `approvalStatus` ist exakt `approved`,
- `approvedBy` ist vorhanden und nicht leer.

### Fehlercodes

```text
APPROVAL_REQUIRED
MISSING_REQUIRED_FIELD
```

---

## Node 7 – Approval Valid?

### Typ

`IF`

### Bedingung

```text
approvalPassed ist true
```

### Verzweigung

- `true` → Validate Audit
- `false` → Build Rejected Result

---

## Node 8 – Validate Audit

### Typ

`Code`

### Aufgabe

Prüft, ob:

```text
auditStatus = passed
```

### Fehlercode

```text
AUDIT_NOT_PASSED
```

---

## Node 9 – Audit Passed?

### Typ

`IF`

### Bedingung

```text
auditPassed ist true
```

### Verzweigung

- `true` → Validate Safety
- `false` → Build Rejected Result

---

## Node 10 – Validate Safety

### Typ

`Code`

### Aufgabe

Führt die zentralen Sicherheitsprüfungen durch.

### Prüfungen

1. Der Betriebsmodus ist `simulation`.
2. `objectId` entspricht einer gültigen JaMoKo-Objekt-ID.
3. `path` ist ein relativer Repository-Pfad.
4. `path` enthält kein `..`.
5. `path` verweist nicht auf `.git`.
6. `path` enthält kein URL- oder Protokollschema.
7. `field` ist vorhanden und zulässig.
8. `sourceSha` ist vorhanden.
9. Es wurde keine Schreiboperation angefordert.

### Beispiel für eine gültige Objekt-ID

```text
FIN-0001
```

### Fehlercodes

```text
INVALID_MODE
INVALID_OBJECT_ID
INVALID_PATH
INVALID_FIELD
SOURCE_SHA_MISSING
```

---

## Node 11 – Safety Valid?

### Typ

`IF`

### Bedingung

```text
safetyPassed ist true
```

### Verzweigung

- `true` → Build Change Plan
- `false` → Build Rejected Result

---

## Node 12 – Build Change Plan

### Typ

`Code`

### Aufgabe

Erzeugt den geplanten Änderungsvorgang, ohne auf eine Datei zuzugreifen.

### Ausgabe

```json
{
  "changePlan": {
    "objectId": "FIN-0001",
    "path": "01_Objects/Finance/FIN-0001/object.md",
    "field": "status",
    "from": null,
    "to": "active",
    "sourceSha": "abc123",
    "approvedBy": "manual_review",
    "auditStatus": "passed",
    "operation": "replace_field_value",
    "executionAllowed": false
  }
}
```

---

## Node 13 – Simulate Patch

### Typ

`Code`

### Aufgabe

Erzeugt eine strukturierte Patch-Vorschau.

Der Node liest und verändert keine Repository-Datei.

### Beispielausgabe

```json
{
  "simulatedPatch": {
    "path": "01_Objects/Finance/FIN-0001/object.md",
    "field": "status",
    "before": null,
    "after": "active",
    "applied": false
  }
}
```

### Sicherheitswerte

```json
{
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

---

## Node 14 – Validate Patch

### Typ

`Code`

### Aufgabe

Prüft, ob der simulierte Patch:

- einen gültigen Zielpfad enthält,
- das betroffene Feld enthält,
- Ausgangs- und Zielwert nachvollziehbar dokumentiert,
- nicht als angewendet markiert ist,
- keine Schreibwirkung meldet.

### Erfolgreiche Ausgabe

```json
{
  "patchValid": true
}
```

### Fehlercode

```text
PATCH_VALIDATION_FAILED
```

---

## Node 15 – Patch Valid?

### Typ

`IF`

### Bedingung

```text
patchValid ist true
```

### Verzweigung

- `true` → Build Simulated Result
- `false` → Build Rejected Result

---

## Node 16 – Build Simulated Result

### Typ

`Code`

### Aufgabe

Erzeugt das erfolgreiche Endergebnis des Workflows.

### Ausgabe

```json
{
  "workflowId": "WF-0011",
  "version": "0.1.0",
  "mode": "simulation",
  "status": "simulated",
  "objectId": "FIN-0001",
  "path": "01_Objects/Finance/FIN-0001/object.md",
  "field": "status",
  "currentValue": null,
  "approvedValue": "active",
  "sourceSha": "abc123",
  "approvedBy": "manual_review",
  "auditStatus": "passed",
  "patchValid": true,
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

---

## Node 17 – Build Rejected Result

### Typ

`Code`

### Aufgabe

Erzeugt für jeden kontrollierten Abbruch ein einheitliches Fehlerergebnis.

### Ausgabe

```json
{
  "workflowId": "WF-0011",
  "version": "0.1.0",
  "mode": "simulation",
  "status": "rejected",
  "errorCode": "APPROVAL_REQUIRED",
  "message": "Der Änderungsvorschlag ist nicht freigegeben.",
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

---

## Fehlerpfade

Alle Fehlerpfade enden im Node `Build Rejected Result`.

| Prüfstufe | Fehlercode |
|---|---|
| Anzahl der Eingaben ungültig | `INVALID_INPUT_COUNT` |
| Pflichtfeld fehlt | `MISSING_REQUIRED_FIELD` |
| Objekt-ID ungültig | `INVALID_OBJECT_ID` |
| Pfad ungültig | `INVALID_PATH` |
| Feld ungültig | `INVALID_FIELD` |
| Freigabe fehlt | `APPROVAL_REQUIRED` |
| Audit nicht bestanden | `AUDIT_NOT_PASSED` |
| SHA fehlt | `SOURCE_SHA_MISSING` |
| Modus ungültig | `INVALID_MODE` |
| Patch ungültig | `PATCH_VALIDATION_FAILED` |

---

## Verbindungsplan

```text
Manual Trigger
→ Set Simulation Input
→ Normalize Input
→ Validate Required Fields
→ Required Fields Valid?

Required Fields Valid? [true]
→ Validate Approval
→ Approval Valid?

Required Fields Valid? [false]
→ Build Rejected Result

Approval Valid? [true]
→ Validate Audit
→ Audit Passed?

Approval Valid? [false]
→ Build Rejected Result

Audit Passed? [true]
→ Validate Safety
→ Safety Valid?

Audit Passed? [false]
→ Build Rejected Result

Safety Valid? [true]
→ Build Change Plan
→ Simulate Patch
→ Validate Patch
→ Patch Valid?

Safety Valid? [false]
→ Build Rejected Result

Patch Valid? [true]
→ Build Simulated Result

Patch Valid? [false]
→ Build Rejected Result
```

---

## Erfolgsbedingungen

Ein Lauf erhält den Status `simulated`, wenn:

- genau ein Änderungsvorschlag verarbeitet wurde,
- alle Pflichtfelder vorhanden sind,
- die manuelle Freigabe bestätigt ist,
- das Audit bestanden wurde,
- der Betriebsmodus `simulation` aktiv ist,
- Objekt-ID, Feld und Zielpfad gültig sind,
- der `sourceSha` dokumentiert ist,
- der Change Plan erstellt wurde,
- der simulierte Patch gültig ist,
- keine Datei verändert wurde,
- kein Commit erstellt wurde,
- kein Push ausgeführt wurde.

---

## Sicherheitsgarantie

WF-0011 v0.1.0 enthält keinen Node, der:

- Dateien schreibt,
- Git-Befehle ausführt,
- einen Branch erstellt,
- einen Commit erstellt,
- einen Push ausführt,
- eine GitHub-API mit Schreibrechten aufruft.

Der Ablauf endet immer mit einem reinen Simulationsergebnis.

---

## Abgrenzung

Nicht Bestandteil dieses Flows sind:

- tatsächlicher Zugriff auf Repository-Dateien,
- Vergleich des `sourceSha` mit GitHub,
- Anwendung eines Patches,
- Branch-Erstellung,
- Commit-Erstellung,
- Push-Ausführung,
- Pull-Request-Erstellung,
- Verarbeitung mehrerer Änderungsvorschläge.

---

## Nächster Umsetzungsschritt

Nach Freigabe dieses Flow-Dokuments wird WF-0011 v0.1.0 in n8n aufgebaut.

Die Nodes werden zunächst mit festen Testdaten eingerichtet. Anschließend werden Erfolgs-, Ablehnungs- und Sicherheitsfälle anhand von `TESTS.md` geprüft.