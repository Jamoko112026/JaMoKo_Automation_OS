# WF-0011 – Specification

## Version

0.1.0

## Status

draft

---

## Zweck

WF-0011 setzt freigegebene Änderungen kontrolliert im GitHub-Repository des JaMoKo OS um.

Version 0.1.0 arbeitet ausschließlich im Betriebsmodus `simulation`.

Der Workflow erzeugt eine Änderungsvorschau und einen validierten Patch, verändert jedoch keine Datei, erstellt keinen Commit und führt keinen Push aus.

---

## Eingangsdaten

WF-0011 erwartet genau einen freigegebenen Änderungsvorschlag pro Lauf.

Beispiel:

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

## Pflichtfelder

Folgende Felder müssen vorhanden und gültig sein:

| Feld | Beschreibung |
|---|---|
| `objectId` | Eindeutige ID des zu ändernden Objekts |
| `path` | Relativer Pfad der Zieldatei im Repository |
| `field` | Zu änderndes Feld |
| `currentValue` | Erwarteter aktueller Wert |
| `approvedValue` | Freigegebener neuer Wert |
| `approvalStatus` | Status der manuellen Freigabe |
| `sourceSha` | SHA des geprüften Ausgangsstands |
| `approvedBy` | Herkunft oder Instanz der Freigabe |
| `auditStatus` | Ergebnis der vorgelagerten Prüfung |

---

## Annahmebedingungen

Ein Änderungsvorschlag wird nur verarbeitet, wenn alle folgenden Bedingungen erfüllt sind:

- Es liegt genau ein Änderungsvorschlag vor.
- `objectId` entspricht einer gültigen JaMoKo-Objekt-ID.
- `path` ist ein relativer Pfad innerhalb des erlaubten Repository-Bereichs.
- `field` ist vorhanden und für die Änderung zugelassen.
- `approvalStatus` ist `approved`.
- `auditStatus` ist `passed`.
- `sourceSha` ist vorhanden.
- `approvedBy` ist dokumentiert.
- Der Betriebsmodus ist `simulation`.

Wird eine Bedingung nicht erfüllt, wird die Verarbeitung kontrolliert abgebrochen.

---

## Sicherheitsregeln

WF-0011 folgt DEC-0009 – Writer Safety Modes.

Für Version 0.1.0 gelten verbindlich folgende Regeln:

- Der Betriebsmodus ist fest auf `simulation` gesetzt.
- Es findet kein Schreibzugriff auf das Dateisystem statt.
- Es werden keine Repository-Dateien verändert.
- Es werden keine Git-Commits erstellt.
- Es wird kein Push ausgeführt.
- Es werden keine Branches erstellt oder verändert.
- Es werden keine GitHub-API-Schreiboperationen ausgeführt.
- Absolute Pfade sind nicht zulässig.
- Pfade außerhalb des freigegebenen Repository-Bereichs sind nicht zulässig.
- Nicht freigegebene oder nicht auditierte Änderungen werden abgewiesen.
- Der `sourceSha` wird als Referenz des geprüften Ausgangsstands dokumentiert.

---

## Verarbeitung

WF-0011 führt in Version 0.1.0 folgende Schritte aus:

1. Eingangsdaten übernehmen.
2. Pflichtfelder validieren.
3. Freigabestatus prüfen.
4. Auditstatus prüfen.
5. Betriebsmodus prüfen.
6. Objekt-ID und Zielpfad validieren.
7. Änderung aus `currentValue` und `approvedValue` ableiten.
8. Änderungsvorschau erzeugen.
9. Simulierten Patch erzeugen.
10. Ergebnisobjekt mit Prüf- und Statusinformationen ausgeben.

---

## Änderungsvorschau

Die Änderungsvorschau beschreibt mindestens:

- betroffenes Objekt,
- betroffene Datei,
- betroffenes Feld,
- erwarteten Ausgangswert,
- freigegebenen Zielwert,
- zugrunde liegenden `sourceSha`,
- Freigabeinstanz,
- Auditstatus,
- Betriebsmodus.

Die Vorschau muss eindeutig erkennen lassen, dass keine tatsächliche Änderung durchgeführt wurde.

---

## Patch

Der Workflow erzeugt einen validierten, aber nicht angewendeten Patch.

Der Patch dient ausschließlich:

- der menschlichen Prüfung,
- der technischen Validierung,
- der Vorbereitung späterer Writer-Versionen,
- der nachvollziehbaren Dokumentation der vorgesehenen Änderung.

Der Patch darf in Version 0.1.0 nicht auf eine Datei angewendet werden.

---

## Ausgangsdaten

Beispiel eines erfolgreichen Simulationsergebnisses:

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

## Fehlerausgabe

Bei einer abgewiesenen Verarbeitung muss das Ergebnis mindestens enthalten:

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
| `INVALID_INPUT_COUNT` | Es liegt nicht genau ein Änderungsvorschlag vor |
| `MISSING_REQUIRED_FIELD` | Ein Pflichtfeld fehlt |
| `INVALID_OBJECT_ID` | Die Objekt-ID ist ungültig |
| `INVALID_PATH` | Der Zielpfad ist ungültig oder nicht zulässig |
| `INVALID_FIELD` | Das angegebene Feld ist nicht zulässig |
| `APPROVAL_REQUIRED` | Der Änderungsvorschlag ist nicht freigegeben |
| `AUDIT_NOT_PASSED` | Die vorgelagerte Prüfung wurde nicht bestanden |
| `SOURCE_SHA_MISSING` | Der geprüfte Ausgangsstand ist nicht dokumentiert |
| `INVALID_MODE` | Der Betriebsmodus ist nicht `simulation` |
| `PATCH_VALIDATION_FAILED` | Der simulierte Patch konnte nicht validiert werden |

---

## Abgrenzung

Version 0.1.0 übernimmt nicht:

- das tatsächliche Schreiben einer Datei,
- das Anwenden des erzeugten Patches,
- das Erstellen eines Git-Branches,
- das Erstellen eines Commits,
- das Ausführen eines Pushs,
- das Erstellen eines Pull Requests,
- das Zusammenführen von Änderungen,
- die Verarbeitung mehrerer Änderungsvorschläge in einem Lauf.

Diese Funktionen sind möglichen späteren Versionen vorbehalten.

---

## Abhängigkeiten

WF-0011 setzt voraus:

- einen Änderungsvorschlag aus WF-0009,
- ein bestandenes Audit aus WF-0010,
- eine dokumentierte manuelle Freigabe,
- DEC-0007 – Workflow Pipeline Architecture,
- DEC-0008 – Workflow ID Governance,
- DEC-0009 – Writer Safety Modes,
- STD-0003 – Workflow Architecture Standard.

---

## Erfolgskriterium

WF-0011 gilt in Version 0.1.0 als erfolgreich ausgeführt, wenn:

- der Eingang vollständig validiert wurde,
- Freigabe und Audit bestätigt wurden,
- der Betriebsmodus `simulation` eingehalten wurde,
- eine nachvollziehbare Änderungsvorschau erzeugt wurde,
- ein gültiger simulierter Patch erzeugt wurde,
- keine Datei verändert wurde,
- kein Commit erstellt wurde,
- kein Push ausgeführt wurde.