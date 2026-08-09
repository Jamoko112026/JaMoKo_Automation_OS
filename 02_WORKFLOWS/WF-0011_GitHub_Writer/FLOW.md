# WF-0011 – Flow

## Version

0.1.0

## Status

released

---

## Zweck

Dieses Dokument beschreibt den tatsächlich exportierten Ablauf von WF-0011 –
GitHub Writer v0.1.0.

Der deaktivierte Workflow arbeitet ausschließlich im Modus `simulation`. Er
erzeugt lokal einen festen Testdatensatz, prüft ihn in einem monolithischen
Code-Knoten und erzeugt einen Change Plan sowie eine nicht angewendete
Patch-Vorschau. Er greift nicht auf GitHub oder andere externe Systeme zu.

---

## Tatsächliche Knotenstruktur

Der Export `exports/WF-0011_GitHub_Writer_v0.1.0.json` enthält genau drei
Knoten:

| Nr. | Exportierter Knotenname | n8n-Typ | Aufgabe |
|---:|---|---|---|
| 1 | `01 – Manual Trigger` | Manual Trigger | startet den lokalen Lauf manuell |
| 2 | `02 – Test Input` | Code | erzeugt genau einen festen lokalen Testdatensatz |
| 3 | `03 – Validate and Simulate` | Code | validiert den Datensatz monolithisch, erzeugt `changePlan` und `patchPreview` und baut genau ein Ergebnis |

Es gibt keine getrennten Knoten für Normalisierung, einzelne Gates,
Patch-Simulation, Verzweigungen oder Ergebnisaufbau. Diese fachlichen Schritte
sind vollständig im dritten Code-Knoten zusammengefasst.

---

## Verbindungsplan

```text
01 – Manual Trigger
        |
        v
02 – Test Input
        |
        v
03 – Validate and Simulate
        |
        +-- kontrollierte Ablehnung: genau ein rejected-Ergebnis
        |
        +-- erfolgreiche Simulation: genau ein simulated-Ergebnis
```

Der Export ist deaktiviert (`active: false`).

---

## Knoten 1 – `01 – Manual Trigger`

Der Knoten startet den Workflow ausschließlich manuell. Der Export enthält
keinen Webhook, keinen Zeitplan und keinen Repository-Trigger.

---

## Knoten 2 – `02 – Test Input`

Der Code-Knoten erzeugt genau einen festen Testdatensatz:

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

Der Export besitzt keine technische Anbindung an WF-0009, WF-0010, WF-0013
oder eine andere externe Eingangsquelle.

---

## Knoten 3 – `03 – Validate and Simulate`

Der Code-Knoten liest alle eingehenden Items und führt die folgenden Prüfungen
und Ableitungen in dieser Reihenfolge aus:

1. Genau ein Eingangs-Item muss vorhanden sein.
2. `sourceSha` muss vorhanden und ein nichtleerer String sein.
3. Alle Pflichtfelder müssen vorhanden sein. Außer `currentValue` dürfen sie
   nicht `null`, `undefined` oder nach String-Konvertierung leer sein.
4. `approvalStatus` muss exakt `approved` sein.
5. `auditStatus` muss exakt `passed` sein.
6. `mode` muss exakt `simulation` sein.
7. `objectId` muss dem Muster `^[A-Z][A-Z0-9]*-\d{4}$` entsprechen.
8. `path` muss die tatsächlich implementierten Pfadregeln erfüllen.
9. `field` muss einer der Werte `status`, `name`, `description` oder `version`
   sein.
10. Der Knoten erzeugt `changePlan` und `patchPreview` ausschließlich im
    Arbeitsspeicher.
11. Der intern erzeugte Patch wird auf Übereinstimmung ausgewählter Werte und
    `patchPreview.applied === false` geprüft.
12. Der Knoten gibt genau ein strukturiertes Erfolgs- oder Ablehnungsobjekt aus.

Es findet keine Normalisierung der Eingangswerte statt. `sourceSha.trim()` und
`String(...).trim()` werden nur zur Leerheitsprüfung verwendet; die ausgegebenen
Werte werden dadurch nicht ersetzt.

---

## Tatsächlich implementierte Pfadprüfung

`path` wird akzeptiert, wenn alle folgenden Bedingungen erfüllt sind:

- Datentyp String,
- Endung `.md`,
- beginnt weder mit `/` noch mit `~`,
- enthält weder `..` noch `\`,
- enthält weder `://` noch ein Nullbyte.

Der Export enthält keine ausdrückliche Prüfung auf `.git` oder andere
geschützte Repository-Bereiche. `.git/config` wird wegen der fehlenden
`.md`-Endung abgelehnt; daraus folgt keine allgemeine Sperre für `.git`-Pfade.

---

## Change Plan

Nach erfolgreicher Validierung entsteht lokal:

```json
{
  "operation": "replace_field_value",
  "objectId": "FIN-0001",
  "path": "01_Objects/Finance/FIN-0001/object.md",
  "field": "status",
  "from": null,
  "to": "active",
  "sourceSha": "abc123"
}
```

Der Change Plan beschreibt nur eine beabsichtigte Feldwertänderung. Er wird
nicht auf eine Datei angewendet.

---

## Patch-Vorschau

Der gleiche Code-Knoten erzeugt lokal:

```json
{
  "format": "structured_preview",
  "target": "01_Objects/Finance/FIN-0001/object.md",
  "field": "status",
  "before": null,
  "after": "active",
  "applied": false
}
```

Die Patch-Vorschau ist kein Datei-Patch und enthält keine ausführbare
Schreiboperation.

---

## Kontrollierte Fehlercodes

| Fehlercode | Tatsächlich geprüfte Bedingung |
|---|---|
| `INVALID_INPUT_COUNT` | Eingangsanzahl ist nicht genau eins |
| `SOURCE_SHA_MISSING` | `sourceSha` fehlt, ist kein String oder ist leer |
| `MISSING_REQUIRED_FIELD` | Pflichtfeld fehlt oder besitzt keinen zulässigen Wert |
| `APPROVAL_REQUIRED` | `approvalStatus` ist nicht `approved` |
| `AUDIT_NOT_PASSED` | `auditStatus` ist nicht `passed` |
| `INVALID_MODE` | `mode` ist nicht `simulation` |
| `INVALID_OBJECT_ID` | `objectId` erfüllt das Muster nicht |
| `INVALID_PATH` | `path` erfüllt eine implementierte Pfadbedingung nicht |
| `INVALID_FIELD` | `field` steht nicht in der Allowlist |
| `PATCH_VALIDATION_FAILED` | die intern erzeugte Vorschau erfüllt die Patch-Prüfung nicht |

Kontrollierte Ablehnungen enthalten `status: rejected` und alle drei
Schreibschutzwerte `false`. Ein zentraler sicherer Fehlerpfad für unerwartete
JavaScript-Laufzeitfehler ist nicht implementiert.

---

## Erfolgsbedingungen

Ein Lauf erhält `status: simulated`, wenn alle Validierungen erfolgreich sind,
`changePlan` und `patchPreview` erzeugt wurden und die interne Patch-Prüfung
bestanden ist.

Das Ergebnis enthält insbesondere:

```json
{
  "workflowId": "WF-0011",
  "version": "0.1.0",
  "mode": "simulation",
  "status": "simulated",
  "patchValid": true,
  "fileChanged": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

Zusätzlich enthält es die geprüften fachlichen Werte sowie `changePlan` und
`patchPreview`.

---

## Sicherheitsgrenze

Der Export enthält:

- keine Credentials,
- keinen GitHub-Knoten,
- keinen HTTP-Knoten,
- keinen Datei-Schreibknoten,
- keine Git-Befehle,
- keine Branch-, Commit-, Push- oder Pull-Request-Funktion.

Alle Verarbeitung findet lokal in den beiden Code-Knoten statt. `sourceSha`
wird nur als nichtleerer String geprüft und nicht gegen GitHub verglichen.

---

## Versionsabgrenzung

Dieses Dokument beschreibt ausschließlich den exportierten Stand v0.1.0. Die
Datei `FLOW_v0.2.0.md` gehört zu einem noch nicht implementierten Entwurf und
beschreibt keinen vorhandenen n8n-Workflow.
