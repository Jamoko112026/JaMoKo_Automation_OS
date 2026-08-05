# WF-0011 – Known Issues

## Version

0.1.0

## Status

released

---

## Zweck

Dieses Dokument erfasst bekannte Einschränkungen und noch nicht umgesetzte Funktionen von WF-0011 – GitHub Writer.

---

## KI-001 – Reiner Simulationsbetrieb

### Beschreibung

WF-0011 v0.1.0 arbeitet ausschließlich im Modus `simulation`.

### Auswirkung

Der Workflow erzeugt einen Change Plan und eine Patch-Vorschau, verändert jedoch keine Repository-Datei.

### Bewertung

Beabsichtigte Sicherheitsbeschränkung.

---

## KI-002 – Kein Zugriff auf GitHub

### Beschreibung

Der Workflow liest keine Dateien und Metadaten direkt aus GitHub.

### Auswirkung

Der angegebene `sourceSha` kann noch nicht mit dem tatsächlichen Stand des Repositorys verglichen werden.

### Bewertung

Für Version 0.1.0 akzeptiert.

---

## KI-003 – Keine Prüfung des aktuellen Feldwertes

### Beschreibung

`currentValue` wird aus den Eingangsdaten übernommen, aber nicht mit dem tatsächlichen Dateiinhalt verglichen.

### Auswirkung

Ein zwischenzeitlich veränderter Objektstand kann in der Simulation nicht erkannt werden.

### Bewertung

Muss vor einer späteren Schreibfreigabe gelöst werden.

---

## KI-004 – Nur ein Änderungsvorschlag

### Beschreibung

Pro Ausführung darf genau ein Änderungsvorschlag verarbeitet werden.

### Auswirkung

Mehrere Änderungen müssen einzeln simuliert werden.

### Bewertung

Bewusste Begrenzung zur besseren Nachvollziehbarkeit.

---

## KI-005 – Keine tatsächliche Patch-Anwendung

### Beschreibung

Der Workflow erzeugt nur eine strukturierte Patch-Vorschau.

### Auswirkung

Die technische Anwendbarkeit auf eine reale Markdown-Datei wird noch nicht geprüft.

### Bewertung

Für die Simulationsversion akzeptiert.

---

## KI-006 – Keine Git-Operationen

### Beschreibung

Branch-, Commit-, Push- und Pull-Request-Funktionen sind nicht vorhanden.

### Auswirkung

Eine freigegebene Änderung kann nicht automatisch in das Repository übernommen werden.

### Bewertung

Beabsichtigte Sicherheitsbeschränkung.


---

## Voraussetzung für eine spätere Schreibversion

Vor der Entwicklung einer schreibenden Version müssen mindestens folgende Punkte gelöst und dokumentiert sein:

- sicherer lesender GitHub-Zugriff,
- Abgleich des `sourceSha`,
- Prüfung des tatsächlichen Feldwertes,
- Schutz vor parallelen Änderungen,
- sichere Patch-Anwendung,
- eingeschränkte Schreibrechte,
- Branch-Strategie,
- Commit- und Pull-Request-Verfahren,
- Wiederherstellungs- und Abbruchverfahren,
- zusätzliche Sicherheits- und Integrationstests,
- gesonderte dokumentierte Freigabe.

---

## Aktueller Gesamtstatus

Es sind keine unbeabsichtigten Schreibwirkungen bekannt.

Der Simulationsworkflow wurde in n8n aufgebaut und alle 17 definierten Testfälle wurden am 03.08.2026 erfolgreich ausgeführt.

Die aufgeführten Einschränkungen entsprechen überwiegend dem bewusst begrenzten Funktionsumfang von WF-0011 v0.1.0.

Der n8n-Export und die visuellen Nachweise wurden vollständig in den vorgesehenen Nachweisordnern abgelegt.