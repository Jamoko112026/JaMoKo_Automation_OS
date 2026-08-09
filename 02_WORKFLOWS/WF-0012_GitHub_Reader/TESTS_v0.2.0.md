# WF-0012 – GitHub Reader

## Tests v0.2.0

Version: `v0.2.0`
Status: `draft`
Umsetzungsstatus: `not-started`
Testausführungen: `0`

---

## 1. Testgrenze

Diese Datei plant die Nachweise, die vor einem echten `ORT-001` bestanden sein
müssen. Sie dokumentiert keine ausgeführten v0.2.0-Tests.

`LRT-001`, `LRT-002` und `LRT-003` bleiben eigenständige, veröffentlichte
Testnachweise für begrenzte lokale Bedingungen:

| Nachweis | Belegter Zustand | Nicht belegt |
|---|---|---|
| `LRT-001` | sauberes lokales Git-Repository und begrenzte Metadatenaufnahme | implementierter ORT-Runner |
| `LRT-002` | unsauberer Working Tree wird neutral erkannt; Testzustand reversibel | stabiler operativer Lauf auf einem beliebigen `dirty` Zustand |
| `LRT-003` | leerer Nicht-Git-Ordner wird lokal erkannt und abgelehnt | finales ORT-Ablehnungsschema |

Keiner dieser Nachweise darf als bereits ausgeführter `ORT-001` umgedeutet
werden.

---

## 2. Statuswerte

| Status | Bedeutung |
|---|---|
| `not-started` | Test wurde nicht ausgeführt |
| `blocked` | notwendige Implementierung oder Vorbedingung fehlt |
| `passed-local` | lokale Ausführung nachvollziehbar bestanden |
| `failed` | Erwartung nicht erfüllt |

Für alle folgenden Fälle gilt aktuell `not-started`.

---

## 3. Geplante Testfälle

### 3.1 Eingangs- und Zielgrenze

| ID | Testfall | Erwartung | Status |
|---|---|---|---|
| `ORT-INP-001` | exakt freigegebener Alias und Profilwert | Eingabe erreicht den Trusted Target Resolver | `not-started` |
| `ORT-INP-002` | fehlendes, zusätzliches oder abweichendes Feld | minimale Ablehnung vor lokaler Repository-Lesung | `not-started` |
| `ORT-INP-003` | absoluter oder frei gewählter Pfad als Eingang | Ablehnung; Pfad wird nicht aufgelöst oder ausgegeben | `not-started` |
| `ORT-TGT-001` | Alias löst statisch auf genau ein freigegebenes Ziel auf | exakt dieses Ziel wird verwendet | `not-started` |
| `ORT-TGT-002` | Alias unbekannt oder Auflösung mehrdeutig | Ablehnung vor Repository-Lesung | `not-started` |

### 3.2 Repository-Zustände

| ID | Testfall | Erwartung | Status |
|---|---|---|---|
| `ORT-REP-001` | unterstütztes Repository, `clean` | `completed / accepted` | `not-started` |
| `ORT-REP-002` | unterstütztes Repository, `dirty` | `completed / reported-neutral`; keine Bereinigung | `not-started` |
| `ORT-REP-003` | leerer Nicht-Git-Ordner | minimale `rejected`-Ausgabe | `not-started` |
| `ORT-REP-004` | Detached HEAD oder ungeborener Branch | kontrollierte Ablehnung | `not-started` |
| `ORT-REP-005` | Bare-Repository, Submodule oder verknüpfter Worktree | bis zur eigenen Freigabe kontrollierte Ablehnung | `not-started` |
| `ORT-REP-006` | Zustand ändert sich während des Laufs | Stabilitätsgate lehnt ohne Teilmetadaten ab | `not-started` |

### 3.3 Datei- und Metadatengrenze

| ID | Testfall | Erwartung | Status |
|---|---|---|---|
| `ORT-FIL-001` | alle allowlistfähigen Pflichtdateien regulär vorhanden | ausschließlich erlaubte Metadaten | `not-started` |
| `ORT-FIL-002` | Pfad nicht in der Allowlist | Ablehnung vor Metadatenaufnahme | `not-started` |
| `ORT-FIL-003` | Symlink oder Ziel außerhalb der Repository-Grenze | Ablehnung | `not-started` |
| `ORT-FIL-004` | Pflichtdatei fehlt oder ist unlesbar | Ablehnung ohne Teilmetadaten | `not-started` |
| `ORT-FIL-005` | lokale Commit-Historie | höchstens drei Commit-IDs mit exakt zwölf Hex-Zeichen; keine Texte, Autoren oder E-Mails | `not-started` |
| `ORT-FIL-006` | Dateimetadaten | nur relativer Allowlist-Pfad, Existenz, Typ, Größe und SHA-256 | `not-started` |

### 3.4 Sanitizer und Ergebnisgrenze

| ID | Testfall | Erwartung | Status |
|---|---|---|---|
| `ORT-SAN-001` | akzeptierter sauberer Zustand | exakt das freigegebene Erfolgsschema | `not-started` |
| `ORT-SAN-002` | neutral gemeldeter unsauberer Zustand | keine geänderten Pfade oder Inhalte im Report | `not-started` |
| `ORT-SAN-003` | Ablehnung | keine Repository-, Commit- oder Dateimetadaten | `not-started` |
| `ORT-SAN-004` | absolute Pfade, Rohfehler oder Umgebungsdaten intern vorhanden | vollständig blockiert | `not-started` |
| `ORT-SAN-005` | token-, secret- oder credentialartige Marker | keine Ausgabe; Lauf fail-closed | `not-started` |
| `ORT-SAN-006` | jeder Ergebnisweg | genau ein Endobjekt und vier Schreibschutzwerte `false` | `not-started` |
| `ORT-SAN-007` | Sanitizer- oder interner Ausgabefehler | genau eine statische minimale Ablehnung ohne dynamische Daten | `not-started` |

### 3.5 Seiteneffekt- und Verbotsnachweise

| ID | Testfall | Erwartung | Status |
|---|---|---|---|
| `ORT-SEF-001` | Vorher-/Nachher-Vergleich bei `clean` | Zustand bytegenau identisch | `not-started` |
| `ORT-SEF-002` | Vorher-/Nachher-Vergleich bei `dirty` | vorhandener Zustand bytegenau identisch | `not-started` |
| `ORT-SEF-003` | statische und dynamische Kommandoprüfung | keine mutierende Datei- oder Git-Operation | `not-started` |
| `ORT-SEF-004` | Netzwerk- und Remote-Prüfung | kein HTTP, GitHub API, Fetch, Pull oder sonstiger Remote-Zugriff | `not-started` |
| `ORT-SEF-005` | Integrationsprüfung | kein n8n-, Credential-, WF-0011- oder Writer-Aufruf | `not-started` |
| `ORT-SEF-006` | Laufabschluss | ORT-001 erzeugt und überschreibt keine Reportdatei | `not-started` |

### 3.6 Getrennte Evidenzablage

| ID | Testfall | Erwartung | Status |
|---|---|---|---|
| `ORT-EVD-001` | separat autorisierte Ablage eines validierten Objekts | neuer repository-relativer Pfad; kein Überschreiben | `not-started` |
| `ORT-EVD-002` | JSON-, Geheimnisindikator- und Pfadprüfung | alle Prüfungen sauber | `not-started` |
| `ORT-EVD-003` | keine Ablageautorisation | keine Datei wird erzeugt | `not-started` |

### 3.7 Wiederholbarkeit

| ID | Testfall | Erwartung | Status |
|---|---|---|---|
| `ORT-DET-001` | gleiche Eingabe und identischer Repository-Snapshot | feldgleiches Ergebnis mit stabiler Listenreihenfolge | `not-started` |
| `ORT-DET-002` | Ergebnisbildung | keine Zeitstempel, Zufallswerte oder automatisch erzeugten Laufkennungen | `not-started` |

---

## 4. Mindestvoraussetzungen vor einem Testlauf

Vor dem ersten v0.2.0-Test müssen vorliegen:

1. freigegebener Dokumentvertrag,
2. separat geprüfte lokale Runner-Implementierung,
3. statische Alias-, Datei- und Kommando-Allowlist,
4. implementierter Sanitizer, Stability Guard und Single Output Boundary,
5. isolierte Testumgebung ohne Netzwerk- und Credential-Zugriff,
6. reversibler Testaufbau außerhalb produktiver Repository-Dateien,
7. definierter Abbruch- und Cleanup-Nachweis.

Vorher darf kein Fall von `not-started` auf einen bestandenen Status gesetzt
werden.

---

## 5. Voraussetzungen vor ORT-001

`ORT-001` darf erst operativ ausgeführt werden, wenn:

- sämtliche Tests dieses Dokuments bestanden und belegt sind,
- keine offene kritische oder hohe Sicherheitslücke besteht,
- der Runner selbst unverändert und eindeutig versioniert ist,
- der Zielalias und das Leseprofil für den Lauf freigegeben sind,
- ein unabhängiger Audit die Verbots- und Sanitizer-Grenzen bestätigt,
- die getrennte Reportablage ausdrücklich geklärt ist.

---

## 6. Aktueller Teststatus

```text
Version: v0.2.0
Status: draft
Umsetzungsstatus: not-started
Geplante Testfälle: 35
Ausgeführte v0.2.0-Testfälle: 0
ORT-001 ausgeführt: nein
```
