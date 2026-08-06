# WF-0011 – Tests v0.2.0

## Version

0.2.0

## Status

draft

## Betriebsmodus

simulation

---

## 1. Zweck

Dieses Dokument definiert die verbindlichen Tests für WF-0011 – GitHub Writer v0.2.0.

Die Tests weisen nach, dass der Workflow:

1. genau einen kontrollierten Auftrag verarbeitet,
2. Pflichtfelder und Datentypen prüft,
3. ausschließlich freigegebene Ziele akzeptiert,
4. unsichere Pfade zuverlässig ablehnt,
5. Inhalte und Commit-Nachrichten begrenzt,
6. Fehler deterministisch priorisiert,
7. ausschließlich einen vollständigen Dateiersatz simuliert,
8. interne und vertrauliche Daten aus der Ausgabe entfernt,
9. keine Datei verändert,
10. keinen Commit und keinen Push ausführt,
11. keine Credentials verwendet,
12. genau ein bereinigtes Endergebnis erzeugt.

WF-0011 v0.2.0 darf während sämtlicher Tests keine Schreiboperation ausführen.

---

## 2. Testgrundlagen

Die Tests basieren auf:

```text
SPECIFICATION_v0.2.0.md
ARCHITECTURE_v0.2.0.md
FLOW_v0.2.0.md
WF-0013 – Interface Contract
DEC-0009 – Writer Safety Modes
STD-0003 – Workflow Architecture Standard
```

Bei einem Widerspruch zwischen Testfall und Kerndokumentation wird der Test gestoppt.

Die Dokumentation wird vor einer Änderung des Workflows geklärt.

---

## 3. Teststatuswerte

Für einzelne Testfälle gelten:

```text
not-run
passed
failed
blocked
not-applicable
```

Für die gesamte Testakte gelten:

```text
draft
testing
passed
failed
```

Der Gesamtstatus darf erst auf `passed` gesetzt werden, wenn:

- alle Pflichtprüfungen ausgeführt wurden,
- alle erwarteten Ergebnisse bestätigt wurden,
- keine Seiteneffekte festgestellt wurden,
- der bereinigte Export geprüft wurde,
- keine Credentials enthalten sind,
- keine verbotenen Nodes enthalten sind.

---

## 4. Testarten

WF-0011 v0.2.0 wird in folgenden Bereichen geprüft:

1. Strukturtests
2. Funktions- und Erfolgswegtests
3. Input-Gate-Tests
4. Schema- und Datentypprüfungen
5. Betriebsmodus- und Controller-Tests
6. Allowlist-Tests
7. Pfadsicherheitstests
8. Referenz- und SHA-Tests
9. Inhalts- und Grenzwerttests
10. Commit-Nachrichten-Tests
11. Request-ID-Tests
12. Fehlerprioritätstests
13. Patch-Simulationstests
14. Patch-Validierungstests
15. Sanitizer-Tests
16. Determinismustests
17. Seiteneffektfreiheitstests
18. Credential- und Exporttests
19. n8n-Datenhaltungstests
20. End-to-End-Tests

---

## 5. Sicherer Basis-Testauftrag

Soweit ein Testfall keine Abweichung definiert, wird dieser Basisauftrag verwendet:

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

Der Basisauftrag enthält:

- keine realen Credentials,
- keine vertraulichen Kundendaten,
- keinen produktiven Dateiinhalt,
- keinen GitHub-Token,
- keine Authorization-Daten.

---

## 6. Erwartetes erfolgreiches Endergebnis

Der Basisauftrag muss zu genau einem bereinigten n8n-Item führen:

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

Nicht enthalten sein dürfen:

```text
change.content
change.commit_message
source.approved_by
interne checks
interne normalized-Werte
vollständige Patch-Daten
Credentials
Tokens
Authorization-Header
Stacktraces
Node-Metadaten
unbekannte Eingangsfelder
```

---

## 7. Allgemeine Testregeln

Für jeden Test gilt:

1. Der Workflow bleibt inaktiv.
2. Die Ausführung wird ausschließlich manuell gestartet.
3. Pro Ausführung wird genau ein definierter Testfall verwendet.
4. Es werden keine produktiven Inhalte eingesetzt.
5. Es werden keine Credentials eingebunden.
6. Es wird keine GitHub-API aufgerufen.
7. Es wird keine Datei gelesen oder geschrieben.
8. Es wird kein Commit erzeugt.
9. Es wird kein Push ausgeführt.
10. Das Endergebnis wird vollständig geprüft.
11. Die Anzahl der End-Items wird geprüft.
12. Der tatsächlich erreichte Node-Pfad wird dokumentiert.
13. Fehlerausgaben werden auf Datenabfluss geprüft.
14. Abweichungen werden in `KNOWN_ISSUES.md` dokumentiert.

---

## 8. Testprotokoll pro Testfall

Für jede Ausführung werden mindestens erfasst:

```text
Test-ID:
Datum:
Workflow-Version:
Ausführende Person:
Eingangsvariante:
Erwarteter Status:
Erwarteter Fehlercode:
Tatsächlicher Status:
Tatsächlicher Fehlercode:
End-Items:
Erreichter Node-Pfad:
Seiteneffekt festgestellt:
Ergebnis:
Notiz:
```

Vertrauliche oder vollständige Dateiinhalte werden nicht in das Testprotokoll kopiert.

---

# A. Strukturtests

## T-STR-001 – Manueller Trigger vorhanden

### Prüfung

Der Workflow enthält genau einen:

```text
00_MANUAL_TRIGGER
```

### Erwartung

- Node-Typ: Manual Trigger
- genau einmal vorhanden
- kein automatischer Eingang

### Ergebnis

```text
not-run
```

---

## T-STR-002 – Keine öffentlichen Trigger

### Prüfung

Der Workflow enthält keinen:

```text
Webhook Trigger
Schedule Trigger
Form Trigger
GitHub Trigger
```

### Erwartung

```text
keiner vorhanden
```

### Ergebnis

```text
not-run
```

---

## T-STR-003 – Keine automatische Controller-Verbindung

### Prüfung

Es existiert keine automatische Verbindung von WF-0013 zu WF-0011.

### Erwartung

```text
keine Execute-Workflow-Verbindung
keine Webhook-Verbindung
keine automatische Übergabe
```

### Ergebnis

```text
not-run
```

---

## T-STR-004 – Verbindliche Nodes vorhanden

### Prüfung

Folgende Nodes sind vollständig vorhanden:

```text
00_MANUAL_TRIGGER
01_TEST_PAYLOAD
10_INPUT_GATE
20_SCHEMA_VALIDATOR
30_SECURITY_VALIDATOR
40_DECISION_ENGINE_INITIAL
41_ROUTE_INITIAL_DECISION
50_CANONICAL_REQUEST
51_PATCH_SIMULATOR
60_PATCH_VALIDATOR
61_DECISION_ENGINE_PATCH
62_ROUTE_PATCH_DECISION
70_REJECTION_BUILDER
71_SUCCESS_BUILDER
80_OUTPUT_SANITIZER
90_FINAL_OUTPUT
```

### Erwartung

```text
16 von 16 Nodes vorhanden
```

### Ergebnis

```text
not-run
```

---

## T-STR-005 – Node-Namensstandard

### Prüfung

Alle Node-Namen sind:

- eindeutig,
- stabil,
- in Großbuchstaben,
- mit numerischem Präfix,
- ohne Versionsnummer.

### Erwartung

```text
alle Node-Namen gültig
```

### Ergebnis

```text
not-run
```

---

## T-STR-006 – Verbindungen entsprechen dem Flow

### Prüfung

Der Erfolgsweg und beide Ablehnungswege entsprechen `FLOW_v0.2.0.md`.

### Erwartung

- kein Validator führt direkt zum Final Output,
- kein Builder umgeht den Sanitizer,
- der Patch Simulator wird nur nach erfolgreicher erster Entscheidung erreicht,
- der Success Builder wird nur nach erfolgreicher Patch-Prüfung erreicht.

### Ergebnis

```text
not-run
```

---

## T-STR-007 – Keine verbotenen Nodes

### Prüfung

Der Workflow enthält keine:

```text
GitHub Write Nodes
HTTP Write Requests an GitHub
Execute Command Nodes
Read/Write Files from Disk Nodes
FTP-, SFTP- oder SSH-Schreibnodes
Execute Workflow Nodes zu schreibenden Workflows
Credential-Test-Nodes mit Schreibzugriff
```

### Erwartung

```text
0 verbotene Nodes
```

### Ergebnis

```text
not-run
```

---

# B. Funktions- und Erfolgswegtests

## T-FUN-001 – Gültiger Basisauftrag

### Eingang

Sicherer Basis-Testauftrag aus Abschnitt 5.

### Erwarteter Pfad

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

### Erwartung

```text
status = simulated
error_code nicht vorhanden
simulation.patch_valid = true
simulation.patch_applied = false
file_changed = false
commit_created = false
push_executed = false
write_executed = false
End-Items = 1
```

### Ergebnis

```text
not-run
```

---

## T-FUN-002 – Unbekannte Eingangsfelder

### Änderung

Der Basisauftrag wird ergänzt um:

```json
{
  "unknown_field": "must-not-pass",
  "target": {
    "unknown_target_field": "must-not-pass"
  }
}
```

Die vorhandenen Basisfelder bleiben erhalten.

### Erwartung

```text
status = simulated
unknown_field nicht im Endergebnis
target.unknown_target_field nicht im Endergebnis
End-Items = 1
```

### Ergebnis

```text
not-run
```

---

## T-FUN-003 – Leerer Dateiinhalt

### Änderung

```json
{
  "change": {
    "content": ""
  }
}
```

Die übrigen Basisfelder bleiben erhalten.

### Erwartung

```text
status = simulated
simulation.content_valid = true
metadata.content_bytes intern = 0
simulation.patch_applied = false
```

### Ergebnis

```text
not-run
```

---

# C. Input-Gate-Tests

## T-INP-001 – Kein Auftrag

### Eingang

```text
0 Items
```

### Erwartung

```text
status = rejected
error_code = INVALID_INPUT_COUNT
```

### Ergebnis

```text
not-run
```

---

## T-INP-002 – Zwei Aufträge

### Eingang

```text
2 Items
```

### Erwartung

```text
status = rejected
error_code = INVALID_INPUT_COUNT
Patch Simulator nicht erreicht
End-Items = 1
```

### Ergebnis

```text
not-run
```

---

## T-INP-003 – Eingang ist null

### Eingang

```json
null
```

### Erwartung

```text
status = rejected
error_code = INVALID_FIELD_TYPE
```

Falls n8n den Fall bereits als fehlendes Item darstellt:

```text
error_code = INVALID_INPUT_COUNT
```

Die tatsächlich verwendete n8n-Darstellung muss im Testprotokoll festgehalten werden.

### Ergebnis

```text
not-run
```

---

## T-INP-004 – Eingang ist Array

### Eingang

```json
[]
```

### Erwartung

```text
status = rejected
error_code = INVALID_FIELD_TYPE
```

### Ergebnis

```text
not-run
```

---

## T-INP-005 – Leeres Objekt

### Eingang

```json
{}
```

### Erwartung

```text
status = rejected
error_code = MISSING_REQUIRED_FIELD
```

### Ergebnis

```text
not-run
```

---

## T-INP-006 – Liste mehrerer Änderungen

### Änderung

```json
{
  "change": [
    {
      "content": "A"
    },
    {
      "content": "B"
    }
  ]
}
```

### Erwartung

```text
status = rejected
error_code = INVALID_INPUT_COUNT
```

### Ergebnis

```text
not-run
```

---

# D. Schema- und Datentypprüfungen

## T-SCH-001 – Pflichtfeld fehlt

### Änderung

Entferne:

```text
target.path
```

### Erwartung

```text
status = rejected
error_code = MISSING_REQUIRED_FIELD
```

### Ergebnis

```text
not-run
```

---

## T-SCH-002 – Mehrere Pflichtfelder fehlen

### Änderung

Entferne:

```text
target.path
source.expected_sha
change.commit_message
```

### Erwartung

```text
status = rejected
error_code = MISSING_REQUIRED_FIELD
genau ein öffentlicher Fehlercode
```

### Ergebnis

```text
not-run
```

---

## T-SCH-003 – target ist kein Objekt

### Änderung

```json
{
  "target": "Jamoko112026/JaMoKo_Automation_OS"
}
```

### Erwartung

```text
status = rejected
error_code = INVALID_FIELD_TYPE
```

### Ergebnis

```text
not-run
```

---

## T-SCH-004 – expected_sha ist Zahl

### Änderung

```json
{
  "source": {
    "expected_sha": 123456
  }
}
```

### Erwartung

```text
status = rejected
error_code = INVALID_FIELD_TYPE
```

### Ergebnis

```text
not-run
```

---

## T-SCH-005 – content ist Objekt

### Änderung

```json
{
  "change": {
    "content": {
      "text": "Test"
    }
  }
}
```

### Erwartung

```text
status = rejected
error_code = INVALID_FIELD_TYPE
```

### Ergebnis

```text
not-run
```

---

## T-SCH-006 – execution.mode ist Boolean

### Änderung

```json
{
  "execution": {
    "mode": true
  }
}
```

### Erwartung

```text
status = rejected
error_code = INVALID_FIELD_TYPE
```

### Ergebnis

```text
not-run
```

---

# E. Betriebsmodus- und Controller-Tests

## T-CTL-001 – Ungültiger Betriebsmodus write

### Änderung

```json
{
  "execution": {
    "mode": "write"
  }
}
```

### Erwartung

```text
status = rejected
error_code = INVALID_MODE
```

### Ergebnis

```text
not-run
```

---

## T-CTL-002 – Falsche Großschreibung des Modus

### Änderung

```json
{
  "execution": {
    "mode": "Simulation"
  }
}
```

### Erwartung

```text
status = rejected
error_code = INVALID_MODE
```

### Ergebnis

```text
not-run
```

---

## T-CTL-003 – Falscher Controller

### Änderung

```json
{
  "source": {
    "controller_workflow": "WF-0012"
  }
}
```

### Erwartung

```text
status = rejected
error_code = INVALID_CONTROLLER_SOURCE
```

### Ergebnis

```text
not-run
```

---

## T-CTL-004 – Controller nicht vorbereitet

### Änderung

```json
{
  "source": {
    "controller_status": "draft"
  }
}
```

### Erwartung

```text
status = rejected
error_code = CONTROLLER_NOT_PREPARED
```

### Ergebnis

```text
not-run
```

---

## T-CTL-005 – Audit nicht bestanden

### Änderung

```json
{
  "source": {
    "audit_status": "failed"
  }
}
```

### Erwartung

```text
status = rejected
error_code = AUDIT_NOT_PASSED
```

### Ergebnis

```text
not-run
```

---

## T-CTL-006 – approved_by leer

### Änderung

```json
{
  "source": {
    "approved_by": ""
  }
}
```

### Erwartung

```text
status = rejected
error_code = AUDIT_NOT_PASSED
```

### Ergebnis

```text
not-run
```

---

## T-CTL-007 – Falsche Freigabeart

### Änderung

```json
{
  "source": {
    "approved_by": "automatic"
  }
}
```

### Erwartung

```text
status = rejected
error_code = AUDIT_NOT_PASSED
```

### Ergebnis

```text
not-run
```

---

# F. Allowlist-Tests

## T-ALL-001 – Falscher Owner

### Änderung

```json
{
  "target": {
    "owner": "OtherOwner"
  }
}
```

### Erwartung

```text
status = rejected
error_code = TARGET_NOT_ALLOWED
```

### Ergebnis

```text
not-run
```

---

## T-ALL-002 – Falsches Repository

### Änderung

```json
{
  "target": {
    "repository": "OtherRepository"
  }
}
```

### Erwartung

```text
status = rejected
error_code = TARGET_NOT_ALLOWED
```

### Ergebnis

```text
not-run
```

---

## T-ALL-003 – Abweichende Großschreibung

### Änderung

```json
{
  "target": {
    "owner": "jamoko112026"
  }
}
```

### Erwartung

```text
status = rejected
error_code = TARGET_NOT_ALLOWED
```

### Ergebnis

```text
not-run
```

---

## T-ALL-004 – Owner mit freigegebenem Teilstring

### Änderung

```json
{
  "target": {
    "owner": "Jamoko112026-attacker"
  }
}
```

### Erwartung

```text
status = rejected
error_code = TARGET_NOT_ALLOWED
```

### Ergebnis

```text
not-run
```

---

## T-ALL-005 – Repository mit freigegebenem Präfix

### Änderung

```json
{
  "target": {
    "repository": "JaMoKo_Automation_OS-copy"
  }
}
```

### Erwartung

```text
status = rejected
error_code = TARGET_NOT_ALLOWED
```

### Ergebnis

```text
not-run
```

---

# G. Pfadsicherheitstests

## T-PTH-001 – Gültiger relativer Pfad

### Wert

```text
test-fixtures/controller-target.md
```

### Erwartung

```text
status = simulated
normalisierter Pfad entspricht exakt dem Eingang
```

### Ergebnis

```text
not-run
```

---

## T-PTH-002 – Parent-Traversal

### Wert

```text
../secret.txt
```

### Erwartung

```text
status = rejected
error_code = INVALID_PATH
```

### Ergebnis

```text
not-run
```

---

## T-PTH-003 – Eingebettetes Parent-Segment

### Wert

```text
folder/../secret.txt
```

### Erwartung

```text
status = rejected
error_code = INVALID_PATH
```

### Ergebnis

```text
not-run
```

---

## T-PTH-004 – Absoluter Unix-Pfad

### Wert

```text
/absolute/file.md
```

### Erwartung

```text
status = rejected
error_code = INVALID_PATH
```

### Ergebnis

```text
not-run
```

---

## T-PTH-005 – Absoluter Windows-Pfad

### Wert

```text
C:\secret.txt
```

### Erwartung

```text
status = rejected
error_code = INVALID_PATH
```

### Ergebnis

```text
not-run
```

---

## T-PTH-006 – Backslash im relativen Pfad

### Wert

```text
folder\secret.txt
```

### Erwartung

```text
status = rejected
error_code = INVALID_PATH
```

### Ergebnis

```text
not-run
```

---

## T-PTH-007 – URL-kodiertes Parent-Segment

### Wert

```text
folder/%2e%2e/secret.txt
```

### Erwartung

```text
status = rejected
error_code = INVALID_PATH
```

### Ergebnis

```text
not-run
```

---

## T-PTH-008 – Doppelt kodiertes Parent-Segment

### Wert

```text
folder/%252e%252e/secret.txt
```

### Erwartung

```text
status = rejected
error_code = INVALID_PATH
```

### Ergebnis

```text
not-run
```

---

## T-PTH-009 – Leeres Pfadsegment

### Wert

```text
folder//file.md
```

### Erwartung

```text
status = rejected
error_code = INVALID_PATH
```

### Ergebnis

```text
not-run
```

---

## T-PTH-010 – Punktsegment

### Wert

```text
folder/./file.md
```

### Erwartung

```text
status = rejected
error_code = INVALID_PATH
```

### Ergebnis

```text
not-run
```

---

## T-PTH-011 – Nullzeichen

### Wert

```text
folder/file.md\u0000.txt
```

### Erwartung

```text
status = rejected
error_code = INVALID_PATH
```

### Ergebnis

```text
not-run
```

---

## T-PTH-012 – Zeilenumbruch

### Wert

```text
folder/file.md
secret.txt
```

### Erwartung

```text
status = rejected
error_code = INVALID_PATH
```

### Ergebnis

```text
not-run
```

---

## T-PTH-013 – Tabulator

### Wert

```text
folder	file.md
```

### Erwartung

```text
status = rejected
error_code = INVALID_PATH
```

### Ergebnis

```text
not-run
```

---

## T-PTH-014 – Ungültige URL-Kodierung

### Wert

```text
folder/%ZZ/file.md
```

### Erwartung

```text
status = rejected
error_code = INVALID_PATH
```

### Ergebnis

```text
not-run
```

---

## T-PTH-015 – Verbleibende Kodierung nach einmaligem Dekodieren

### Wert

```text
folder/%2541/file.md
```

### Erwartung

```text
status = rejected
error_code = INVALID_PATH
```

### Ergebnis

```text
not-run
```

---

## T-PTH-016 – Leerer Pfad

### Wert

```text

```

### Erwartung

```text
status = rejected
error_code = INVALID_PATH
```

### Ergebnis

```text
not-run
```

---

# H. Referenz- und SHA-Tests

## T-REF-001 – Zulässige Referenz

### Wert

```text
main
```

### Erwartung

```text
Referenzprüfung bestanden
```

### Ergebnis

```text
not-run
```

---

## T-REF-002 – Andere Branch-Referenz

### Wert

```text
develop
```

### Erwartung

```text
status = rejected
error_code = INVALID_REF
```

### Ergebnis

```text
not-run
```

---

## T-REF-003 – Abweichende Großschreibung

### Wert

```text
Main
```

### Erwartung

```text
status = rejected
error_code = INVALID_REF
```

### Ergebnis

```text
not-run
```

---

## T-SHA-001 – Gültiger SHA mit Kleinbuchstaben

### Wert

```text
0123456789abcdef0123456789abcdef01234567
```

### Erwartung

```text
SHA-Prüfung bestanden
```

### Ergebnis

```text
not-run
```

---

## T-SHA-002 – Gültiger SHA mit Großbuchstaben

### Wert

```text
0123456789ABCDEF0123456789ABCDEF01234567
```

### Erwartung

```text
SHA-Prüfung bestanden
```

### Ergebnis

```text
not-run
```

---

## T-SHA-003 – SHA fehlt

### Änderung

Entferne:

```text
source.expected_sha
```

### Erwartung

Da das Feld als Pflichtfeld bereits im Schema fehlt:

```text
status = rejected
error_code = MISSING_REQUIRED_FIELD
```

### Ergebnis

```text
not-run
```

---

## T-SHA-004 – SHA leer

### Wert

```text

```

### Erwartung

```text
status = rejected
error_code = SOURCE_SHA_MISSING
```

### Ergebnis

```text
not-run
```

---

## T-SHA-005 – SHA zu kurz

### Wert

```text
0123456789abcdef
```

### Erwartung

```text
status = rejected
error_code = INVALID_SOURCE_SHA
```

### Ergebnis

```text
not-run
```

---

## T-SHA-006 – SHA mit ungültigem Zeichen

### Wert

```text
g123456789abcdef0123456789abcdef01234567
```

### Erwartung

```text
status = rejected
error_code = INVALID_SOURCE_SHA
```

### Ergebnis

```text
not-run
```

---

# I. Inhalts- und Grenzwerttests

## T-CNT-001 – Normaler UTF-8-Inhalt

### Wert

```text
# Simulierter Inhalt
```

### Erwartung

```text
status = simulated
content_valid = true
```

### Ergebnis

```text
not-run
```

---

## T-CNT-002 – Unicode-Inhalt

### Wert

```text
Hamburg – ruhig, klar und zuverlässig. ⚓
```

### Erwartung

```text
status = simulated
UTF-8-Byte-Länge korrekt berechnet
```

### Ergebnis

```text
not-run
```

---

## T-CNT-003 – Genau 100000 UTF-8-Bytes

### Eingang

Ein kontrollierter Teststring mit exakt:

```text
100000 UTF-8-Bytes
```

### Erwartung

```text
status = simulated
content_size_valid = true
```

### Ergebnis

```text
not-run
```

---

## T-CNT-004 – 100001 UTF-8-Bytes

### Eingang

Ein kontrollierter Teststring mit exakt:

```text
100001 UTF-8-Bytes
```

### Erwartung

```text
status = rejected
error_code = CONTENT_TOO_LARGE
```

### Ergebnis

```text
not-run
```

---

## T-CNT-005 – Mehrbyte-Zeichen an der Grenze

### Eingang

Ein UTF-8-Teststring mit Mehrbyte-Zeichen, dessen Zeichenanzahl unter 100000 liegt, dessen Byte-Länge aber 100000 überschreitet.

### Erwartung

```text
status = rejected
error_code = CONTENT_TOO_LARGE
```

### Ergebnis

```text
not-run
```

---

## T-CNT-006 – Nullzeichen im Inhalt

### Wert

```text
Test\u0000Inhalt
```

### Erwartung

```text
status = rejected
error_code = INVALID_CONTENT
```

### Ergebnis

```text
not-run
```

---

# J. Commit-Nachrichten-Tests

## T-CMT-001 – Gültige Commit-Nachricht

### Wert

```text
Update controller test fixture
```

### Erwartung

```text
status = simulated
commit_message_valid = true
```

### Ergebnis

```text
not-run
```

---

## T-CMT-002 – Äußere Leerzeichen

### Wert

```text
  Update controller test fixture
```

### Erwartung

```text
status = simulated
intern gespeicherte Nachricht = Update controller test fixture
```

### Ergebnis

```text
not-run
```

---

## T-CMT-003 – Leere Nachricht

### Wert

```text

```

### Erwartung

```text
status = rejected
error_code = INVALID_COMMIT_MESSAGE
```

### Ergebnis

```text
not-run
```

---

## T-CMT-004 – Nur Leerzeichen

### Wert

```text

```

### Erwartung

```text
status = rejected
error_code = INVALID_COMMIT_MESSAGE
```

### Ergebnis

```text
not-run
```

---

## T-CMT-005 – Genau 120 Zeichen

### Eingang

Eine einzeilige, kontrollierte Commit-Nachricht mit exakt 120 Zeichen.

### Erwartung

```text
status = simulated
commit_message_valid = true
```

### Ergebnis

```text
not-run
```

---

## T-CMT-006 – 121 Zeichen

### Eingang

Eine einzeilige, kontrollierte Commit-Nachricht mit exakt 121 Zeichen.

### Erwartung

```text
status = rejected
error_code = INVALID_COMMIT_MESSAGE
```

### Ergebnis

```text
not-run
```

---

## T-CMT-007 – Mehrzeilige Nachricht

### Wert

```text
Update fixture
Second line
```

### Erwartung

```text
status = rejected
error_code = INVALID_COMMIT_MESSAGE
```

### Ergebnis

```text
not-run
```

---

## T-CMT-008 – Nullzeichen

### Wert

```text
Update\u0000fixture
```

### Erwartung

```text
status = rejected
error_code = INVALID_COMMIT_MESSAGE
```

### Ergebnis

```text
not-run
```

---

## T-CMT-009 – Credential-Zeichenfolge

### Wert

```text
Authorization: Bearer test-secret-value
```

### Erwartung

```text
status = rejected
error_code = INVALID_COMMIT_MESSAGE
```

### Ergebnis

```text
not-run
```

---

## T-CMT-010 – Tokenartige Zeichenfolge

### Wert

```text
Update token=example-secret-value
```

### Erwartung

```text
status = rejected
error_code = INVALID_COMMIT_MESSAGE
```

### Ergebnis

```text
not-run
```

---

# K. Request-ID-Tests

## T-REQ-001 – Gültige Request-ID

### Wert

```text
REQ-TEST-001
```

### Erwartung

```text
status = simulated
request_id im Endergebnis enthalten
```

### Ergebnis

```text
not-run
```

---

## T-REQ-002 – Request-ID fehlt

### Änderung

Entferne:

```text
request_id
```

### Erwartung

```text
status = simulated
request_id nicht im Endergebnis
keine automatische ID erzeugt
```

### Ergebnis

```text
not-run
```

---

## T-REQ-003 – Request-ID zu kurz

### Wert

```text
REQ-A
```

### Erwartung

```text
status = rejected
error_code = INVALID_FIELD_TYPE
```

### Ergebnis

```text
not-run
```

---

## T-REQ-004 – Request-ID in Kleinbuchstaben

### Wert

```text
REQ-test-001
```

### Erwartung

```text
status = rejected
error_code = INVALID_FIELD_TYPE
```

### Ergebnis

```text
not-run
```

---

## T-REQ-005 – Request-ID mit Sonderzeichen

### Wert

```text
REQ-TEST_001
```

### Erwartung

```text
status = rejected
error_code = INVALID_FIELD_TYPE
```

### Ergebnis

```text
not-run
```

---

## T-REQ-006 – Request-ID als Zahl

### Wert

```json
123456789
```

### Erwartung

```text
status = rejected
error_code = INVALID_FIELD_TYPE
```

### Ergebnis

```text
not-run
```

---

# L. Fehlerprioritätstests

## T-PRI-001 – Fehlendes Feld vor ungültigem Modus

### Änderungen

- `target.path` entfernen
- `execution.mode = write`

### Erwartung

```text
status = rejected
error_code = MISSING_REQUIRED_FIELD
```

### Ergebnis

```text
not-run
```

---

## T-PRI-002 – Datentyp vor Modus

### Änderungen

- `target.owner = 123`
- `execution.mode = write`

### Erwartung

```text
status = rejected
error_code = INVALID_FIELD_TYPE
```

### Ergebnis

```text
not-run
```

---

## T-PRI-003 – Modus vor Controller-Quelle

### Änderungen

- `execution.mode = write`
- `source.controller_workflow = WF-0012`

### Erwartung

```text
status = rejected
error_code = INVALID_MODE
```

### Ergebnis

```text
not-run
```

---

## T-PRI-004 – Controller-Quelle vor Controller-Status

### Änderungen

- `source.controller_workflow = WF-0012`
- `source.controller_status = draft`

### Erwartung

```text
status = rejected
error_code = INVALID_CONTROLLER_SOURCE
```

### Ergebnis

```text
not-run
```

---

## T-PRI-005 – Controller-Status vor Auditstatus

### Änderungen

- `source.controller_status = draft`
- `source.audit_status = failed`

### Erwartung

```text
status = rejected
error_code = CONTROLLER_NOT_PREPARED
```

### Ergebnis

```text
not-run
```

---

## T-PRI-006 – Auditstatus vor Ziel-Allowlist

### Änderungen

- `source.audit_status = failed`
- `target.owner = OtherOwner`

### Erwartung

```text
status = rejected
error_code = AUDIT_NOT_PASSED
```

### Ergebnis

```text
not-run
```

---

## T-PRI-007 – Ziel-Allowlist vor Pfad

### Änderungen

- `target.owner = OtherOwner`
- `target.path = ../secret.txt`

### Erwartung

```text
status = rejected
error_code = TARGET_NOT_ALLOWED
```

### Ergebnis

```text
not-run
```

---

## T-PRI-008 – Pfad vor Referenz

### Änderungen

- `target.path = ../secret.txt`
- `target.ref = develop`

### Erwartung

```text
status = rejected
error_code = INVALID_PATH
```

### Ergebnis

```text
not-run
```

---

## T-PRI-009 – Referenz vor fehlendem SHA

### Änderungen

- `target.ref = develop`
- `source.expected_sha = ""`

### Erwartung

```text
status = rejected
error_code = INVALID_REF
```

### Ergebnis

```text
not-run
```

---

## T-PRI-010 – Fehlender SHA vor ungültigem Inhalt

### Änderungen

- `source.expected_sha = ""`
- `change.content` enthält ein Nullzeichen

### Erwartung

```text
status = rejected
error_code = SOURCE_SHA_MISSING
```

### Ergebnis

```text
not-run
```

---

## T-PRI-011 – Ungültiger Inhalt vor Größenfehler

### Änderung

Der Inhalt enthält ein Nullzeichen und überschreitet zugleich 100000 UTF-8-Bytes.

### Erwartung

```text
status = rejected
error_code = INVALID_CONTENT
```

### Ergebnis

```text
not-run
```

---

## T-PRI-012 – Inhaltsgröße vor Commit-Nachricht

### Änderungen

- Inhalt besitzt 100001 UTF-8-Bytes
- Commit-Nachricht ist leer

### Erwartung

```text
status = rejected
error_code = CONTENT_TOO_LARGE
```

### Ergebnis

```text
not-run
```

---

## T-PRI-013 – Eingangsanzahl vor fehlendem Feld

### Änderungen

- Der Workflow erhält zwei Eingangs-Items.
- In einem Auftrag fehlt `target.path`.

### Erwartung

```text
status = rejected
error_code = INVALID_INPUT_COUNT
```

### Ergebnis

```text
not-run
```

---

## T-PRI-014 – Fehlendes Feld vor ungültigem Datentyp

### Änderungen

- `target.path` entfernen
- `target.owner = 123`

### Erwartung

```text
status = rejected
error_code = MISSING_REQUIRED_FIELD
```

### Ergebnis

```text
not-run
```

---

## T-PRI-015 – Ungültiger SHA vor ungültigem Inhalt

### Änderungen

- `source.expected_sha = invalid-sha`
- `change.content` enthält ein Nullzeichen

### Erwartung

```text
status = rejected
error_code = INVALID_SOURCE_SHA
```

### Ergebnis

```text
not-run
```

---

## T-PRI-016 – Genau ein öffentlicher Fehler

### Eingang

Ein Auftrag mit mindestens fünf gleichzeitig erfüllten Fehlerbedingungen.

### Erwartung

```text
status = rejected
genau ein error_code
höchstpriorisierter Fehlercode
keine öffentliche Fehlerliste
keine rohen Prüfergebnisse
```

### Ergebnis

```text
not-run
```

---

## T-PRI-017 – Commit-Nachricht vor Patch-Validierung

### Änderungen

- Die Commit-Nachricht ist leer.
- Die Patch-Simulation würde zugleich fehlschlagen.

### Erwartung

```text
status = rejected
error_code = INVALID_COMMIT_MESSAGE
```

### Ergebnis

```text
not-run
```

---

## T-PRI-018 – Patch-Validierung vor internem Fehler

### Änderungen

- Die Patch-Validierung schlägt fehl.
- Zugleich liegt eine nachgelagerte interne Fehlerbedingung vor.

### Erwartung

```text
status = rejected
error_code = PATCH_VALIDATION_FAILED
```

### Ergebnis

```text
not-run
```

---


# M. Patch-Simulationstests

## T-PAT-001 – Patch-Typ

### Eingang

Gültiger Basisauftrag.

### Erwartung intern

```text
patch.patch_type = full-file-replacement
```

### Ergebnis

```text
not-run
```

---

## T-PAT-002 – Zielpfad im Patch

### Eingang

Gültiger Basisauftrag.

### Erwartung intern

```text
patch.target_path = test-fixtures/controller-target.md
```

### Ergebnis

```text
not-run
```

---

## T-PAT-003 – SHA im Patch

### Eingang

Gültiger Basisauftrag.

### Erwartung intern

```text
patch.expected_sha = 0123456789abcdef0123456789abcdef01234567
```

### Ergebnis

```text
not-run
```

---

## T-PAT-004 – UTF-8-Kodierung

### Erwartung intern

```text
patch.content_encoding = utf-8
```

### Ergebnis

```text
not-run
```

---

## T-PAT-005 – Patch nicht angewendet

### Erwartung intern

```text
patch.applied = false
```

### Erwartung öffentlich

```text
simulation.patch_applied = false
```

### Ergebnis

```text
not-run
```

---

## T-PAT-006 – Kein vollständiger Inhalt in Patch-Metadaten

### Prüfung

Das Patch-Metadatenobjekt enthält nicht:

```text
change.content
content
full_content
raw_content
```

### Erwartung

```text
vollständiger Inhalt nicht in Patch-Metadaten kopiert
```

### Ergebnis

```text
not-run
```

---

# N. Patch-Validierungstests

Diese Tests dürfen nur mit kontrollierter Manipulation der internen Testdaten durchgeführt werden. Sie dürfen keine Schreiboperation ergänzen.

## T-PVL-001 – Falscher Patch-Typ

### Manipulation

```text
patch.patch_type = unified-diff
```

### Erwartung

```text
status = rejected
error_code = PATCH_VALIDATION_FAILED
```

### Ergebnis

```text
not-run
```

---

## T-PVL-002 – Abweichender Zielpfad

### Manipulation

```text
patch.target_path = other/file.md
```

### Erwartung

```text
status = rejected
error_code = PATCH_VALIDATION_FAILED
```

### Ergebnis

```text
not-run
```

---

## T-PVL-003 – Abweichender SHA

### Manipulation

```text
patch.expected_sha = ffffffffffffffffffffffffffffffffffffffff
```

### Erwartung

```text
status = rejected
error_code = PATCH_VALIDATION_FAILED
```

### Ergebnis

```text
not-run
```

---

## T-PVL-004 – Falsche Inhaltskodierung

### Manipulation

```text
patch.content_encoding = base64
```

### Erwartung

```text
status = rejected
error_code = PATCH_VALIDATION_FAILED
```

### Ergebnis

```text
not-run
```

---

## T-PVL-005 – Falsche Byte-Länge

### Manipulation

```text
patch.content_bytes = 999
```

### Erwartung

```text
status = rejected
error_code = PATCH_VALIDATION_FAILED
```

### Ergebnis

```text
not-run
```

---

## T-PVL-006 – applied ist true

### Manipulation

```text
patch.applied = true
```

### Erwartung

```text
status = rejected
error_code = PATCH_VALIDATION_FAILED
file_changed = false
write_executed = false
```

### Ergebnis

```text
not-run
```

---

## T-PVL-007 – Schreibstatus vorhanden

### Manipulation

```json
{
  "patch": {
    "write_status": "completed"
  }
}
```

### Erwartung

```text
status = rejected
error_code = PATCH_VALIDATION_FAILED
```

### Ergebnis

```text
not-run
```

---

## T-PVL-008 – Commit-ID vorhanden

### Manipulation

```json
{
  "patch": {
    "commit_id": "abc123"
  }
}
```

### Erwartung

```text
status = rejected
error_code = PATCH_VALIDATION_FAILED
```

### Ergebnis

```text
not-run
```

---

## T-PVL-009 – Push-Ergebnis vorhanden

### Manipulation

```json
{
  "patch": {
    "push_result": "success"
  }
}
```

### Erwartung

```text
status = rejected
error_code = PATCH_VALIDATION_FAILED
```

### Ergebnis

```text
not-run
```

---

## T-PVL-010 – Credential-Daten vorhanden

### Manipulation

Das Patch-Objekt wird um einen eindeutig als Testwert markierten Credential-Platzhalter ergänzt.

### Erwartung

```text
status = rejected
error_code = PATCH_VALIDATION_FAILED
Credential-Testwert nicht im Endergebnis
```

### Ergebnis

```text
not-run
```

---

# O. Sanitizer-Tests

## T-SAN-001 – Erfolgs-Allowlist

### Eingang

Gültiger Basisauftrag mit zusätzlichen unbekannten Feldern.

### Erwartung

Das Endergebnis enthält ausschließlich die für Erfolg erlaubten Felder.

### Ergebnis

```text
not-run
```

---

## T-SAN-002 – Fehler-Allowlist

### Eingang

Basisauftrag mit:

```text
target.path = ../secret.txt
```

### Erwartung

Das Endergebnis enthält ausschließlich:

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

### Ergebnis

```text
not-run
```

---

## T-SAN-003 – Inhalt wird entfernt

### Eingang

Der Dateiinhalt enthält einen eindeutigen Testmarker:

```text
SANITIZER-CONTENT-MARKER-001
```

### Erwartung

Der Marker erscheint nicht:

- im Endergebnis,
- in `message`,
- in `error_code`,
- in öffentlich weitergegebenen Objekten.

### Ergebnis

```text
not-run
```

---

## T-SAN-004 – approved_by wird entfernt

### Eingang

```text
source.approved_by = manual_review
```

### Erwartung

```text
source.approved_by nicht im Endergebnis
```

### Ergebnis

```text
not-run
```

---

## T-SAN-005 – Interne Prüfobjekte werden entfernt

### Prüfung

Das Endergebnis enthält keine:

```text
checks
normalized
errors
decision
patch
patch_check
metadata
```

### Ergebnis

```text
not-run
```

---

## T-SAN-006 – Sicherheitsflags werden überschrieben

### Kontrollierte Manipulation vor dem Sanitizer

```json
{
  "file_changed": true,
  "commit_created": true,
  "push_executed": true,
  "write_executed": true
}
```

### Erwartung nach Sanitizer

```json
{
  "file_changed": false,
  "commit_created": false,
  "push_executed": false,
  "write_executed": false
}
```

### Ergebnis

```text
not-run
```

---

## T-SAN-007 – Credential-Marker wird entfernt

### Kontrollierter Testmarker

```text
TEST-AUTHORIZATION-MARKER-001
```

### Erwartung

Der Marker erscheint nicht im bereinigten Endergebnis.

Es dürfen keine realen Zugangsdaten verwendet werden.

### Ergebnis

```text
not-run
```

---

## T-SAN-008 – Stacktrace wird entfernt

### Kontrollierte interne Fehlerstruktur

```json
{
  "stack": "TEST-STACKTRACE-MARKER-001"
}
```

### Erwartung

```text
status = rejected
error_code = INTERNAL_ERROR
Stacktrace-Marker nicht im Endergebnis
```

### Ergebnis

```text
not-run
```

---

## T-SAN-009 – Genau ein End-Item

### Prüfung

Erfolgsweg und alle Ablehnungswege erzeugen jeweils:

```text
genau 1 End-Item
```

### Ergebnis

```text
not-run
```

---

# P. Determinismustests

## T-DET-001 – Gleicher Eingang, gleiches Ergebnis

### Durchführung

Der identische Basisauftrag wird dreimal manuell ausgeführt.

### Erwartung

Alle drei bereinigten Endergebnisse sind fachlich identisch.

### Nicht zulässig

- wechselnde IDs,
- wechselnde Statuswerte,
- Zeitstempel im Ergebnis,
- zufällige Werte,
- wechselnde Standardwerte.

### Ergebnis

```text
not-run
```

---

## T-DET-002 – Gleiche Mehrfachfehler, gleiche Priorität

### Durchführung

Ein identischer Auftrag mit mehreren Fehlern wird dreimal ausgeführt.

### Erwartung

In allen Ausführungen erscheint derselbe höchstpriorisierte Fehlercode.

### Ergebnis

```text
not-run
```

---

## T-DET-003 – Keine automatisch erzeugte Request-ID

### Eingang

Gültiger Basisauftrag ohne `request_id`.

### Durchführung

Dreimal ausführen.

### Erwartung

```text
status = simulated
request_id in keiner Ausgabe vorhanden
```

### Ergebnis

```text
not-run
```

---

# Q. Seiteneffektfreiheitstests

## T-SEF-001 – Keine Dateiänderung

### Prüfung vor und nach der Ausführung

Es wird kontrolliert, dass keine Datei:

- erstellt,
- verändert,
- gelöscht,
- umbenannt

wurde.

### Erwartung

```text
keine Dateiänderung
file_changed = false
write_executed = false
```

### Ergebnis

```text
not-run
```

---

## T-SEF-002 – Kein Commit

### Prüfung

Vor und nach der Ausführung existiert kein durch WF-0011 erzeugter Commit.

### Erwartung

```text
commit_created = false
```

### Ergebnis

```text
not-run
```

---

## T-SEF-003 – Kein Push

### Prüfung

WF-0011 führt keinen Push und keinen externen Schreibaufruf aus.

### Erwartung

```text
push_executed = false
```

### Ergebnis

```text
not-run
```

---

## T-SEF-004 – Kein GitHub-Aufruf

### Prüfung

Die Ausführung enthält keinen Aufruf an die GitHub-API.

### Erwartung

```text
0 GitHub-Aufrufe
```

### Ergebnis

```text
not-run
```

---

## T-SEF-005 – Kein weiterer Workflow gestartet

### Prüfung

WF-0011 startet keinen anderen Workflow.

### Erwartung

```text
0 automatisch gestartete Folge-Workflows
```

### Ergebnis

```text
not-run
```

---

## T-SEF-006 – Kein Shell-Befehl

### Prüfung

Es wird kein Shell- oder Systembefehl ausgeführt.

### Erwartung

```text
0 Shell-Ausführungen
```

### Ergebnis

```text
not-run
```

---

# R. Credential- und Exporttests

## T-EXP-001 – Keine Credentials im Workflow

### Prüfung

Die Credential-Zuordnungen aller Nodes werden geprüft.

### Erwartung

```text
eingebundene Credentials = 0
```

### Ergebnis

```text
not-run
```

---

## T-EXP-002 – Bereinigter Export enthält keine Credential-ID

### Prüfung

Der Workflow-Export wird nach Credential-Feldern und Credential-IDs geprüft.

### Erwartung

```text
keine Credential-ID
keine Credential-Referenz
kein Token
kein Secret
kein Authorization-Header
```

### Ergebnis

```text
not-run
```

---

## T-EXP-003 – Export enthält keine verbotenen Nodes

### Prüfung

Der bereinigte Export wird auf verbotene Node-Typen geprüft.

### Erwartung

```text
0 verbotene Nodes
```

### Ergebnis

```text
not-run
```

---

## T-EXP-004 – Workflow ist inaktiv

### Prüfung

Der Export weist den Workflow als inaktiv aus.

### Erwartung

```text
active = false
```

### Ergebnis

```text
not-run
```

---

## T-EXP-005 – Keine realen Testdaten im Export

### Prüfung

Der Export enthält keine:

- realen Tokens,
- realen Kundendaten,
- produktiven Dateiänderungen,
- vertraulichen Inhalte.

### Erwartung

```text
nur kontrollierte Testdaten oder keine gespeicherten Ausführungsdaten
```

### Ergebnis

```text
not-run
```

---

# S. n8n-Datenhaltungstests

## T-DAT-001 – Erfolgreiche Ausführung

### Prüfung

Es wird festgestellt, welche Node-Daten n8n bei einer erfolgreichen Ausführung speichert.

Besonders geprüft wird:

```text
change.content
canonical request
patch metadata
interne checks
```

### Erwartung

Das Ergebnis wird dokumentiert. Bis zur abgeschlossenen Bewertung bleibt der Workflow im Status:

```text
draft
```

### Ergebnis

```text
not-run
```

---

## T-DAT-002 – Fehlerhafte Ausführung

### Prüfung

Es wird festgestellt, welche Daten n8n bei einer abgelehnten oder technisch fehlgeschlagenen Ausführung speichert.

Besonders geprüft wird:

```text
rohe Eingangsdaten
Fehlerobjekte
Stacktraces
Node-Konfiguration
vollständiger Inhalt
```

### Erwartung

Keine ungeprüfte Freigabe für produktive oder vertrauliche Inhalte.

### Ergebnis

```text
not-run
```

---

## T-DAT-003 – Speicherungseinstellungen

### Prüfung

Folgende n8n-Einstellungen werden dokumentiert:

```text
Speicherung erfolgreicher Ausführungen
Speicherung fehlerhafter Ausführungen
manuelle Ausführungsdaten
Aufbewahrungsdauer
Pruning oder automatische Löschung
```

### Erwartung

Die Einstellungen sind bekannt und für den Simulationsbetrieb bewertet.

### Ergebnis

```text
not-run
```

---

## T-DAT-004 – Keine vertraulichen Testinhalte

### Prüfung

Alle bis zur abgeschlossenen Datenhaltungsprüfung verwendeten Inhalte sind künstlich und nicht vertraulich.

### Erwartung

```text
keine vertraulichen Daten verwendet
```

### Ergebnis

```text
not-run
```

---

# T. End-to-End-Tests

## T-E2E-001 – Vollständiger Erfolgsweg

### Eingang

Gültiger Basisauftrag.

### Erwartung

```text
status = simulated
End-Items = 1
Patch Simulator erreicht
Success Builder erreicht
Rejection Builder nicht erreicht
Sanitizer erreicht
Final Output erreicht
keine Seiteneffekte
```

### Ergebnis

```text
not-run
```

---

## T-E2E-002 – Ablehnung vor Simulation

### Eingang

```text
target.path = ../secret.txt
```

### Erwartung

```text
status = rejected
error_code = INVALID_PATH
Patch Simulator nicht erreicht
Rejection Builder erreicht
Sanitizer erreicht
Final Output erreicht
End-Items = 1
keine Seiteneffekte
```

### Ergebnis

```text
not-run
```

---

## T-E2E-003 – Ablehnung nach Patch-Simulation

### Eingang

Gültiger Basisauftrag mit kontrolliert manipulierten Patch-Metadaten.

### Erwartung

```text
status = rejected
error_code = PATCH_VALIDATION_FAILED
Patch Simulator erreicht
Patch Validator erreicht
Success Builder nicht erreicht
Rejection Builder erreicht
Sanitizer erreicht
Final Output erreicht
End-Items = 1
keine Seiteneffekte
```

### Ergebnis

```text
not-run
```

---

## T-E2E-004 – Sicher abgefangener interner Fehler

### Durchführung

Ein kontrollierter technischer Testfehler wird innerhalb eines dafür vorbereiteten Testzustands ausgelöst.

Der Test darf keine Schreibkomponente ergänzen.

### Erwartung

```text
status = rejected
error_code = INTERNAL_ERROR
message = Die Simulation konnte nicht sicher abgeschlossen werden.
kein Stacktrace
kein Eingabedump
keine Node-Konfiguration
End-Items = 1
```

### Ergebnis

```text
not-run
```

---

## T-E2E-005 – Sanitizer auf allen Ergebniswegen

### Prüfung

Folgende Wege werden einzeln ausgeführt:

```text
Erfolg
Schema-Ablehnung
Sicherheitsablehnung
Patch-Ablehnung
interner Fehler
```

### Erwartung

Jeder Weg erreicht:

```text
80_OUTPUT_SANITIZER
90_FINAL_OUTPUT
```

Kein Weg umgeht den Sanitizer.

### Ergebnis

```text
not-run
```

---

## 9. Verbindliche Fehlercodes

Folgende Fehlercodes müssen durch mindestens einen Test abgedeckt sein:

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

Es dürfen keine nicht dokumentierten öffentlichen Fehlercodes entstehen.

---

## 10. Verbindliche Fehlertexte

Die Ablehnungen müssen exakt die statischen Meldungen aus `FLOW_v0.2.0.md` verwenden.

Dynamische Ergänzungen mit Eingangswerten sind nicht zulässig.

Insbesondere dürfen Meldungen nicht enthalten:

- Zielpfade,
- Dateiinhalt,
- Commit-Nachrichten,
- Credentials,
- Tokens,
- rohe Fehlerobjekte,
- Stacktraces.

---

## 11. Mindestabdeckung vor Status testing

Vor dem Wechsel von `draft` zu `testing` müssen mindestens erfüllt sein:

```text
Struktur vollständig geprüft
keine verbotenen Nodes
keine Credentials
Basisauftrag erfolgreich
alle Fehlercodes technisch erreichbar
Pfadsicherheitstests vorbereitet
Sanitizer auf allen Wegen verbunden
Seiteneffektfreiheit technisch nachvollziehbar
n8n-Datenhaltung geprüft
Workflow weiterhin inaktiv
```

---

## 12. Mindestabdeckung vor Status passed

Vor dem Gesamtstatus `passed` müssen:

- alle Testfälle ausgeführt sein,
- alle Pflichtfälle den Status `passed` besitzen,
- keine ungeklärten Abweichungen bestehen,
- alle Fehlercodes reproduzierbar sein,
- die Fehlerpriorität bestätigt sein,
- alle Pfadsicherheitstests bestanden sein,
- alle Grenzwerttests bestanden sein,
- der Sanitizer keine internen Daten ausgeben,
- genau ein End-Item entstehen,
- der Workflow deterministisch arbeiten,
- die Seiteneffektfreiheit bestätigt sein,
- der bereinigte Export geprüft sein,
- die Credential-Anzahl `0` betragen,
- der Workflow inaktiv bleiben.

---

## 13. Abbruchkriterien

Die Tests werden sofort gestoppt, wenn:

```text
eine Datei verändert wird
eine Datei erstellt oder gelöscht wird
ein GitHub-Schreibaufruf erfolgt
ein Commit erstellt wird
ein Push ausgeführt wird
ein Credential eingebunden ist
ein Token oder Secret sichtbar wird
ein automatischer Trigger aktiv ist
ein anderer Workflow automatisch gestartet wird
der Sanitizer umgangen wird
ein technischer Fehler ungefiltert ausgegeben wird
```

Nach einem Abbruch bleibt der Workflow:

```text
inactive
draft
not-approved
```

Der Vorfall wird in `KNOWN_ISSUES.md` dokumentiert.

---

## 14. Testabschlussprotokoll

Nach Abschluss aller Tests wird folgender Block ausgefüllt:

```text
Workflow: WF-0011 – GitHub Writer
Version: 0.2.0
Mode: simulation
Testdatum:
Getestet durch:
Gesamtzahl Testfälle:
Passed:
Failed:
Blocked:
Not applicable:
End-to-End-Erfolg:
Fehlerpriorität bestätigt:
Pfadsicherheit bestätigt:
Sanitizer bestätigt:
Seiteneffektfreiheit bestätigt:
Credentials im Export:
Verbotene Nodes:
Workflow aktiv:
Gesamtstatus:
Freigabe:
Notizen:
```

---

## 15. Definition of Done für die Tests

`TESTS_v0.2.0.md` gilt als vollständig umgesetzt, wenn:

- alle dokumentierten Testbereiche ausgeführt wurden,
- der gültige Basisauftrag den Status `simulated` erzeugt,
- alle ungültigen Aufträge sicher abgelehnt werden,
- jeder Fehlercode reproduzierbar ausgelöst werden kann,
- die Fehlerpriorität exakt eingehalten wird,
- gefährliche und mehrdeutige Pfade abgelehnt werden,
- Inhalts- und Nachrichtenlimits korrekt greifen,
- manipulierte Patch-Metadaten abgelehnt werden,
- alle Ergebnisse den Output Sanitizer durchlaufen,
- keine internen oder vertraulichen Daten öffentlich ausgegeben werden,
- genau ein Endergebnis entsteht,
- keine Datei verändert wird,
- kein Commit und kein Push ausgeführt wird,
- keine externe GitHub-Abfrage stattfindet,
- keine Credentials eingebunden sind,
- der bereinigte Export sicher geprüft wurde,
- die n8n-Datenhaltung bewertet wurde,
- der Workflow weiterhin inaktiv ist.

---

## 16. Aktueller Teststatus

```text
Version: 0.2.0
Status: draft
Mode: simulation
Test specification: defined
Implementation: not-started
Tests executed: 0
Tests passed: 0
Tests failed: 0
Credentials: forbidden
Side effects: forbidden
Workflow activation: forbidden
```

---

## 17. Nächster konkreter Schritt

```text
Die Dokumente SPECIFICATION_v0.2.0.md,
ARCHITECTURE_v0.2.0.md, FLOW_v0.2.0.md und
TESTS_v0.2.0.md gemeinsam auf Widersprüche prüfen.

Erst nach bestandenem Dokumentationsabgleich darf der
inaktive n8n-Workflow WF-0011 v0.2.0 angelegt werden.
```
