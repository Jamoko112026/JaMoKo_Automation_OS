# WF-0006 – Test Specification

## Version

1.0.0

## Status

Stable

---

# Testziel

Die Tests prüfen, ob WF-0006 Operations-Objekte zuverlässig mit dem JaMoKo Operations Dashboard synchronisiert.

Dabei werden alle Kernfunktionen des Workflows überprüft:

- Laden der Operations-Objekte
- Laden der Trello-Karten
- Vergleich über `objectId`
- Erstellen neuer Karten
- Aktualisieren bestehender Karten
- Zusammenfassung des Synchronisationslaufs

---

# TEST-001 – Neue Karte erstellen

## Voraussetzung

- Ein Operations-Objekt besitzt eine gültige `objectId`.
- Im Trello-Board existiert keine Karte mit dieser `objectId`.

## Erwartetes Ergebnis

- Genau eine neue Trello-Karte wird erstellt.
- Die Karte wird der richtigen Liste zugeordnet.
- Die `objectId` ist im Titel oder in der Beschreibung vorhanden.

---

# TEST-002 – Doppelte Karte verhindern

## Voraussetzung

- Eine Trello-Karte mit derselben `objectId` existiert bereits.

## Erwartetes Ergebnis

- Es wird keine zweite Karte erstellt.
- Das Ergebnis lautet `none`.

---

# TEST-003 – Bestehende Karte aktualisieren

## Voraussetzung

- Eine Trello-Karte besitzt dieselbe `objectId`.
- Titel oder Beschreibung unterscheiden sich vom Operations-Objekt.

## Erwartetes Ergebnis

- Die Karte wird aktualisiert.
- Es wird keine neue Karte erstellt.

---

# TEST-004 – Keine Änderung erforderlich

## Voraussetzung

Operations-Objekt und Trello-Karte sind identisch.

## Erwartetes Ergebnis

- Keine Aktualisierung.
- Ergebnis: `none`.

---

# TEST-005 – Kartenliste ändern

## Voraussetzung

Der Status eines Operations-Objekts hat sich geändert.

## Erwartetes Ergebnis

- Die Karte wird in die korrekte Trello-Liste verschoben.

---

# TEST-006 – Fehlende objectId

## Voraussetzung

Ein Operations-Objekt besitzt keine gültige `objectId`.

## Erwartetes Ergebnis

- Der Workflow beendet die Verarbeitung mit einer Fehlermeldung.
- Es wird keine Trello-Karte erzeugt.

---

# TEST-007 – Doppelte objectId

## Voraussetzung

Mehrere Operations-Objekte besitzen dieselbe `objectId`.

## Erwartetes Ergebnis

- Der Workflow erkennt den Konflikt.
- Die Verarbeitung wird abgebrochen.

---

# TEST-008 – Mehrere Trello-Karten

## Voraussetzung

Mehrere Trello-Karten besitzen dieselbe `objectId`.

## Erwartetes Ergebnis

- Ergebnis: `error`
- Keine automatische Aktualisierung.

---

# TEST-009 – Wiederholter Durchlauf

## Voraussetzung

Der Workflow wird zweimal ohne Datenänderungen ausgeführt.

## Erwartetes Ergebnis

- Keine Duplikate.
- Keine unnötigen Aktualisierungen.

---

# TEST-010 – Summary

## Erwartetes Ergebnis

Die Summary enthält mindestens:

- Gesamtzahl der Operations-Objekte
- Anzahl erstellter Karten
- Anzahl aktualisierter Karten
- Anzahl unveränderter Objekte
- Anzahl Fehler
- Workflow-ID
- Workflow-Version

---

# Abnahmekriterien

Version **1.0.0** gilt als freigegeben, wenn:

- alle Testfälle erfolgreich durchlaufen wurden,
- keine doppelten Trello-Karten entstehen,
- Wiederholungen reproduzierbar sind,
- alle Fehler eindeutig protokolliert werden,
- Create-, Update- und Summary-Prozess erfolgreich arbeiten.

---

# Letzte Prüfung

**Datum:** 2026-08-01

**Ergebnis:** Alle Kernfunktionen erfolgreich getestet.

**Status:** Release 1.0 freigegeben.