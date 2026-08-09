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
| `ORT-INP-002` | null/mehrere Eingaben; fehlendes, zusätzliches, leeres oder Nicht-String-Feld | `rejected / INPUT_INVALID` vor Zielauflösung | `not-started` |
| `ORT-INP-003` | absoluter oder relativer Pfad als zusätzliches Eingabefeld | `rejected / INPUT_INVALID`; Pfad wird nicht aufgelöst oder ausgegeben | `not-started` |
| `ORT-TGT-001` | einziger Alias löst auf das kanonische Prozessstart-Arbeitsverzeichnis auf | exakt dieses eine Ziel wird intern verwendet | `not-started` |
| `ORT-TGT-002` | formal gültiger, aber abweichender Alias oder Profilwert | `rejected / TARGET_NOT_ALLOWED` vor Repository-Lesung | `not-started` |
| `ORT-TGT-003` | Arbeitsverzeichnis ist nicht kanonisch oder enthält Symlink-Komponente | `rejected / TARGET_RESOLUTION_FAILED` | `not-started` |
| `ORT-TGT-004` | Git-Toplevel weicht von Root ab oder Auflösung ist null/mehrdeutig | `rejected / TARGET_RESOLUTION_FAILED` | `not-started` |
| `ORT-TGT-005` | Identitätsmarker fehlt oder SHA-256 weicht ab | `rejected / TARGET_NOT_ALLOWED` | `not-started` |

### 3.2 Repository-Zustände

| ID | Testfall | Erwartung | Status |
|---|---|---|---|
| `ORT-REP-001` | unterstütztes Repository, `clean` | `completed / accepted` | `not-started` |
| `ORT-REP-002` | unterstütztes Repository, `dirty` | `completed / reported-neutral`; keine Bereinigung | `not-started` |
| `ORT-REP-003` | `.git` fehlt | `rejected / NOT_A_GIT_REPOSITORY` | `not-started` |
| `ORT-REP-004` | Detached HEAD oder ungeborener Branch | `rejected / REPOSITORY_STATE_UNSUPPORTED` | `not-started` |
| `ORT-REP-005` | Bare-Repository, Submodule oder verknüpfter Worktree | `rejected / REPOSITORY_STATE_UNSUPPORTED` | `not-started` |
| `ORT-REP-006` | Zustand ändert sich während des Laufs | `rejected / STATE_CHANGED_DURING_RUN` ohne Teilmetadaten | `not-started` |
| `ORT-REP-007` | `.git` ist Datei, Symlink oder anderer Dateityp | `rejected / REPOSITORY_STATE_UNSUPPORTED` | `not-started` |
| `ORT-REP-008` | Branchbytes leer, zu lang oder enthalten NUL/CR/zusätzliches LF | `rejected / REPOSITORY_STATE_UNSUPPORTED` | `not-started` |
| `ORT-REP-009` | `.git/objects` fehlt, ist kein reales Verzeichnis oder ein alternativer Object Store ist konfiguriert | `rejected / REPOSITORY_STATE_UNSUPPORTED` | `not-started` |

### 3.3 Datei- und Metadatengrenze

| ID | Testfall | Erwartung | Status |
|---|---|---|---|
| `ORT-FIL-001` | alle allowlistfähigen Pflichtdateien regulär vorhanden | ausschließlich erlaubte Metadaten | `not-started` |
| `ORT-FIL-002` | Pfad nicht in der Allowlist | `rejected / FILE_SCOPE_INVALID` vor Metadatenaufnahme | `not-started` |
| `ORT-FIL-003` | Symlink in Datei/Pfadsegment oder Ziel außerhalb der Repository-Grenze | `rejected / FILE_SCOPE_INVALID` | `not-started` |
| `ORT-FIL-004` | andere Pflichtdatei als der Identitätsmarker fehlt oder ist unlesbar | `rejected / FILE_SCOPE_INVALID` ohne Teilmetadaten | `not-started` |
| `ORT-FIL-005` | lokale Commit-Historie | höchstens drei Commit-IDs mit exakt zwölf Hex-Zeichen; keine Texte, Autoren oder E-Mails | `not-started` |
| `ORT-FIL-006` | Dateimetadaten | nur relativer Allowlist-Pfad, Existenz, Typ, Größe und SHA-256 | `not-started` |
| `ORT-FIL-007` | Datei größer als 1048576 Byte oder kein regulärer Dateityp | `rejected / FILE_SCOPE_INVALID` | `not-started` |
| `ORT-FIL-008` | `completed`-Report | exakt acht Dateiobjekte in statischer Allowlist-Reihenfolge | `not-started` |

### 3.4 Sanitizer und Ergebnisgrenze

| ID | Testfall | Erwartung | Status |
|---|---|---|---|
| `ORT-SAN-001` | akzeptierter sauberer Zustand | exakt das freigegebene Erfolgsschema | `not-started` |
| `ORT-SAN-002` | neutral gemeldeter unsauberer Zustand | keine geänderten Pfade oder Inhalte im Report | `not-started` |
| `ORT-SAN-003` | Ablehnung | keine Repository-, Commit- oder Dateimetadaten | `not-started` |
| `ORT-SAN-004` | absolute Pfade, Rohfehler oder Umgebungsdaten intern vorhanden | vollständig blockiert | `not-started` |
| `ORT-SAN-005` | token-, secret- oder credentialartige freie Werte im Reportkandidaten | Kandidat verworfen; statisches `rejected / SAFE_FAILURE` | `not-started` |
| `ORT-SAN-006` | jeder Ergebnisweg | genau ein Endobjekt und vier Schreibschutzwerte `false` | `not-started` |
| `ORT-SAN-007` | Sanitizer- oder interner Ausgabefehler | genau ein statisches `rejected / SAFE_FAILURE` ohne dynamische Daten | `not-started` |
| `ORT-SAN-008` | `completed`-Kandidat | exakte Pflichtfelder, Typen, Kardinalitäten und Konstanten aus Schema 6.1 bis 6.4 | `not-started` |
| `ORT-SAN-009` | `rejected`-Kandidat | exakte zehn Pflichtfelder; kein `outcome`, Repository, Commit oder Dateiobjekt | `not-started` |
| `ORT-SAN-010` | unbekanntes oder zusätzliches Feld im internen Kandidaten | nicht ignorieren; statisches `rejected / SAFE_FAILURE` | `not-started` |
| `ORT-SAN-011` | Branch enthält sensitive oder tokenartige Zeichenfolge | Rohwert fehlt vollständig; nur deterministischer SHA-256 im Branchobjekt | `not-started` |
| `ORT-SAN-012` | gleiche geprüfte Branchbytes zweimal | identische 64-stellige lowercase SHA-256-Repräsentation | `not-started` |
| `ORT-SAN-013` | dynamischer oder normalisierter Pfad im Kandidaten | statisches `rejected / SAFE_FAILURE`; nur wortgleiche Allowlist-Konstanten zulässig | `not-started` |
| `ORT-SAN-014` | separat autorisierte JSON-Serialisierung | UTF-8 ohne BOM, zwei Leerzeichen, LF, ein finales LF, feste Feldreihenfolge | `not-started` |

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
| `ORT-DET-003` | identische Dateimetadaten | exakt gleiche Allowlist-Reihenfolge und Repräsentation | `not-started` |
| `ORT-DET-004` | identische lokale Commit-Historie | newest-first und identische Verkürzung auf zwölf Hex-Zeichen | `not-started` |
| `ORT-DET-005` | gleiche Ablehnungsbedingung | identisches statisches `rejected`-Objekt | `not-started` |

### 3.8 Read-only-Kommandovertrag

| ID | Testfall | Erwartung | Status |
|---|---|---|---|
| `ORT-CMD-001` | jeder Git-Aufruf | exakt der festgelegte gemeinsame Präfix und die geschlossene Kindprozessumgebung | `not-started` |
| `ORT-CMD-002` | statische Aufrufanalyse | ausschließlich `GIT-01` bis `GIT-08` mit exakt dokumentierten Argumenten | `not-started` |
| `ORT-CMD-003` | Argumente mit Shell-Metazeichen | direkte Argumentübergabe; keine Shell, Expansion, Pipe, Umleitung oder Ausführung | `not-started` |
| `ORT-CMD-004` | erwartete Exit-0-Ausgaben aller acht Git-Aufrufe | exakte Formatvalidierung gemäß Spezifikation | `not-started` |
| `ORT-CMD-005` | jeder dokumentierte Sonderexit und ein sonstiger Exit | exakt zugeordneter Reason; sonst `LOCAL_READ_FAILED` | `not-started` |
| `ORT-CMD-006` | Timeout, stdout- oder stderr-Limit überschritten | `rejected / LOCAL_READ_FAILED`; keine Rohdaten | `not-started` |
| `ORT-CMD-007` | statische Dateisystemaufrufanalyse | nur `getcwd`, `realpath`, `lstat`, read-only `open`, `fstat`, SHA-256-Lesestrom und `close` | `not-started` |
| `ORT-CMD-008` | Git-Aufrufe auf unverändertem Fixture | keine Locks, Index- oder Repository-Änderung; `GIT_OPTIONAL_LOCKS=0` wirksam | `not-started` |
| `ORT-CMD-009` | statische und isolierte Netzwerkprüfung | kein Remote-Argument, DNS, Socket, HTTP, SSH oder Credential-Zugriff erreichbar | `not-started` |
| `ORT-CMD-010` | `GIT-06`-Records und Mode `160000` | exakte Record-Grammatik; Submodule führt zu `REPOSITORY_STATE_UNSUPPORTED` | `not-started` |
| `ORT-CMD-011` | `GIT-07` leer beziehungsweise nicht leer | ausschließlich `clean` beziehungsweise `dirty`; keine Pfadzerlegung oder Ausgabe | `not-started` |
| `ORT-CMD-012` | Partial-Clone-/Promisor-Konfiguration bei gesperrtem Netzwerk | `--no-lazy-fetch` verhindert jeden impliziten Fetch; fehlende lokale Objekte führen bereinigt zu `LOCAL_READ_FAILED` | `not-started` |

### 3.9 Ablehnungsgrund-Matrix

| ID | Ausgelöste Bedingung | Exakter Reason | Status |
|---|---|---|---|
| `ORT-RSN-001` | ungültige Eingangsstruktur | `INPUT_INVALID` | `not-started` |
| `ORT-RSN-002` | nicht erlaubter Alias/Profilwert oder Marker | `TARGET_NOT_ALLOWED` | `not-started` |
| `ORT-RSN-003` | nicht eindeutige/kanonische Zielauflösung | `TARGET_RESOLUTION_FAILED` | `not-started` |
| `ORT-RSN-004` | `.git` fehlt | `NOT_A_GIT_REPOSITORY` | `not-started` |
| `ORT-RSN-005` | nicht unterstützter Repository-/Branchzustand | `REPOSITORY_STATE_UNSUPPORTED` | `not-started` |
| `ORT-RSN-006` | Datei- oder Pfadgrenze verletzt | `FILE_SCOPE_INVALID` | `not-started` |
| `ORT-RSN-007` | erlaubte lokale Leseoperation technisch fehlgeschlagen | `LOCAL_READ_FAILED` | `not-started` |
| `ORT-RSN-008` | Snapshotabweichung | `STATE_CHANGED_DURING_RUN` | `not-started` |
| `ORT-RSN-009` | unerwarteter interner/Sanitizer-/Schemafehler | `SAFE_FAILURE` | `not-started` |
| `ORT-RSN-010` | mehrere fachliche Fehler gleichzeitig | Reason des zuerst erreichten fehlgeschlagenen Gates | `not-started` |

### 3.10 Snapshot- und Stabilitätsvertrag

| ID | Testfall | Erwartung | Status |
|---|---|---|---|
| `ORT-STB-001` | Snapshotbildung | exakt alle neun geordneten Tupelbestandteile aus Spezifikation 9.5 | `not-started` |
| `ORT-STB-002` | Branch oder HEAD ändert sich zwischen Snapshots | `rejected / STATE_CHANGED_DURING_RUN` | `not-started` |
| `ORT-STB-003` | Index-, tracked- oder untracked-Status ändert sich | `rejected / STATE_CHANGED_DURING_RUN` | `not-started` |
| `ORT-STB-004` | Allowlist-Datei ändert Inode, Modus, Größe, Mtime oder SHA-256 | `rejected / STATE_CHANGED_DURING_RUN` | `not-started` |
| `ORT-STB-005` | alle Snapshotbestandteile bleiben bytegleich | Weiterleitung an Result Builder | `not-started` |

---

## 4. Mindestvoraussetzungen vor einem Testlauf

Vor dem ersten v0.2.0-Test müssen vorliegen:

1. nach erneutem Audit für die Implementierung freigegebener Dokumentvertrag,
2. buildbare Kandidatenimplementierung mit eindeutigem Versionsbezug, die erst
   durch diese Tests geprüft wird,
3. statisch prüfbare Alias-, Datei-, Kommando- und Schema-Allowlist im Kandidaten,
4. isolierter lokaler Test-Harness mit deaktiviertem Netzwerk und ohne
   Credential-Zugriff,
5. ausschließlich temporäre, reversible Fixtures außerhalb produktiver
   Repository-Dateien,
6. vorab definierter Abbruch-, Timeout- und Cleanup-Nachweis,
7. unveränderter, sauberer Ausgangszustand des Ziel-Repositories für alle Tests,
   die nicht ausdrücklich einen isolierten `dirty`-Fixturezustand prüfen.

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
Geplante Testfälle: 80
Ausgeführte v0.2.0-Testfälle: 0
ORT-001 ausgeführt: nein
```
