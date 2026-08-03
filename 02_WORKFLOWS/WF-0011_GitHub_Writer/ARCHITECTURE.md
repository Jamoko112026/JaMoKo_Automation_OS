# WF-0011 – Architecture

## Version

0.1.0

## Status

draft

---

## Architekturziel

WF-0011 bildet die kontrollierte Schreibstufe der Object Quality Pipeline des JaMoKo Automation OS.

Der Workflow übernimmt ausschließlich geprüfte und freigegebene Änderungsvorschläge, erzeugt daraus eine sichere Änderungsvorschau und bereitet die spätere Ausführung im GitHub-Repository vor.

Version 0.1.0 arbeitet ausschließlich im Betriebsmodus `simulation`.

Es werden keine Dateien verändert, keine Commits erstellt und keine Änderungen nach GitHub geschrieben.

---

## Position im System

```text
JaMoKo OS
      │
      ▼
WF-0008 – Object Loader
      │
      ▼
Normalisierte Objekte
      │
      ▼
WF-0009 – Object Repair Engine
      │
      ▼
Repair Proposals
      │
      ▼
WF-0010 – Object Auditor
      │
      ▼
Audit Report
      │
      ▼
Manual Approval
      │
      ▼
WF-0011 – GitHub Writer
      │
      ▼
Simulierter Schreibplan
```

In Version 0.1.0 endet die Pipeline bei einem simulierten Schreibplan.

Das GitHub-Repository ist das spätere Schreibziel, wird in dieser Version jedoch nicht verändert.

---

## Architekturprinzipien

WF-0011 folgt diesen Prinzipien:

- freigegebene Änderungen als einzige zulässige Eingabe,
- strikte Trennung von Prüfung und Ausführung,
- Simulation vor tatsächlichem Schreiben,
- keine impliziten Änderungen,
- nachvollziehbare Ein- und Ausgangsdaten,
- kontrollierter Abbruch bei ungültigen Daten,
- dokumentierte Sicherheitsentscheidungen,
- reproduzierbare Verarbeitung,
- minimale Schreibrechte in späteren Versionen.

---

## Betriebsmodus

Version 0.1.0 unterstützt ausschließlich:

```text
simulation
```

Der Betriebsmodus wird innerhalb des Workflows verbindlich geprüft.

Andere Werte werden mit `INVALID_MODE` abgewiesen.

Insbesondere sind folgende Modi in Version 0.1.0 nicht zulässig:

- `write`
- `commit`
- `push`
- `execute`
- `production`

---

## Eingangsarchitektur

WF-0011 verarbeitet genau einen freigegebenen Änderungsvorschlag pro Lauf.

```text
Approved Change Proposal
          │
          ▼
Input Normalization
          │
          ▼
Schema Validation
          │
          ▼
Safety Validation
```

Der Eingang enthält mindestens:

- `objectId`
- `path`
- `field`
- `currentValue`
- `approvedValue`
- `approvalStatus`
- `sourceSha`
- `approvedBy`
- `auditStatus`

Fehlende oder ungültige Pflichtfelder führen zu einem kontrollierten Abbruch.

---

## Verarbeitungskomponenten

Die Architektur von WF-0011 besteht aus sieben logischen Komponenten.

### 1. Input Receiver

Übernimmt genau einen Änderungsvorschlag und stellt ihn für die weitere Verarbeitung bereit.

Der Input Receiver führt keine fachliche Änderung durch.

### 2. Input Validator

Prüft:

- Anzahl der eingegangenen Änderungsvorschläge,
- Vorhandensein der Pflichtfelder,
- Gültigkeit der Objekt-ID,
- Gültigkeit des Zielfeldes,
- Grundstruktur der Eingangsdaten.

### 3. Approval Gate

Prüft, ob:

- `approvalStatus` den Wert `approved` besitzt,
- `approvedBy` dokumentiert ist,
- `auditStatus` den Wert `passed` besitzt.

Nicht freigegebene oder nicht bestandene Vorschläge dürfen die nächste Stufe nicht erreichen.

### 4. Safety Gate

Prüft:

- Betriebsmodus `simulation`,
- relativen und zulässigen Zielpfad,
- vorhandenen `sourceSha`,
- Ausschluss absoluter Pfade,
- Ausschluss von Pfadüberschreitungen,
- Ausschluss tatsächlicher Schreiboperationen.

### 5. Change Planner

Leitet aus den validierten Eingangsdaten die vorgesehene Änderung ab.

Der Change Planner beschreibt:

- betroffenes Objekt,
- betroffene Datei,
- betroffenes Feld,
- erwarteten Ausgangswert,
- freigegebenen Zielwert.

Er verändert keine Repository-Datei.

### 6. Patch Simulator

Erzeugt einen nicht angewendeten Patch beziehungsweise eine strukturierte Änderungsvorschau.

Der Patch ist ausschließlich ein Simulationsartefakt.

### 7. Result Builder

Erzeugt das abschließende Ergebnisobjekt.

Das Ergebnis dokumentiert:

- Workflow-ID,
- Version,
- Betriebsmodus,
- Verarbeitungsstatus,
- Änderungsvorschau,
- Patchstatus,
- Sicherheitsstatus,
- Fehlercode bei Ablehnung,
- unveränderten Repository-Zustand.

---

## Komponentenfluss

```text
Input Receiver
      │
      ▼
Input Validator
      │
      ▼
Approval Gate
      │
      ▼
Safety Gate
      │
      ▼
Change Planner
      │
      ▼
Patch Simulator
      │
      ▼
Result Builder
```

Jede Komponente darf die Verarbeitung abbrechen, wenn ihre Prüfbedingungen nicht erfüllt sind.

Ein Abbruch führt immer zu einem strukturierten Ergebnis mit dem Status `rejected`.

---

## Sicherheitsgrenzen

Die Sicherheitsgrenze von Version 0.1.0 liegt vor jedem tatsächlichen Schreibzugriff.

```text
Validierter Änderungsvorschlag
          │
          ▼
Änderungsvorschau
          │
          ▼
Simulierter Patch
          │
          ▼
SICHERHEITSGRENZE
          │
          X
Kein Dateizugriff mit Schreibwirkung
Kein Commit
Kein Push
```

Hinter der Sicherheitsgrenze befinden sich Funktionen, die erst in späteren Versionen eingeführt werden dürfen.

---

## Pfadsicherheit

Der angegebene Zielpfad wird als relativer Repository-Pfad behandelt.

Nicht zulässig sind insbesondere:

- absolute Pfade,
- Pfade mit `..`,
- Pfade außerhalb des freigegebenen Repository-Bereichs,
- leere Pfade,
- nicht erwartete Protokoll- oder URL-Angaben,
- Pfade zu geschützten Git- oder Systembereichen.

Die Pfadprüfung erzeugt keine Datei und prüft in Version 0.1.0 ausschließlich die Zulässigkeit des geplanten Ziels.

---

## SHA-Sicherheit

`sourceSha` dokumentiert den Ausgangsstand, auf dessen Grundlage Audit und Freigabe erfolgt sind.

Version 0.1.0 verwendet den SHA als Referenz im Simulationsergebnis.

Ein fehlender SHA führt zum Abbruch mit:

```text
SOURCE_SHA_MISSING
```

Ein tatsächlicher Vergleich mit dem aktuellen Repository-Stand und die Behandlung von SHA-Konflikten sind späteren Writer-Versionen vorbehalten.

---

## Datenfluss

```text
Approved Proposal
      │
      ▼
Validated Proposal
      │
      ▼
Approved and Audited Change
      │
      ▼
Safe Change Plan
      │
      ▼
Simulated Patch
      │
      ▼
Simulation Result
```

Die ursprünglichen Eingangsdaten bleiben während der Verarbeitung nachvollziehbar.

Jede abgeleitete Stufe ergänzt Prüf- und Statusinformationen, ohne die freigegebenen Werte stillschweigend zu verändern.

---

## Erfolgszustand

Eine erfolgreiche Simulation erzeugt:

```text
status: simulated
mode: simulation
patchValid: true
fileChanged: false
commitCreated: false
pushExecuted: false
```

Der Status `simulated` bedeutet ausschließlich, dass der geplante Änderungsvorgang erfolgreich geprüft und dargestellt wurde.

Er bedeutet nicht, dass die Änderung ausgeführt wurde.

---

## Fehlerarchitektur

Fehler werden kontrolliert behandelt und als strukturiertes Ergebnis ausgegeben.

```text
Prüfschritt
    │
    ├── gültig ──► nächster Prüfschritt
    │
    └── ungültig ──► Rejected Result
```

Ein Fehlerergebnis enthält mindestens:

- `workflowId`
- `version`
- `mode`
- `status`
- `errorCode`
- `message`
- `fileChanged`
- `commitCreated`
- `pushExecuted`

Auch bei Fehlern müssen die letzten drei Werte `false` bleiben.

---

## Fehlerklassen

| Fehlerklasse | Zuständige Komponente |
|---|---|
| Ungültige Eingangsanzahl | Input Validator |
| Fehlendes Pflichtfeld | Input Validator |
| Ungültige Objekt-ID | Input Validator |
| Ungültiges Feld | Input Validator |
| Fehlende Freigabe | Approval Gate |
| Nicht bestandenes Audit | Approval Gate |
| Ungültiger Betriebsmodus | Safety Gate |
| Ungültiger Zielpfad | Safety Gate |
| Fehlender `sourceSha` | Safety Gate |
| Ungültiger simulierter Patch | Patch Simulator |

---

## Technische Umsetzung in n8n

Die logischen Komponenten werden in n8n durch klar getrennte Nodes umgesetzt.

Vorgesehene Node-Gruppen:

1. Trigger und Input
2. Input-Normalisierung
3. Pflichtfeldprüfung
4. Approval- und Auditprüfung
5. Safety-Mode-Prüfung
6. Objekt- und Pfadprüfung
7. Erzeugung des Change Plans
8. Patch-Simulation
9. Ergebnisaufbau
10. Erfolgs- oder Fehlerausgabe

Die konkrete Node-Reihenfolge und die einzelnen Verzweigungen werden in `FLOW.md` beschrieben.

---

## Externe Schnittstellen

Version 0.1.0 benötigt keine schreibende externe Schnittstelle.

Insbesondere werden nicht verwendet:

- GitHub Contents API mit Schreibrechten,
- GitHub Commit API,
- GitHub Pull Request API,
- lokale Git-Schreibbefehle,
- Repository-Dateizugriffe mit Schreibwirkung.

Eine lesende oder schreibende GitHub-Anbindung ist nicht Bestandteil des produktiven Funktionsumfangs von Version 0.1.0.

---

## Vertrauensgrenzen

WF-0011 vertraut Eingangsdaten nicht automatisch.

Folgende Informationen müssen innerhalb des Workflows geprüft werden:

- Struktur des Änderungsvorschlags,
- dokumentierte Freigabe,
- bestandener Auditstatus,
- zulässiger Betriebsmodus,
- gültige Objekt-ID,
- zulässiger Zielpfad,
- vorhandener Ausgangs-SHA.

Erst nach erfolgreicher Prüfung darf ein simulierter Patch erzeugt werden.

---

## Abgrenzung der Verantwortlichkeiten

WF-0011 ist verantwortlich für:

- Validierung des freigegebenen Änderungsvorschlags,
- Durchsetzung des Sicherheitsmodus,
- Planung der vorgesehenen Änderung,
- Erzeugung einer Änderungsvorschau,
- Erzeugung eines simulierten Patches,
- strukturierte Erfolgs- und Fehlerausgaben.

WF-0011 ist in Version 0.1.0 nicht verantwortlich für:

- Erkennung von Qualitätsmängeln,
- Erzeugung fachlicher Reparaturvorschläge,
- Durchführung des Audits,
- Erteilung der manuellen Freigabe,
- tatsächliche Dateiveränderungen,
- Commit-Erstellung,
- Push-Ausführung,
- Pull-Request-Erstellung,
- Merge-Entscheidungen.

---

## Abhängigkeiten

Die Architektur basiert auf:

- DEC-0006 – Workflow als Objekttyp
- DEC-0007 – Workflow Pipeline Architecture
- DEC-0008 – Workflow ID Governance
- DEC-0009 – Writer Safety Modes
- STD-0003 – Workflow Architecture Standard v1.1.0
- WF-0009 – Object Repair Engine
- WF-0010 – Object Auditor

---

## Erweiterungspfad

Spätere Versionen können zusätzliche Sicherheitsmodi einführen.

Mögliche Entwicklungsstufen:

```text
v0.1.0
Simulation und Patch-Vorschau

v0.2.0
Lesender Abgleich mit Repository und sourceSha

v0.3.0
Kontrolliertes Schreiben in einen isolierten Branch

v0.4.0
Commit-Erstellung nach erneuter Freigabe

v1.0.0
Produktiver Writer mit vollständiger Governance
```

Jede Erweiterung benötigt:

- eine dokumentierte Sicherheitsentscheidung,
- angepasste Tests,
- aktualisierte Spezifikation,
- aktualisierte Architektur,
- nachvollziehbare Freigabe.

---

## Architekturentscheidung für Version 0.1.0

WF-0011 endet nach der Erzeugung eines validierten simulierten Patches.

Die Architektur enthält bewusst keine ausführende Schreibkomponente.

Damit wird die spätere GitHub-Schreiblogik vorbereitet, ohne das Repository während Entwicklung und Tests einem Änderungsrisiko auszusetzen.