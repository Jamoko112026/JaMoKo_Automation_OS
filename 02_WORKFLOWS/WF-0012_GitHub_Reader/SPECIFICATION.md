# WF-0012 – GitHub Reader

## Specification

Version: `v0.1.0`
Status: `released`
Typ: `Reader`

---

## 1. Zweck

WF-0012 liest eine definierte Datei aus einem freigegebenen GitHub-Repository.

Der Workflow stellt den aktuellen Dateiinhalt sowie den zugehörigen GitHub-SHA für nachfolgende Prüf- und Simulationsprozesse bereit.

WF-0012 bildet damit die lesende Grundlage der Repository Automation.

---

## 2. Ziel

Der Workflow soll:

1. eine eindeutig definierte Repository-Datei abrufen,
2. den aktuellen Dateiinhalt bereitstellen,
3. den aktuellen Datei-SHA ermitteln,
4. technische Metadaten normalisieren,
5. Fehler kontrolliert und nachvollziehbar zurückgeben,
6. keinerlei Änderungen am Repository ausführen.

---

## 3. Einsatzbereich

WF-0012 wird eingesetzt, wenn ein nachfolgender Workflow den tatsächlichen Stand einer Repository-Datei benötigt.

Typische Verbraucher sind:

- WF-0010 – Object Auditor
- WF-0011 – GitHub Writer
- zukünftige Vergleichs-, Prüf- und Synchronisationsworkflows

---

## 4. Abgrenzung

WF-0012:

- erstellt keine Dateien,
- verändert keine Dateien,
- löscht keine Dateien,
- erzeugt keine Commits,
- führt keinen Push aus,
- erstellt keine Branches,
- führt keinen Merge aus,
- schreibt keine Daten nach GitHub zurück.

Der Workflow besitzt ausschließlich lesenden Zugriff.

---

## 5. Eingabe

Die Eingabe erfolgt als JSON-Objekt.

### Pflichtfelder

```json
{
  "owner": "Jamoko112026",
  "repository": "JaMoKo_Automation_OS",
  "path": "01_REGISTRY/workflow_registry.md",
  "ref": "main"
}