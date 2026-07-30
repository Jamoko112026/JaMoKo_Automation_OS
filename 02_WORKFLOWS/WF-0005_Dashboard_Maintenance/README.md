# WF-0005 – Dashboard Maintenance

## Status

Planned

## Version

v0.1

## Zweck

Prüft den Zustand des JaMoKo-Trello-Mainboards und erzeugt einen strukturierten Wartungsbericht.

Der Workflow arbeitet in der ersten Version ausschließlich lesend.

## Prüfbereiche

- benötigte Trello-Listen
- definierte Dashboard-Karten
- fehlende Karten
- Karten in falschen Listen
- Duplikate
- Gesamtzustand des Boards

## Abgrenzung

WF-0004 synchronisiert fehlende Dashboard-Karten.

WF-0005 prüft den vollständigen Zustand des Boards und berichtet über Abweichungen.

## Sicherheitsregel

Version 0.1 darf keine Karten:

- erstellen
- verschieben
- archivieren
- löschen
- verändern

## Geplanter Ablauf

1. Board-Listen einlesen
2. vorhandene Karten einlesen
3. Soll-Zustand definieren
4. Listen prüfen
5. Karten prüfen
6. Wartungsbericht erzeugen
7. Bericht ausgeben

## Zugehöriger Standard

- STD-0001 – Dashboard Synchronisation
