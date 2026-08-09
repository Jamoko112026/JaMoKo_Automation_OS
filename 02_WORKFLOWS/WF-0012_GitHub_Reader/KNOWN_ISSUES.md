# WF-0012 – GitHub Reader

## Known Issues

Version: `v0.1.0`
Status: `released`
Typ: `Reader`

---

## 1. Zweck

Diese Datei dokumentiert bekannte Einschränkungen, offene technische Punkte und mögliche Risiken von WF-0012.

Ein Eintrag bedeutet nicht automatisch, dass der Workflow fehlerhaft ist. Er macht sichtbar:

- welche Grenzen bewusst bestehen,
- welche Punkte während der Umsetzung geprüft werden müssen,
- welche Probleme eine Veröffentlichung blockieren,
- welche Themen erst in späteren Versionen behandelt werden.

---

## 2. Statuswerte

| Status | Bedeutung |
|---|---|
| `open` | Problem oder Prüfpunkt ist noch offen |
| `in-review` | Ursache oder Lösung wird geprüft |
| `resolved` | Problem wurde nachweisbar behoben |
| `accepted` | Einschränkung ist bekannt und wird für diese Version akzeptiert |
| `deferred` | Bearbeitung wurde auf eine spätere Version verschoben |
| `not-reproducible` | Problem konnte nicht reproduziert werden |

---

## 3. Prioritätsstufen

| Priorität | Bedeutung |
|---|---|
| `critical` | Sicherheitsgrenze oder Read-only-Prinzip verletzt |
| `high` | Kernfunktion oder Veröffentlichung blockiert |
| `medium` | Funktion eingeschränkt, kontrollierter Betrieb aber möglich |
| `low` | Geringe Auswirkung oder rein dokumentarischer Punkt |

---
## 4. Veröffentlichungsregel

WF-0012 darf nicht veröffentlicht werden, solange mindestens ein Eintrag mit folgender Kombination besteht:

```text
status: open oder in-review
priority: critical oder high

```

Andere Einträge mit `medium` oder `low` dürfen veröffentlicht werden, wenn sie ausdrücklich als `accepted` oder `deferred` dokumentiert sind.

---

## 5. Aktueller Stand

Für `v0.1.0` bestehen keine bekannten offenen Probleme mit der Priorität `critical` oder `high`.

| ID | Beschreibung | Priorität | Status |
|---|---|---|---|
| – | Keine freigabeblockierenden Known Issues bekannt | – | `resolved` |

---

## 6. Akzeptierte Einschränkungen

- Das Verhalten bei sehr großen Dateien wurde in `v0.1.0` nicht umfassend untersucht.
- Weiterführende Validierungen unterschiedlicher Git-Referenzformate sind für spätere Versionen vorgesehen.
- Der Workflow liest ausschließlich einzelne Dateien und keine vollständigen Verzeichnisstrukturen.

Diese Einschränkungen verletzen weder die Kernfunktion noch das Read-only-Prinzip und blockieren die Freigabe nicht.

---

## 7. Freigabebewertung

Freigabeblockierende Known Issues: `0`
Sicherheitskritische Known Issues: `0`
Bewertung: `release-ready`

---

## 8. Planungsgrenzen für v0.2.0

Die folgenden Punkte sind offene Voraussetzungen des Entwurfs
`v0.2.0/draft/not-started`. Sie sind keine rückwirkenden Known Issues des
veröffentlichten Stands `v0.1.1/released` und ändern dessen Freigabestatus nicht.

- Es existiert noch kein lokaler `ORT-001`-Runner.
- Die statische Aliasauflösung sowie Datei- und Kommando-Allowlist sind noch
  nicht technisch umgesetzt oder unabhängig geprüft.
- Die endgültigen Ablehnungsgründe und das finale sanitisierte Report-Schema
  sind noch nicht freigegeben.
- Zentraler Output Sanitizer, Stability Guard und Single Output Boundary sind
  noch nicht implementiert oder getestet.
- Die Seiteneffektfreiheit bei unverändert vorhandenen `clean`- und `dirty`-
  Zuständen ist für einen operativen Runner noch nicht nachgewiesen.
- Die spätere Evidenzablage muss technisch und organisatorisch vom
  Read-only-Lauf getrennt bleiben; ein freigegebener Mechanismus existiert noch
  nicht.
- Die geplanten v0.2.0-Testfälle wurden noch nicht ausgeführt.

Solange einer dieser Punkte offen ist, darf `ORT-001` nicht ausgeführt und
`v0.2.0` nicht als implementiert, getestet oder veröffentlicht bezeichnet
werden.
