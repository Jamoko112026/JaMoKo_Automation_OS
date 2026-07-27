# CON-0001 – Task Contract

## Zweck

Dieser Vertrag definiert die Daten, die WF-0001 verarbeitet.

## Version

0.1.0

## Eingabedaten

| Feld | Typ | Pflicht | Beschreibung |
|------|-----|----------|--------------|
| title | string | Ja | Titel der Aufgabe |
| description | string | Ja | Beschreibung der Aufgabe |
| priority | string | Nein | low, medium, high |
| project | string | Nein | Projekt-ID |

## Beispiel

```json
{
  "title": "Hero Reifendienst überarbeiten",
  "description": "Neues Hero-Bild erstellen",
  "priority": "high",
  "project": "PROJ-0003"
}
```

## Ausgabe

Der Workflow erstellt daraus eine Trello-Karte.