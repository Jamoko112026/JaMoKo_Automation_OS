# WF-0012 – GitHub Reader

## Architecture

Version: `v0.1.0`
Status: `released`
Typ: `Reader`

---

## 1. Architekturziel

WF-0012 stellt einen kontrollierten, ausschließlich lesenden Zugriff auf eine definierte Datei in einem freigegebenen GitHub-Repository bereit.

Die Architektur trennt:

1. Eingabe,
2. Validierung,
3. Zugriffskontrolle,
4. GitHub-Lesezugriff,
5. Dekodierung,
6. Ergebnisvalidierung,
7. normalisierte Ausgabe.

Jede Anfrage muss entweder zu einem geprüften Leseergebnis oder zu einer kontrollierten Fehlerausgabe führen.

---

## 2. Architekturprinzipien

WF-0012 folgt diesen Prinzipien:

- Read-only by Design
- Validierung vor externem Zugriff
- Allowlist statt freier Repository-Auswahl
- eindeutige Referenz durch `ref`
- normalisierte Erfolgs- und Fehlerausgaben
- keine Zugangsdaten in Workflow-Daten
- keine impliziten Schreiboperationen
- nachvollziehbare Fehlercodes
- explizite Schreibschutzwerte

---

## 3. Systemgrenze

WF-0012 verarbeitet genau eine Datei pro Ausführung.

Innerhalb der Systemgrenze liegen:

- Entgegennahme der Eingabe,
- Prüfung der Pflichtfelder,
- Prüfung von Owner und Repository,
- Prüfung des Dateipfads,
- lesender GitHub-API-Aufruf,
- Dekodierung des Datei­inhalts,
- Übernahme von Datei-SHA und Metadaten,
- Validierung des Leseergebnisses,
- Erzeugung der normalisierten Ausgabe.

Außerhalb der Systemgrenze liegen:

- Änderung von Repository-Inhalten,
- Erstellung von Commits,
- Push- und Merge-Operationen,
- Branch-Verwaltung,
- Pull Requests,
- Speicherung gelesener Inhalte,
- Verarbeitung mehrerer Dateien,
- automatische Weitergabe an WF-0011.

---

## 4. Beteiligte Systeme

| System | Aufgabe |
|---|---|
| Aufrufender Workflow oder manueller Test | Liefert die definierte Leseanfrage |
| n8n | Führt Validierung, API-Aufruf und Normalisierung aus |
| n8n Credential Store | Verwaltet die GitHub-Zugangsdaten |
| GitHub API | Liefert Dateiinhalt, Datei-SHA und Metadaten |
| WF-0012 | Kontrolliert und normalisiert den gesamten Leseprozess |
| Nachfolgender Workflow | Kann das geprüfte Ergebnis später verwenden |

---

## 5. Komponenten

### 5.1 Input Receiver

Übernimmt das Eingabeobjekt mit:

- `owner`
- `repository`
- `path`
- `ref`

Der Input Receiver verändert die fachlichen Eingabewerte nicht.

---

### 5.2 Input Validator

Prüft:

- Vorhandensein aller Pflichtfelder,
- Datentypen,
- leere Werte,
- Vorhandensein von `ref`,
- grundlegende Struktur des Dateipfads.

Ungültige Eingaben werden vor dem GitHub-Zugriff abgelehnt.

---

### 5.3 Access Guard

Vergleicht:

- `owner` mit der freigegebenen Owner-Allowlist,
- `repository` mit der freigegebenen Repository-Allowlist.

Nicht freigegebene Ziele führen zu einer kontrollierten Ablehnung.

Der Access Guard verhindert, dass WF-0012 beliebige Repositorys auslesen kann.

---

### 5.4 Path Guard

Prüft den relativen Repository-Pfad.

Abgelehnt werden insbesondere:

- leere Pfade,
- absolute Pfade,
- Pfadüberschreitungen mit `../`,
- Pfade ohne eindeutig definiertes Ziel,
- technisch unzulässige Pfadwerte.

---

### 5.5 GitHub Read Adapter

Führt den ausschließlich lesenden GitHub-API-Aufruf aus.

Verwendete Parameter:

- Owner
- Repository
- Dateipfad
- Referenz

Der Adapter darf nur eine GitHub-Operation zum Abruf von Dateiinhalt und Dateimetadaten enthalten.

Schreibende GitHub-Operationen sind in dieser Komponente nicht zulässig.

---

### 5.6 Content Decoder

Dekodiert den von GitHub gelieferten Dateiinhalt.

Für v0.1.0 wird eine GitHub-Antwort mit Base64-kodiertem Dateiinhalt erwartet.

Schlägt die Dekodierung fehl, wird der Prozess mit folgendem Fehlercode beendet:

```text
CONTENT_DECODE_FAILED