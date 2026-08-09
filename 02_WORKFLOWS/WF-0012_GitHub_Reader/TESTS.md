# WF-0012 – GitHub Reader

## Tests

Version: `v0.1.1`
Status: `testing`
Typ: `Reader`

Die Abschnitte 1 bis 6 dokumentieren die abgeschlossenen Tests von `v0.1.0`.
Abschnitt 7 dokumentiert den gesonderten Konformitätsnachweis des neuen
`v0.1.1`-Exports. Abschnitt 8 dokumentiert einen davon getrennten, strikt
lokalen Read-only-Test des aktuell geöffneten Repositorys.

---

## 1. Testziel

Die Tests weisen nach, dass WF-0012:

- genau eine definierte GitHub-Datei lesen kann,
- Eingaben vollständig validiert,
- ausschließlich freigegebene Repositorys verarbeitet,
- Dateiinhalt korrekt dekodiert,
- Datei-SHA und Metadaten korrekt übernimmt,
- Fehler kontrolliert normalisiert,
- keinerlei Schreiboperationen ausführt,
- sämtliche Schreibschutzwerte immer auf `false` setzt.

---

## 2. Testumgebung

| Bestandteil | Wert |
|---|---|
| Workflow | WF-0012 – GitHub Reader |
| Version | `v0.1.0` |
| Plattform | n8n |
| GitHub Owner | `Jamoko112026` |
| Repository | `JaMoKo_Automation_OS` |
| Referenz | `main` |
| Testdatei | `01_REGISTRY/workflow_registry.md` |
| Betriebsmodus | `read-only` |

GitHub-Zugangsdaten werden ausschließlich über die n8n-Credential-Verwaltung eingebunden.

Token und andere Geheimnisse dürfen weder in Testdaten noch in Screenshots oder Exporten erscheinen.

---

## 3. Standard-Testeingabe

```json
{
  "owner": "Jamoko112026",
  "repository": "JaMoKo_Automation_OS",
  "path": "01_REGISTRY/workflow_registry.md",
  "ref": "main"
}
```

---

## 4. Teststatus

| Test-ID | Testfall | Erwartung | Status |
|---|---|---|---|
| `T-001` | Gültige Datei lesen | Dateiinhalt, SHA und Metadaten werden übernommen | `passed` |
| `T-002` | Pfad mit `../` | Ablehnung mit `PATH_INVALID` | `passed` |
| `T-003` | Nicht erlaubter Owner | Ablehnung mit `OWNER_NOT_ALLOWED` | `passed` |
| `T-004` | Nicht erlaubtes Repository | Ablehnung mit `REPOSITORY_NOT_ALLOWED` | `passed` |
| `T-005` | Schreibschutz nach erfolgreichem Lesen | Alle Schreibschutzwerte sind `false` | `passed` |
| `T-006` | Schreibschutz nach Ablehnung | Alle Schreibschutzwerte sind `false` | `passed` |
| `T-007` | Fehlendes Pflichtfeld | Ablehnung mit `INPUT_INVALID` | `passed` |
| `T-008` | Leere Referenz | Ablehnung mit `REF_MISSING` | `passed` |
| `T-009` | Nicht vorhandene Datei | Ablehnung mit `FILE_NOT_FOUND` | `passed` |
| `T-010` | Ungültige GitHub-Antwort | Ablehnung mit `READ_VALIDATION_FAILED` | `passed` |
| `T-011` | Fehlerhafte Base64-Daten | Ablehnung mit `CONTENT_DECODE_FAILED` | `passed` |
| `T-012` | GitHub-Authentifizierungsfehler | Ablehnung mit `AUTHENTICATION_FAILED` | `passed` |
| `T-013` | GitHub-Zugriff verweigert | Ablehnung mit `ACCESS_DENIED` | `passed` |
| `T-014` | Sonstiger GitHub-API-Fehler | Ablehnung mit `GITHUB_API_ERROR` | `passed` |

---

## 5. Testergebnis

Alle definierten Tests `T-001` bis `T-014` wurden erfolgreich durchgeführt.

Bestätigt wurden:

- erfolgreicher und kontrollierter GitHub-Lesezugriff,
- vollständige Eingabe- und Pfadvalidierung,
- Beschränkung auf freigegebene Repositorys,
- korrekte Base64-Prüfung und Dekodierung,
- kontrollierte Normalisierung aller vorgesehenen Fehlerfälle,
- zuverlässiger Schreibschutz auf Erfolgs- und Fehlerpfaden,
- keine Schreib-, Commit- oder Push-Operationen.

## 6. Testfreigabe

Testergebnis: `passed`
Getestete Version: `v0.1.0`
Betriebsmodus: `read-only`

WF-0012 ist technisch bereit für die Freigabe.

---

## 7. Export-Konformität v0.1.1

Geprüfter Export:

```text
exports/WF-0012_GitHub_Reader_v0.1.1.json
```

Die Statuswerte unterscheiden echte n8n-Laufzeittests von lokalen Code- und
Strukturprüfungen. `passed-runtime` bestätigt eine tatsächliche Ausführung in
n8n; `passed-local` bestätigt ausschließlich die lokale Exportprüfung.

| Test-ID | Testfall | Erwartung | Status |
|---|---|---|---|
| `T-015` | Gültiger SHA | Exakt 40 Hex-Zeichen führen bei sonst gültiger Antwort zu `status = read` | `passed-local` |
| `T-016` | Leerer oder ungültiger SHA | Ablehnung mit `READ_VALIDATION_FAILED` | `passed-local` |
| `T-017` | Ungültige UTF-8-Bytefolge | Verlustfreie Rundreise scheitert; Ablehnung mit `CONTENT_DECODE_FAILED` | `passed-local` |
| `T-018` | Gültig kodiertes U+FFFD | Erfolg; Zeichen bleibt unverändert erhalten | `passed-local` |
| `T-019` | UTF-8-BOM | Erfolg; BOM bleibt unverändert erhalten | `passed-local` |
| `T-020` | LF und CRLF | Erfolg; jeweilige Zeilenenden bleiben exakt erhalten | `passed-local` |
| `T-021` | Leerer Dateiinhalt | Erfolg mit `file.content = ""` | `passed-local` |
| `T-022` | Rand-Leerzeichen in Zielwerten | `owner`, `repository`, `path` und `ref` werden abgelehnt und niemals getrimmt | `passed-local` |
| `T-023` | Minimale Erfolgsausgabe | Nur `status`, `mode`, `source`, `file` und die vier Schreibschutzwerte; `file` enthält nur `path`, `sha`, `encoding`, `content` | `passed-local` |
| `T-024` | Alle kontrollierten Fehlercodes | Fehlerausgabe enthält nur `status`, `mode`, `error` und die vier Schreibschutzwerte | `passed-local` |
| `T-025` | Unerwarteter Laufzeitfehler im finalen Guard | Bereinigte Ablehnung mit `GITHUB_API_ERROR`, ohne Ziel- oder Inhaltsdaten | `passed-local` |
| `T-026` | Kennungen und Schreibfunktionen | Keine `request_id`, `correlation_id` oder schreibende HTTP-/Writer-Funktion | `passed-local` |
| `T-027` | Null Eingangsobjekte am Kardinalitäts-Guard | Kein GitHub-Aufruf; genau eine minimale Fehlerausgabe mit `INPUT_INVALID` | `passed-local` |
| `T-028` | Genau ein Eingangsobjekt | Genau ein Guard-Ausgang; höchstens ein GitHub-Aufruf und genau ein finales Vertragsobjekt | `passed-local` |
| `T-029` | Zwei Eingangsobjekte | Kein GitHub-Aufruf; genau eine minimale Fehlerausgabe mit `INPUT_INVALID` | `passed-local` |
| `T-030` | Kontrollfluss nach Kardinalitätsprüfung | Ablehnungen führen ausschließlich zum finalen Vertrags-Guard; ein gültiges Einzelobjekt führt ausschließlich zur Eingangsvalidierung | `passed-local` |

Die lokalen Tests von `T-027` bis `T-030` führen den Kardinalitäts-Code mit null,
einem und zwei Items aus und prüfen Ausgangsanzahl, Fehlerhülle und den jeweils
erreichbaren Kontrollfluss. Sie führen keinen echten GitHub-Aufruf aus.

### 7.1 Runtime-Exportvergleich vom 2026-08-09

Verglichener Runtime-Export:

```text
exports/WF-0012_GitHub_Reader_v0.1.1-runtime-test-2_2026-08-09.json
```

Ergebnis: `passed-runtime-export`

Der Runtime-Export entspricht dem veröffentlichten `v0.1.1`-Export fachlich
vollständig. Nachgewiesen wurden:

- exakt 12 Knoten einschließlich Execute Workflow Trigger,
- identischer Cardinality Guard und Cardinality Router,
- identische Logik der Knoten 07 und 08,
- ausschließlich lesender GitHub-Zugriff per HTTP GET,
- vorhandener Fehlerpfad über `continueRegularOutput`,
- `file.encoding = "base64"` in der vertraglichen Erfolgsausgabe,
- alle vier Schreibschutzwerte immer `false`,
- keine Writer-, Commit- oder Push-Funktion.

Der Vergleich bestätigt die Exportstruktur und die fachliche Workflow-Logik.
Separate echte Negativ- und Fehlerpfadtests in der n8n-Laufzeit bleiben als
eigene Testfälle offen.

### 7.2 Echte n8n-Runtime-Tests vom 2026-08-09

| Test-ID | Testfall | Ergebnis | Status |
|---|---|---|---|
| `RT-001` | Unvollständiger Input ohne `path` und `ref` | `status = rejected`, `error.code = INPUT_INVALID` | `passed-runtime` |
| `RT-002` | Nicht vorhandene Datei | `status = rejected`, `error.code = FILE_NOT_FOUND` | `passed-runtime` |
| `RT-003` | Erfolgreicher realer GitHub-Lesevorgang | `status = read`, `mode = read-only`, `file.encoding = base64` | `passed-runtime` |
| `RT-004` | Nicht erlaubtes Repository | `status = rejected`, `error.code = REPOSITORY_NOT_ALLOWED`; vor dem GitHub-Aufruf blockiert | `passed-runtime` |
| `RT-005` | Ungültiger Branch beziehungsweise `ref` | `status = rejected`, `error.code = FILE_NOT_FOUND` | `passed-runtime` |
| `RT-006` | Gültigen Standardzustand nach den Tests wiederhergestellt | Erfolgreicher realer GitHub-Lesevorgang mit `status = read` und `file.encoding = base64` | `passed-runtime` |

Die Fälle `RT-001` bis `RT-006` wurden am `2026-08-09` tatsächlich in n8n
ausgeführt und bestanden. Für jeden Fall wurden genau ein kontrollierter Output
und folgende Schreibschutzwerte nachgewiesen:

- `writeRequested = false`
- `writeExecuted = false`
- `commitCreated = false`
- `pushExecuted = false`

`RT-004` wurde vor dem GitHub-Aufruf blockiert. `RT-003` und `RT-006` waren
erfolgreiche echte GitHub-Lesevorgänge. Screenshot-Nachweise wurden gesichert.

### 7.3 Ausstehender Laufzeitnachweis

Noch offen sind:

- Ausführung aller lokalen Fälle `T-015` bis `T-030` im n8n-Laufzeitkontext,
- Nachweis, dass `alwaysOutputData` am Execute-Workflow-Trigger bei null vom
  aufrufenden Workflow gelieferten Items tatsächlich genau einen leeren
  Platzhalter erzeugt und dadurch den Kardinalitäts-Guard ausführt,
- Nachweis des `continueRegularOutput`-Fehlerpfads bei einem tatsächlichen
  HTTP-Transportfehler,
- bereinigte Archivierung der Laufzeitnachweise.

Die Null-Item-Behandlung hängt technisch davon ab, dass die eingesetzte
n8n-Version `alwaysOutputData` am Execute-Workflow-Trigger wie konfiguriert
ausführt. Ohne diesen Laufzeitnachweis ist nur lokal belegt, dass der Guard bei
null empfangenen Items beziehungsweise bei dem erwarteten leeren Platzhalter
genau eine Ablehnung erzeugt. Falls die Zielinstanz den Trigger bei null Items
ohne Folgeausführung beendet, muss der aufrufende Workflow als kleinste
Alternative stets genau ein Eingangsobjekt übergeben; andernfalls ist der
Ein-Objekt-Ausgabevertrag für diesen Fall technisch nicht erfüllbar.

Die sechs Runtime-Fälle `RT-001` bis `RT-006` sind nicht mehr offen. Die
ursprünglichen lokalen Fälle `T-015` bis `T-030` bleiben von diesen zusätzlichen
Runtime-Fällen unberührt. Bis die verbleibenden Punkte abgeschlossen sind,
besitzt `v0.1.1` einen lokal geprüften Export und sechs bestandene echte
Runtime-Fälle, aber noch keinen vollständigen operativen n8n-Konformitätsnachweis.

---

## 8. Lokaler Repository-Read-only-Test

### 8.1 LRT-001 vom 2026-08-09

`LRT-001` ist ein separater lokaler Nachweis und kein Lauf des n8n-Exports. Der
Test erweitert weder den veröffentlichten `v0.1.1`-Ausgabevertrag noch den
zulässigen GitHub-Reader-Betrieb.

| Test-ID | Testfall | Erwartung | Ergebnis | Status |
|---|---|---|---|---|
| `LRT-001` | Sanitisiertes Lesen des aktuell geöffneten lokalen Repositorys | Git-Erkennung, Branch, sauber/dirty-Status, drei letzte Commit-Kurz-Hashes mit Betreff sowie Metadaten explizit erlaubter WF-0012-Dateien; keine Remote-, Inhalts- oder Credential-Daten | Repository erkannt, Branch `main`, Working Tree am Lesezeitpunkt `clean`; alle acht erlaubten Dateien vorhanden | `passed-local-read-only` |

Explizit erlaubt waren ausschließlich:

- `README.md`
- `SPECIFICATION.md`
- `ARCHITECTURE.md`
- `FLOW.md`
- `TESTS.md`
- `CHANGELOG.md`
- `KNOWN_ISSUES.md`
- `exports/WF-0012_GitHub_Reader_v0.1.1.json`

Pro erlaubter Datei wurden nur repository-relativer Pfad, Existenz,
regulärer Dateityp, Byte-Größe und SHA-256 erfasst. Datei-Inhalte wurden nicht
in den Report übernommen. Vollständige oder sanitisierte Git-Remotes wurden
weder gelesen noch ausgegeben.

Während der Lesephase wurden keine Repository-Dateien geändert, kein n8n-Workflow
ausgeführt, kein GitHub-API- oder HTTP-Zugriff ausgelöst, keine Credentials
verwendet und keine Write-, Commit-, Push- oder Tag-Operation ausgeführt. Die
anschließende Erstellung dieses Nachweises ist von der geprüften Lesephase
getrennt.

Sanitisierter Report:

```text
exports/WF-0012_local-read-only-test-report_2026-08-09.json
```

Der Test ändert weder die technische Exportkonformität noch den Release-Status:
Der Dokumentvertrag und der veröffentlichte Workflow-Export bleiben `v0.1.1`.
