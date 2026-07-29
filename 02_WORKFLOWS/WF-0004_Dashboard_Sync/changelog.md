# Changelog

## v1.0 – 2026-07-29

- Workflow vollständig umgesetzt.
- Aktives JaMoKo_Mainboard als Zielboard eingebunden.
- Trello-Karten über die Trello REST API eingelesen.
- Trello-Listen über die Trello REST API eingelesen.
- Dashboard-Sollzustand definiert.
- Vergleichslogik für `exists`, `missing`, `wrong_list` und `duplicate` umgesetzt.
- Filter für fehlende Karten eingerichtet.
- Ziel-Listen-ID dynamisch ermittelt.
- Fehlende Karten werden automatisch erstellt.
- Duplikate werden nicht erneut erzeugt.
- Verhalten bei Karten in falschen Listen getestet.
- Duplikate manuell bereinigt.
- Abschlusstest erfolgreich:
  - drei Karten vorhanden
  - drei Karten mit Status `exists`
  - null Karten an den Erstellungszweig weitergegeben
- Status auf Released v1.0 gesetzt.
