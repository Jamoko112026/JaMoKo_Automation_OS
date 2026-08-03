# WF-0011 – Tests

## Version

0.1.0

## Status

draft

---

## Testziel

Dieses Dokument definiert die Tests für WF-0011 – GitHub Writer.

Geprüft wird, ob der Workflow ausschließlich freigegebene und auditierte Änderungsvorschläge verarbeitet, einen gültigen simulierten Patch erzeugt und unter keinen Umständen eine Datei verändert, einen Commit erstellt oder einen Push ausführt.

---

## Testgegenstand

Getestet werden:

- Eingangsdaten und Pflichtfelder,
- manuelle Freigabe,
- Auditstatus,
- Betriebsmodus,
- Objekt-ID,
- Zielpfad,
- Zielfeld,
- `sourceSha`,
- Change Plan,
- Patch-Simulation,
- Erfolgs- und Fehlerausgaben,
- Sicherheitsgarantien.

---

## Testumgebung

Die Tests werden zunächst manuell in n8n ausgeführt.

Verwendet werden:

- `Manual Trigger`,
- feste Testdaten im Node `Set Simulation Input`,
- Betriebsmodus `simulation`,
- keine GitHub-Schreibzugänge,
- keine Git-Befehle,
- keine Nodes mit Dateischreibwirkung.

---

## Allgemeine Sicherheitsbedingungen

Diese Werte müssen bei jedem Testfall – auch bei Ablehnungen – erhalten bleiben:

```json
{
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}

---

## T-001 – Gültige Simulation

### Eingabe

Die vollständige Referenz-Eingabe wird unverändert verwendet.

### Erwartung

```json
{
  "status": "simulated",
  "mode": "simulation",
  "patchValid": true,
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

### Bestanden, wenn

- der Change Plan erzeugt wird,
- der simulierte Patch erzeugt wird,
- `status` den Wert `simulated` besitzt,
- keine Schreibwirkung eingetreten ist.

---

## T-002 – Fehlendes Pflichtfeld

### Änderung der Referenz-Eingabe

Das Feld `approvedValue` wird entfernt.

### Erwartung

```json
{
  "status": "rejected",
  "errorCode": "MISSING_REQUIRED_FIELD",
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

### Bestanden, wenn

Der Workflow vor der Freigabe- und Sicherheitsprüfung kontrolliert abbricht.

---

## T-003 – Mehrere Eingaben

### Eingabe

Dem Workflow werden zwei Änderungsvorschläge gleichzeitig übergeben.

### Erwartung

```json
{
  "status": "rejected",
  "errorCode": "INVALID_INPUT_COUNT",
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

### Bestanden, wenn

Keiner der beiden Änderungsvorschläge verarbeitet oder simuliert wird.

---

## T-004 – Freigabe fehlt

### Änderung der Referenz-Eingabe

```json
{
  "approvalStatus": "pending"
}
```

### Erwartung

```json
{
  "status": "rejected",
  "errorCode": "APPROVAL_REQUIRED",
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

### Bestanden, wenn

Kein Change Plan und kein simulierter Patch erzeugt wird.

---

## T-005 – Freigabeinstanz fehlt

### Änderung der Referenz-Eingabe

Das Feld `approvedBy` wird entfernt oder erhält einen leeren Wert.

### Erwartung

```json
{
  "status": "rejected",
  "errorCode": "MISSING_REQUIRED_FIELD",
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

### Bestanden, wenn

Der Vorschlag ohne dokumentierte Freigabeinstanz nicht weiterverarbeitet wird.

---

## T-006 – Audit nicht bestanden

### Änderung der Referenz-Eingabe

```json
{
  "auditStatus": "failed"
}
```

### Erwartung

```json
{
  "status": "rejected",
  "errorCode": "AUDIT_NOT_PASSED",
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

### Bestanden, wenn

Der Workflow vor der Safety-Prüfung kontrolliert abbricht und keinen Patch simuliert.


---

## T-007 – Ungültiger Modus

### Änderung der Referenz-Eingabe

```json
{
  "mode": "write"
}
```

### Erwartung

```json
{
  "status": "rejected",
  "errorCode": "INVALID_MODE",
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

### Bestanden, wenn

Der Workflow den Modus `write` ablehnt und keine Schreiboperation vorbereitet oder ausführt.

---

## T-008 – Ungültige Objekt-ID

### Änderung der Referenz-Eingabe

```json
{
  "objectId": "FIN0001"
}
```

### Erwartung

```json
{
  "status": "rejected",
  "errorCode": "INVALID_OBJECT_ID",
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

### Bestanden, wenn

Der Workflow die ungültige Objekt-ID erkennt und keinen Change Plan erzeugt.

---

## T-009 – Absoluter Zielpfad

### Änderung der Referenz-Eingabe

```json
{
  "path": "/Users/mo/Projects/JaMoKo_OS/01_Objects/Finance/FIN-0001/object.md"
}
```

### Erwartung

```json
{
  "status": "rejected",
  "errorCode": "INVALID_PATH",
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

### Bestanden, wenn

Der absolute Pfad abgelehnt und kein Patch simuliert wird.

---

## T-010 – Pfadüberschreitung

### Änderung der Referenz-Eingabe

```json
{
  "path": "../../outside/object.md"
}
```

### Erwartung

```json
{
  "status": "rejected",
  "errorCode": "INVALID_PATH",
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

### Bestanden, wenn

Der Workflow die Pfadüberschreitung erkennt und die Verarbeitung kontrolliert beendet.

---

## T-011 – Geschützter Git-Pfad

### Änderung der Referenz-Eingabe

```json
{
  "path": ".git/config"
}
```

### Erwartung

```json
{
  "status": "rejected",
  "errorCode": "INVALID_PATH",
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

### Bestanden, wenn

Der Zugriff auf einen geschützten Git-Bereich abgelehnt wird.

---

## T-012 – URL als Zielpfad

### Änderung der Referenz-Eingabe

```json
{
  "path": "https://github.com/Jamoko112026/JaMoKo_OS"
}
```

### Erwartung

```json
{
  "status": "rejected",
  "errorCode": "INVALID_PATH",
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

### Bestanden, wenn

Der Workflow die URL nicht als Repository-Pfad akzeptiert.

---

## T-013 – Ungültiges Feld

### Änderung der Referenz-Eingabe

```json
{
  "field": "../status"
}
```

### Erwartung

```json
{
  "status": "rejected",
  "errorCode": "INVALID_FIELD",
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

### Bestanden, wenn

Das ungültige Zielfeld abgelehnt und kein Change Plan erzeugt wird.

---

## T-014 – Fehlender sourceSha

### Änderung der Referenz-Eingabe

Das Feld `sourceSha` wird entfernt oder erhält einen leeren Wert.

### Erwartung

```json
{
  "status": "rejected",
  "errorCode": "SOURCE_SHA_MISSING",
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

### Bestanden, wenn

Der Workflow ohne dokumentierten Ausgangs-SHA kontrolliert abbricht.


---

## T-015 – Ungültiger simulierter Patch

### Testaufbau

Der simulierte Patch wird für diesen Test absichtlich mit einer unzulässigen Schreibwirkung erzeugt:

```json
{
  "simulatedPatch": {
    "path": "01_Objects/Finance/FIN-0001/object.md",
    "field": "status",
    "before": null,
    "after": "active",
    "applied": true
  }
}
```

### Erwartung

```json
{
  "status": "rejected",
  "errorCode": "PATCH_VALIDATION_FAILED",
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

### Bestanden, wenn

Der Workflow den Patch wegen `applied: true` ablehnt und keine tatsächliche Schreibwirkung eintritt.

---

## T-016 – currentValue ist null

### Änderung der Referenz-Eingabe

Keine Änderung erforderlich. Die Referenz-Eingabe enthält:

```json
{
  "currentValue": null
}
```

### Erwartung

```json
{
  "status": "simulated",
  "patchValid": true,
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

### Bestanden, wenn

`null` als zulässiger Ausgangswert verarbeitet und im simulierten Patch nachvollziehbar dokumentiert wird.

---

## T-017 – Schreibwirkung bleibt ausgeschlossen

### Testaufbau

Die Referenz-Eingabe wird vollständig verarbeitet. Anschließend werden das Endergebnis und die vorhandenen Workflow-Nodes geprüft.

### Erwartung

```json
{
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

### Zusätzlich zu prüfen

Der Workflow enthält keinen Node, der:

- eine Repository-Datei schreibt,
- einen Git-Befehl ausführt,
- einen Branch erstellt,
- einen Commit erstellt,
- einen Push ausführt,
- eine GitHub-API mit Schreibrechten aufruft.

### Bestanden, wenn

Alle Sicherheitswerte `false` sind und im Workflow keine ausführende Schreibkomponente vorhanden ist.

---

## Testprotokoll

Die Ergebnisse werden nach dem Aufbau des Workflows in dieser Tabelle dokumentiert:

| Test-ID | Datum | Ergebnis | Tatsächlicher Status | Tatsächlicher Fehlercode | Bemerkung |
|---|---|---|---|---|---|
| T-001 | – | offen | – | – | – |
| T-002 | – | offen | – | – | – |
| T-003 | – | offen | – | – | – |
| T-004 | – | offen | – | – | – |
| T-005 | – | offen | – | – | – |
| T-006 | – | offen | – | – | – |
| T-007 | – | offen | – | – | – |
| T-008 | – | offen | – | – | – |
| T-009 | – | offen | – | – | – |
| T-010 | – | offen | – | – | – |
| T-011 | – | offen | – | – | – |
| T-012 | – | offen | – | – | – |
| T-013 | – | offen | – | – | – |
| T-014 | – | offen | – | – | – |
| T-015 | – | offen | – | – | – |
| T-016 | – | offen | – | – | – |
| T-017 | – | offen | – | – | – |

---

## Abnahmekriterien

WF-0011 v0.1.0 kann vom Status `testing` in den Status `released` überführt werden, wenn:

- alle 17 Testfälle ausgeführt wurden,
- alle Testfälle bestanden sind,
- alle Erfolgs- und Fehlerausgaben strukturiert erzeugt werden,
- jeder kontrollierte Abbruch die vorgesehenen Fehlercodes liefert,
- `currentValue: null` korrekt verarbeitet wird,
- kein Test eine Datei verändert,
- kein Test einen Commit erstellt,
- kein Test einen Push ausführt,
- der Workflow ausschließlich im Modus `simulation` arbeitet,
- das Testprotokoll vollständig dokumentiert ist.

---

## Fehlerbehandlung während der Tests

Wenn ein Test nicht bestanden wird:

1. bleibt der Workflow im Status `testing`,
2. wird das tatsächliche Ergebnis im Testprotokoll dokumentiert,
3. wird die Ursache analysiert,
4. wird die Korrektur in `CHANGELOG.md` festgehalten,
5. wird der betroffene Test erneut ausgeführt,
6. werden bei Architekturänderungen auch `SPECIFICATION.md`, `ARCHITECTURE.md` und `FLOW.md` geprüft.

---

## Sicherheitsentscheidung

Ein fachlich korrektes Simulationsergebnis reicht nicht aus, wenn eine Schreibwirkung nicht eindeutig ausgeschlossen werden kann.

Sobald einer der Werte

```text
fileChanged
commitCreated
pushExecuted
```

den Wert `true` besitzt oder nicht eindeutig nachgewiesen werden kann, gilt der gesamte Testlauf als nicht bestanden.

---

## Abschluss

Die Testakte deckt den Erfolgsfall, kontrollierte Ablehnungen, ungültige Eingaben, Pfadangriffe und die vollständige Schreibschutzgarantie von WF-0011 v0.1.0 ab.

Die Tests werden nach dem Aufbau des n8n-Workflows ausgeführt und im Testprotokoll dokumentiert.