# WF-0008 – Tests

## Version

0.1.0

## Status

Draft

---

## Testziel

Die Tests prüfen, ob WF-0008 Objektakten zuverlässig findet, liest, validiert, normalisiert und als Operations-Objekte ausgibt.

---

## TEST-001 – Gültige Objektakte laden

### Voraussetzung

Eine gültige `object.md` enthält mindestens:

- `objectId`
- `objectType`
- `title`
- `status`

### Erwartetes Ergebnis

- Die Datei wird gefunden.
- Die Datei wird gelesen.
- Das Objekt wird als `loaded` ausgegeben.
- Alle Pflichtfelder sind vorhanden.

---

## TEST-002 – Fehlende objectId

### Voraussetzung

Eine Objektakte besitzt keine `objectId`.

### Erwartetes Ergebnis

- Das Objekt wird nicht als gültiges Operations-Objekt ausgegeben.
- Das Ergebnis lautet `error`.
- Der Dateipfad wird im Fehlerdetail ausgegeben.

---

## TEST-003 – Ungültiges ID-Format

### Voraussetzung

Eine Objektakte enthält eine formal ungültige ID.

Beispiel:

```text
CUSTOMER-1