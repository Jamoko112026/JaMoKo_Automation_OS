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

## KI-007 – Keine allgemeine Sperre geschützter Git-Pfade

### Beschreibung

Der Export prüft die Endung `.md` und mehrere unsichere Pfadmuster, besitzt aber
keine ausdrückliche Sperre für `.git` oder andere geschützte
Repository-Bereiche.

### Auswirkung

Der dokumentierte Testwert `.git/config` wird wegen der fehlenden `.md`-Endung
abgelehnt. Ein Pfad wie `.git/config.md` wird durch die aktuelle Prüfung nicht
allein wegen des `.git`-Segments abgewiesen.

### Bewertung

Vor einer Wiederverwendung der Pfadprüfung oder jeder späteren Schreibversion
muss eine explizite, getestete Sperre geschützter Bereiche definiert und
implementiert werden.

---

## KI-008 – Kein zentraler Fehler-Sanitizer

### Beschreibung

Der Knoten `03 – Validate and Simulate` erzeugt strukturierte Ergebnisse für
erwartete Validierungsfehler. Ein zentraler sicherer Fehlerpfad für unerwartete
JavaScript-Laufzeitfehler ist nicht vorhanden.

### Auswirkung

Fehler außerhalb der ausdrücklich behandelten Prüfbedingungen sind nicht durch
den dokumentierten `rejected`-Vertrag abgesichert.

### Bewertung

Vor einer externen Anbindung muss ein minimaler, bereinigter Fehlerpfad
vorgesehen und getestet werden.

---

## KI-009 – Keine reproduzierbaren Runtime-Artefakte

### Beschreibung

`TESTS.md` dokumentiert 17 manuelle n8n-Tests als bestanden. Innerhalb der
v0.1.0-Dokumentation liegen dazu keine maschinenlesbaren Ausführungsprotokolle
vor.

### Auswirkung

Die historischen Ergebnisse sind aus den geprüften Dateien nicht unabhängig
reproduzierbar. Statisch prüfbar bleiben der Exportcode, die Knotenstruktur und
das Fehlen schreibender Komponenten.

### Bewertung

Bei einer erneuten Abnahme sollten Eingaben, Ausgaben und Laufkontext als
versionierbare Testartefakte gesichert werden.

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

Es sind keine unbeabsichtigten Schreibwirkungen bekannt. Der veröffentlichte,
deaktivierte Export enthält drei Knoten, keine Credentials und keine GitHub-,
HTTP-, Datei-, Commit- oder Push-Funktion.

Die 17 definierten Testfälle sind in `TESTS.md` als am 03.08.2026 bestanden
dokumentiert. Maschinenlesbare Runtime-Protokolle liegen innerhalb der
v0.1.0-Dateien nicht vor.

Die aufgeführten Einschränkungen entsprechen überwiegend dem bewusst begrenzten Funktionsumfang von WF-0011 v0.1.0.

Der n8n-Export liegt im vorgesehenen Exportordner. Aussagen zu visuellen
Nachweisen ersetzen keinen reproduzierbaren Runtime-Nachweis.
