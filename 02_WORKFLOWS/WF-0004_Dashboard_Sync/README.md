# WF-0004 – Dashboard Sync

## Status

Released v1.0

## Zweck

Synchronisiert definierte Dashboard-Karten mit dem JaMoKo-Trello-Mainboard.

Der Workflow vergleicht den definierten Soll-Zustand mit den vorhandenen Karten und erstellt ausschließlich fehlende Karten.

## Funktionen

- liest vorhandene Trello-Karten
- liest vorhandene Trello-Listen
- definiert die gewünschten Dashboard-Karten
- vergleicht Soll- und Ist-Zustand
- erkennt die Zustände:
  - `exists`
  - `missing`
  - `wrong_list`
  - `duplicate`
- filtert ausschließlich fehlende Karten
- ordnet fehlende Karten der richtigen Trello-Liste zu
- erstellt ausschließlich fehlende Karten
- erzeugt bei vollständigem Dashboard keine Duplikate

## Zielboard

- Name: JaMoKo_Mainboard
- Trello-Kennung: `xvUPGjpo`

## Aktuelle Dashboard-Definition

| Karte | Ziel-Liste |
|---|---|
| 🎯 Wochenziel | 📋 Geplant |
| 💳 Krankenversicherung August | 🔍 Klären |
| 📊 Dashboard fertigstellen | 🚀 In Arbeit |

## Voraussetzungen

- Trello-Credentials sind in n8n eingerichtet.
- Das Zielboard ist geöffnet und beschreibbar.
- Die benötigten Ziel-Listen existieren.
- STD-0001 – Dashboard Synchronisation wird eingehalten.

## Sicherheitsregeln

Version 1.0 darf:

- fehlende Karten erkennen
- vorhandene Karten erkennen
- Duplikate erkennen
- Karten in falschen Listen erkennen
- fehlende Karten erstellen

Version 1.0 darf nicht:

- Karten automatisch verschieben
- Karten automatisch archivieren
- Karten automatisch löschen
- vorhandene Karten überschreiben

## Letzter erfolgreicher Test

Datum: 2026-07-29

Ergebnis:

- alle drei definierten Karten wurden korrekt erkannt
- alle drei Karten erhielten den Status `exists`
- der Filter lieferte `Kept: 0`
- es wurden keine neuen Karten erstellt
- nach Archivierung einer Soll-Karte wurde diese erfolgreich neu erstellt

## Version

v1.0

## Zugehöriger Standard

- STD-0001 – Dashboard Synchronisation
