# WF-0006 – Tests

## Version

0.1.0

## Status

development

---

## Testziel

Die Tests prüfen, ob der Operations Synchronizer Arbeitsobjekte zuverlässig mit dem JaMoKo Operations Dashboard synchronisiert.

---

## TEST-001 – Fehlende Karte erstellen

### Voraussetzung

- Ein Arbeitsobjekt mit einer gültigen Objekt-ID existiert.
- Im Trello-Board gibt es noch keine Karte mit dieser Objekt-ID.

### Erwartetes Ergebnis

- Genau eine neue Karte wird erstellt.
- Die Karte liegt in der vorgesehenen Liste.
- Die Objekt-ID ist in der Kartenbeschreibung enthalten.

---

## TEST-002 – Doppelte Karte verhindern

### Voraussetzung

- Eine Karte mit derselben Objekt-ID existiert bereits.

### Erwartetes Ergebnis

- Es wird keine zweite Karte erstellt.
- Die vorhandene Karte bleibt bestehen.

---

## TEST-003 – Geänderte Karte aktualisieren

### Voraussetzung

- Eine Karte mit derselben Objekt-ID existiert.
- Titel, Beschreibung, Status oder Ziel-Liste wurden geändert.

### Erwartetes Ergebnis

- Die vorhandene Karte wird aktualisiert.
- Es wird keine neue Karte erstellt.

---

## TEST-004 – Unveränderte Karte ignorieren

### Voraussetzung

- Arbeitsobjekt und Trello-Karte enthalten dieselben relevanten Werte.

### Erwartetes Ergebnis

- Die Karte wird nicht aktualisiert.
- Das Ergebnis wird als `unchanged` gezählt.

---

## TEST-005 – Karte in richtige Liste verschieben

### Voraussetzung

- Der Status eines Arbeitsobjekts wurde geändert.
- Der neue Status gehört zu einer anderen Trello-Liste.

### Erwartetes Ergebnis

- Die vorhandene Karte wird in die richtige Liste verschoben.
- Die Objekt-ID bleibt unverändert.

---

## TEST-006 – Ungültige Objekt-ID

### Voraussetzung

- Ein Arbeitsobjekt besitzt keine oder eine ungültige Objekt-ID.

### Erwartetes Ergebnis

- Es wird keine Karte erstellt oder aktualisiert.
- Der Fehler erscheint im Synchronisationsbericht.

---

## TEST-007 – Wiederholter Durchlauf

### Voraussetzung

- Der Workflow wurde bereits einmal erfolgreich ausgeführt.
- Die Quelldaten wurden nicht verändert.

### Erwartetes Ergebnis

- Ein zweiter Durchlauf erzeugt keine Duplikate.
- Es erfolgen keine unnötigen Aktualisierungen.

---

## TEST-008 – Synchronisationsbericht

### Erwartetes Ergebnis

Der Workflow liefert mindestens:

- Anzahl erstellter Karten
- Anzahl aktualisierter Karten
- Anzahl unveränderter Karten
- Anzahl fehlerhafter Objekte
- Workflow-ID
- Workflow-Version

---

## Abnahmekriterium für Version 0.1.0

Version 0.1.0 gilt als funktionsfähig, wenn:

- TEST-001 bis TEST-008 erfolgreich durchgeführt wurden,
- keine doppelten Karten entstehen,
- die Wiederholbarkeit nachgewiesen ist,
- und alle Fehler nachvollziehbar protokolliert werden.