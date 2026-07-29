# Workflow Contract

## Workflow

WF-0004 – Dashboard Sync

## Status

Released v1.0

## Ziel

Das JaMoKo-Trello-Mainboard mit einer definierten Dashboard-Konfiguration abgleichen und ausschließlich fehlende Karten erstellen.

## Eingaben

- Trello-Board-Kennung
- vorhandene Trello-Karten
- vorhandene Trello-Listen
- Dashboard-Solldefinition

## Ausgaben

Für jede definierte Karte wird ein Status erzeugt:

- `exists`
- `missing`
- `wrong_list`
- `duplicate`

Fehlende Karten werden mit der richtigen Ziel-Listen-ID an den Trello-Erstellungsnode weitergegeben.

## Erfolgsbedingungen

- vorhandene Karten werden erkannt
- fehlende Karten werden erkannt
- fehlende Karten werden erstellt
- vorhandene Karten werden nicht doppelt erstellt
- Duplikate und falsche Listen werden als Abweichung erkannt
- bei vollständigem Dashboard werden keine Schreibaktionen ausgeführt

## Einschränkungen

Version 1.0 verschiebt, archiviert oder löscht keine vorhandenen Karten automatisch.

## Zugehöriger Standard

STD-0001 – Dashboard Synchronisation
