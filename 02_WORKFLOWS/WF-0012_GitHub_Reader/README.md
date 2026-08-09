# WF-0012 – GitHub Reader

## Überblick

| Feld | Wert |
|---|---|
| Workflow-ID | `WF-0012` |
| Name | GitHub Reader |
| Typ | Reader |
| Version | `v0.1.1` |
| Status | `released` |
| Plattform | n8n |
| Betriebsmodus | `read-only` |
| Verantwortlich | JaMoKo |
| Erstellt | 2026-08-05 |

---

## Versionsabgrenzung

Der veröffentlichte WF-0012-Vertrag und der veröffentlichte Workflow-Export
bleiben `v0.1.1/released`.

Der gesonderte Dokumentsatz für `v0.2.0` plant ausschließlich den lokalen
Operational Read-only Run `ORT-001`:

- `SPECIFICATION_v0.2.0.md`
- `ARCHITECTURE_v0.2.0.md`
- `FLOW_v0.2.0.md`
- `TESTS_v0.2.0.md`

Alle vier Dokumente haben den Status `draft/not-started`. Es existieren weder
eine v0.2.0-Implementierung noch ein v0.2.0-Workflow-Export oder ein
ausgeführter `ORT-001`. Die Nachweise `LRT-001` bis `LRT-003` bleiben
eigenständige lokale Testnachweise und sind keine operativen Läufe.

---

## 1. Zweck

WF-0012 liest genau eine definierte Datei aus einem freigegebenen GitHub-Repository.

Der Workflow übernimmt:

- die Validierung der Eingabedaten,
- die Prüfung des erlaubten Owners,
- die Prüfung des erlaubten Repositorys,
- die Prüfung der konkreten Owner-Repository-Kombination,
- die Validierung des Dateipfads,
- den lesenden Abruf der Datei über GitHub,
- die Dekodierung des Dateiinhalts,
- die Übernahme des aktuellen Datei-SHAs,
- die Normalisierung erfolgreicher Ergebnisse,
- die kontrollierte Normalisierung aller Fehler,
- die abschließende Anwendung des Schreibschutzes.

WF-0012 führt keine Änderungen an GitHub aus.

---

## 2. Einsatzgebiet

Der Workflow ist für textbasierte Dateien innerhalb freigegebener JaMoKo-Repositorys vorgesehen.

Typische Anwendungsfälle:

- Registry-Dateien lesen,
- Workflow-Dokumentationen abrufen,
- Konfigurationsstände prüfen,
- aktuelle Datei-SHAs ermitteln,
- Inhalte für kontrollierte Folgeprozesse bereitstellen,
- Repository-Daten für Audits und Vergleiche lesen.

WF-0012 ist kein allgemeiner GitHub-Browser und kein Schreibworkflow.

---

## 3. Freigegebenes Ziel

Für `v0.1.0` ist folgende Kombination freigegeben:

```text
Owner: Jamoko112026
Repository: JaMoKo_Automation_OS
