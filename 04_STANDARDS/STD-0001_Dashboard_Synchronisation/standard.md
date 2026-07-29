# STD-0001 – Dashboard Synchronisation

## Status

Active

## Zweck

Dieser Standard definiert, wie JaMoKo-Dashboard-Karten zwischen einer Soll-Definition und einem bestehenden Trello-Board abgeglichen werden.

Ziel ist eine nachvollziehbare, sichere und wiederholbare Synchronisation ohne unnötige Duplikate oder unbeabsichtigte Löschungen.

## Geltungsbereich

Dieser Standard gilt zunächst für:

- WF-0004 – Dashboard Sync
- JaMoKo-Trello-Dashboards
- automatisch verwaltete Dashboard-Karten

Eine spätere Erweiterung auf andere Systeme und Objekttypen ist vorgesehen.

## Grundprinzip

Die Synchronisation arbeitet zustandsorientiert.

Sie vergleicht:

- Soll-Zustand: definierte Dashboard-Karten
- Ist-Zustand: vorhandene Karten im Trello-Board

Aus dem Vergleich wird eine konkrete Aktion abgeleitet.

## Eindeutige Erkennung einer Karte

In Version 1.0 wird eine Karte anhand dieser Merkmale erkannt:

1. Kartenname
2. Ziel-Liste

Eine Karte gilt nur dann als vorhanden, wenn Kartenname und Liste übereinstimmen.

## Synchronisationsregeln

### Regel 1 – Karte fehlt

Wenn eine definierte Karte nicht im Zielboard vorhanden ist:

- Karte erstellen
- definierte Ziel-Liste verwenden

### Regel 2 – Karte existiert

Wenn Kartenname und Ziel-Liste übereinstimmen:

- keine neue Karte erstellen
- vorhandene Karte unverändert lassen

### Regel 3 – Gleicher Name, andere Liste

Wenn der Kartenname vorhanden ist, die Karte aber in einer anderen Liste liegt:

- keine automatische Verschiebung in Version 1.0
- Status als Abweichung kennzeichnen
- manuelle Prüfung vorsehen

### Regel 4 – Doppelte Karten

Wenn mehrere Karten mit gleichem Namen in derselben Liste existieren:

- keine weitere Karte erstellen
- Duplikat als Abweichung melden
- keine automatische Löschung

### Regel 5 – Zusätzliche Karten

Karten, die im Board vorhanden, aber nicht in der Soll-Definition enthalten sind:

- bleiben unverändert bestehen
- werden nicht gelöscht
- gelten nicht automatisch als Fehler

## Sicherheitsprinzipien

WF-0004 darf in Version 1.0:

- fehlende Karten erstellen
- vorhandene Karten erkennen
- Abweichungen melden

WF-0004 darf in Version 1.0 nicht:

- Karten löschen
- Karten verschieben
- Kartennamen überschreiben
- Beschreibungen überschreiben
- Duplikate automatisch entfernen

## Vergleichsschlüssel

Der Vergleichsschlüssel lautet:

```text
normalisierter Kartenname + Ziel-Listen-ID
