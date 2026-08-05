# WF-0012 – GitHub Reader

## Flow

Version: `v0.1.0`
Status: `released`
Typ: `Reader`

---

## 1. Zweck des Ablaufs

WF-0012 liest genau eine definierte Datei aus einem freigegebenen GitHub-Repository.

Der Ablauf stellt sicher, dass:

- die Eingabe vollständig geprüft wird,
- nur freigegebene Repositorys gelesen werden,
- ausschließlich lesende GitHub-Operationen stattfinden,
- Dateiinhalt und Datei-SHA korrekt übernommen werden,
- Erfolgs- und Fehlerausgaben normalisiert werden,
- sämtliche Schreibschutzwerte immer `false` bleiben.

---

## 2. Startbedingung

Der Workflow startet mit einem JSON-Objekt.

Erforderliche Felder:

```json
{
  "owner": "Jamoko112026",
  "repository": "JaMoKo_Automation_OS",
  "path": "01_REGISTRY/workflow_registry.md",
  "ref": "main"
}