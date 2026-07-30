# WF-0006 – Operations Synchronizer

## Version

0.1.0

## Status

development

---

# Ziel

Synchronisiert Arbeitsobjekte mit dem JaMoKo Operations Dashboard in Trello.

---

# Eingaben

## Datenquellen (v0.1.0)

- Interne Aufgabenliste (Testdaten)
- Trello Board

---

# Verarbeitung

1. Trello-Karten laden
2. Aufgabenquelle laden
3. Karten anhand der Objekt-ID vergleichen
4. Fehlende Karten erkennen
5. Ziel-Liste bestimmen
6. Neue Karten erstellen
7. Bestehende Karten aktualisieren
8. Synchronisationsbericht erzeugen

---

# Ausgaben

- Neue Trello-Karten
- Aktualisierte Trello-Karten
- Synchronisationsprotokoll

---

# Ziel-Listen

- 🎯 Fokus heute
- ▶️ In Arbeit
- 👀 Warten
- 👥 Kunden
- 📂 Projekte
- 🤖 Automation OS
- 🌐 Websites
- 📥 Inbox
- 💡 Ideen

---

# Regeln

- Keine doppelte Objekt-ID
- Jede Karte besitzt genau eine Objekt-ID
- Status bestimmt die Ziel-Liste
- Synchronisation ist beliebig oft wiederholbar
- Keine vorhandenen Karten überschreiben, wenn sich nichts geändert hat

---

# Fehlerfälle

- Trello nicht erreichbar
- Fehlende List-ID
- Ungültige Objekt-ID
- API-Fehler

---

# Abhängigkeiten

- Trello API
- n8n
- WF-0004
- WF-0005

---

# Zukunft

Geplante Datenquellen:

- JaMoKo OS
- CRM
- GitHub
- Persönliche Verwaltung
- Google Kalender
- Website
