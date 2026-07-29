# WF-0003 – Dashboard Initial Cards

## Status

Released v1.0

## Zweck

Erstellt automatisch die ersten Standardkarten für ein JaMoKo-Trello-Dashboard.

Der Workflow ergänzt ein bereits vorbereitetes Trello-Board um die wichtigsten operativen Dashboard-Karten.

## Ablauf

1. Liest das ausgewählte Trello-Board.
2. Liest alle vorhandenen Listen des Boards.
3. Definiert die vorgesehenen Dashboard-Karten.
4. Ordnet jede Karte anhand des Listennamens der richtigen Trello-Liste zu.
5. Erstellt die Karten im Zielboard.

## Eingabe

- Trello-Board
- vorhandene JaMoKo-Standardlisten

## Ausgabe

Automatisch erzeugte Dashboard-Karten.

Aktuell enthalten:

- 🎯 Wochenziel
- 📊 Dashboard fertigstellen
- 💳 Krankenversicherung August

## Voraussetzungen

- WF-0002 wurde erfolgreich ausgeführt.
- Trello-Credentials sind in n8n eingerichtet.
- Die JaMoKo-Standardlisten existieren auf dem Zielboard.

## Verwendete Dienste

- n8n
- Trello

## Bekannte Einschränkungen

Version 1.0 erstellt die definierten Karten bei jedem Workflow-Lauf erneut.

Eine Prüfung auf bereits vorhandene Karten ist nicht enthalten.

Die Synchronisation und Vermeidung von Duplikaten wird in WF-0004 – Dashboard Sync umgesetzt.

## Version

v1.0

## Verantwortlichkeit

Erstbefüllung eines vorbereiteten JaMoKo-Trello-Dashboards.
