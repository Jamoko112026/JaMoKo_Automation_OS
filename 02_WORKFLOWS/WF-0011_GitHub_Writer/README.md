# WF-0011 – GitHub Writer

## Status

draft

## Version

0.1.0

## Typ

Writer Workflow

## Pipeline-Rolle

Writer – Simulation

---

## Zweck

WF-0011 bereitet freigegebene Änderungen für die kontrollierte Übernahme in das GitHub-Repository des JaMoKo OS vor.

In Version 0.1.0 arbeitet der Workflow ausschließlich im Modus `simulation`. Er prüft einen Änderungsvorschlag, erstellt einen Change Plan und erzeugt eine Patch-Vorschau.

Es werden keine Repository-Dateien verändert.

---

## Ziel

WF-0011 soll nachweisen, dass freigegebene Änderungen sicher, nachvollziehbar und reproduzierbar verarbeitet werden können.

Vor einer späteren Schreibfreigabe müssen insbesondere folgende Punkte geprüft werden:

- Vollständigkeit der Eingangsdaten,
- dokumentierte manuelle Freigabe,
- bestandener Audit,
- gültige Objekt-ID,
- sicherer relativer Repository-Pfad,
- zulässiges Zielfeld,
- vorhandener Ausgangs-SHA,
- nachvollziehbarer Change Plan,
- gültige Patch-Simulation,
- vollständiger Ausschluss einer Schreibwirkung.

---

## Eingang

WF-0011 verarbeitet genau einen geprüften und freigegebenen Änderungsvorschlag.

Mögliche Quellen sind:

- WF-0009 – Object Repair Engine,
- WF-0010 – Object Auditor,
- eine dokumentierte manuelle Freigabe.

### Beispiel

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

## Pflichtfelder

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
mode
```

`currentValue` darf den Wert `null` besitzen.

Alle übrigen Pflichtwerte müssen vorhanden und eindeutig auswertbar sein.

---

## Verarbeitung

Der Workflow führt die folgenden Schritte aus:

1. Eingangsdaten übernehmen und normalisieren.
2. Anzahl und Vollständigkeit der Eingaben prüfen.
3. Manuelle Freigabe validieren.
4. Auditstatus prüfen.
5. Betriebsmodus und Sicherheitsbedingungen validieren.
6. Change Plan erzeugen.
7. Patch strukturiert simulieren.
8. Patch-Simulation validieren.
9. Ein einheitliches Erfolgs- oder Ablehnungsergebnis erzeugen.

---

## Ergebnis bei erfolgreicher Simulation

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

## Ergebnis bei Ablehnung

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

## Fehlercodes

| Fehlercode | Bedeutung |
|---|---|
| `INVALID_INPUT_COUNT` | Es liegt nicht genau ein Änderungsvorschlag vor. |
| `MISSING_REQUIRED_FIELD` | Ein Pflichtfeld fehlt oder enthält keinen zulässigen Wert. |
| `APPROVAL_REQUIRED` | Der Änderungsvorschlag wurde nicht freigegeben. |
| `AUDIT_NOT_PASSED` | Der Audit wurde nicht bestanden. |
| `INVALID_MODE` | Der Workflow wurde nicht im Modus `simulation` gestartet. |
| `INVALID_OBJECT_ID` | Die Objekt-ID entspricht nicht dem JaMoKo-ID-Schema. |
| `INVALID_PATH` | Der Zielpfad ist ungültig oder unsicher. |
| `INVALID_FIELD` | Das Zielfeld ist ungültig oder nicht zugelassen. |
| `SOURCE_SHA_MISSING` | Der dokumentierte Ausgangs-SHA fehlt. |
| `PATCH_VALIDATION_FAILED` | Die simulierte Änderung erfüllt die Sicherheitsbedingungen nicht. |

---

## Sicherheitsgarantie

WF-0011 v0.1.0:

- liest keine Repository-Datei,
- verändert keine Repository-Datei,
- führt keine Git-Befehle aus,
- erstellt keinen Branch,
- erstellt keinen Commit,
- führt keinen Push aus,
- erstellt keinen Pull Request,
- verwendet keine GitHub-API mit Schreibrechten.

Diese Werte müssen bei jedem Ergebnis enthalten sein:

```json
{
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

---

## Abgrenzung

Nicht Bestandteil von Version 0.1.0 sind:

- tatsächlicher Zugriff auf Repository-Inhalte,
- Abgleich des `sourceSha` mit GitHub,
- Anwendung eines Patches,
- Verarbeitung mehrerer Änderungen,
- Branch-Erstellung,
- Commit-Erstellung,
- Push-Ausführung,
- Pull-Request-Erstellung.

Diese Funktionen dürfen erst in einer späteren Version nach eigener Sicherheitsprüfung und dokumentierter Freigabe ergänzt werden.

---

## Dokumentation

Die Workflow-Akte besteht aus:

- `README.md` – Übersicht und Einstieg,
- `SPECIFICATION.md` – fachliche Anforderungen,
- `ARCHITECTURE.md` – Aufbau und Sicherheitsarchitektur,
- `FLOW.md` – Node- und Verbindungsplan,
- `TESTS.md` – Testfälle und Abnahmekriterien,
- `CHANGELOG.md` – Versionshistorie,
- `KNOWN_ISSUES.md` – bekannte Einschränkungen,
- `exports/` – spätere n8n-Exporte,
- `screenshots/` – spätere visuelle Nachweise.

---

## Aktueller Stand

Die fachliche Grundlage für WF-0011 v0.1.0 ist dokumentiert.

Der n8n-Workflow wurde noch nicht aufgebaut. Die 17 definierten Testfälle wurden deshalb noch nicht ausgeführt.

---

## Nächster Schritt

WF-0011 v0.1.0 wird in n8n anhand von `FLOW.md` aufgebaut.

Anschließend werden die Testfälle aus `TESTS.md` ausgeführt und im Testprotokoll dokumentiert.