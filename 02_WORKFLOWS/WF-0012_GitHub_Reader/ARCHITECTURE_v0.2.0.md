# WF-0012 – GitHub Reader

## Architecture v0.2.0

Version: `v0.2.0`
Status: `draft`
Umsetzungsstatus: `not-started`

---

## 1. Architekturgrenze

Diese Architektur plant ausschließlich den lokalen Operational Read-only Run
`ORT-001`. Sie beschreibt keine vorhandenen Komponenten. Der veröffentlichte
GitHub-Reader `v0.1.1/released` und sein n8n-Export bleiben unverändert.

`LRT-001` bis `LRT-003` sind lokale Testnachweise. Sie begründen die drei
Basiszustände `clean`, `dirty` und `kein Git-Repository`, ersetzen aber keinen
Architektur- oder Laufzeitnachweis für `ORT-001`.

---

## 2. Architekturprinzipien

- ausschließlich lokale Leseoperationen,
- statische Alias- und Datei-Allowlist,
- keine frei übergebenen Dateisystempfade,
- minimale Datenaufnahme,
- interne Vorher-/Nachher-Stabilitätsprüfung,
- zentraler Sanitizer,
- genau ein Endobjekt,
- deterministische Listen- und Feldbildung ohne Zeitstempel oder Zufallswerte,
- keine automatische Persistenz,
- geschlossene Grenze zu Netzwerk, n8n und WF-0011.

---

## 3. Geplante Komponenten

| Komponente | Aufgabe | Darf nicht |
|---|---|---|
| Input Gate | genau ein Objekt und die zwei festen Eingabewerte prüfen | Werte ergänzen oder normalisieren |
| Trusted Target Resolver | einzigen Alias auf das kanonisch geprüfte Prozessstart-Arbeitsverzeichnis auflösen | freie, relative, mehrdeutige oder ausgegebene absolute Pfade akzeptieren |
| Repository Guard | reales `.git` samt lokalem, nicht alternativem Object Store, Git-Toplevel und unterstützten Repository-/Branchzustand prüfen | Remote-Kommandos ausführen |
| Identity Marker Gate | festen v0.1.1-Markerpfad und dessen festgelegten SHA-256 prüfen | Markerabweichung als gültiges Ziel behandeln |
| State Snapshot Reader | internen Vorher-Zustand und `clean`/`dirty` lesen | Zustand verändern oder Pfadlisten ausgeben |
| File Scope Guard | feste Datei-Allowlist, Repository-Grenze und regulären Dateityp prüfen | Symlinks oder freie Pfade zulassen |
| Read-only Command Executor | ausschließlich die acht Git-Argument-Suffixe und Dateisystem-Leseoperationen aus der Spezifikation direkt ohne Shell ausführen | andere Argumente, Shell, Netzwerk oder Schreiboperationen starten |
| Metadata Reader | höchstens drei zwölfstellige Commit-Kurz-IDs und erlaubte Dateimetadaten lesen | Inhalte, Commit-Texte, Autoren oder Remotes ausgeben |
| Stability Guard | internen Vorher-/Nachher-Zustand exakt vergleichen | Abweichungen als Erfolg behandeln |
| Result Builder | akzeptierten, neutralen oder abgelehnten Zustand bilden | Teilresultate nach Fehler ausgeben |
| Output Sanitizer | ausschließlich Schemafelder zulassen | Rohfehler oder interne Pfade durchlassen |
| Safe Failure Builder | bei Sanitizer- oder internem Ausgabefehler ein statisches minimales Ablehnungsobjekt ohne dynamische Daten bilden | Eingangs-, Pfad-, Git- oder Fehlerdetails übernehmen |
| Single Output Boundary | genau ein Endobjekt sicherstellen | null oder mehrere Ergebnisse liefern |

Eine Komponente zur Report-Persistenz gehört ausdrücklich nicht zu `ORT-001`.

---

## 4. Vertrauensgrenzen

```text
untrusted invocation
  -> Input Gate
  -> canonical process-start working directory
  -> Repository Guard and Git-Toplevel
  -> fixed identity marker
  -> local repository read boundary
  -> closed direct-process command boundary
  -> internal snapshot and metadata
  -> Output Sanitizer
  -> one sanitized in-memory result
```

Absolute Zielpfade dürfen ausschließlich innerhalb des Trusted Target Resolvers
existieren und keine Ausgabe-, Log- oder Reportgrenze überschreiten. Rohe
Git-Ausgaben, vollständige Statuspfade und technische Fehler bleiben intern.

---

## 5. Zustandsentscheidungen

Die geplante Entscheidungslogik lautet:

| Prüfergebnis | Interne Entscheidung | Öffentliche Ausgabe |
|---|---|---|
| unterstütztes Repository und `clean` | accept | `completed` / `accepted` |
| unterstütztes Repository und `dirty` | neutral report | `completed` / `reported-neutral` |
| ungültige Eingangsstruktur | reject | `INPUT_INVALID` |
| formal gültiger, aber nicht erlaubter Alias/Profilwert oder Marker | reject | `TARGET_NOT_ALLOWED` |
| nicht eindeutige oder nicht kanonische Zielauflösung | reject | `TARGET_RESOLUTION_FAILED` |
| kein Git-Repository | reject | `NOT_A_GIT_REPOSITORY` |
| nicht unterstützter Repository- oder Branchzustand | reject | `REPOSITORY_STATE_UNSUPPORTED` |
| Datei-Allowlist- oder Pfadgrenze verletzt | reject | `FILE_SCOPE_INVALID` |
| erlaubte lokale Leseoperation technisch fehlgeschlagen | reject | `LOCAL_READ_FAILED` |
| Vorher-/Nachher-Snapshot abweichend | reject | `STATE_CHANGED_DURING_RUN` |
| Sanitizer-, Schema-, Single-Output- oder unerwarteter interner Fehler | fail closed | statisches `SAFE_FAILURE` |

Andere öffentliche Gründe sind unzulässig. Die Reihenfolge und genaue
Zuordnung aus `SPECIFICATION_v0.2.0.md` und `FLOW_v0.2.0.md` sind verbindlich.
Es werden keine v0.1.1-Fehlercodes für lokale Zustände umgedeutet.

---

## 6. Stabilitätsgrenze

Der Runner muss das in Abschnitt 9.5 der Spezifikation vollständig definierte
geordnete Tupel unmittelbar vor und nach den erlaubten Leseoperationen erfassen.
Es umfasst kanonische Root und Toplevel, `.git`- und lokale Object-Store-
Identität samt bestätigter Abwesenheit eines alternativen Object Stores, Bare-Wert,
Branchbytes, vollständigen HEAD, Index- und Statusrohbytes, höchstens drei
vollständige Commit-IDs sowie `lstat`- und SHA-256-Werte aller acht Dateien.
Beide Snapshots müssen bytegenau identisch sein. Die internen Pfade und Rohwerte
dürfen den Sanitizer nicht passieren.

Eine Abweichung bedeutet nicht, dass der Runner sie verursacht hat. Sie macht
den Lauf jedoch nicht reproduzierbar und muss deshalb zur kontrollierten
Ablehnung ohne Teilmetadaten führen.

---

## 7. Speicherarchitektur

Der operative Lauf besitzt keinen File Writer und keinen Export Node. Das
sanitisierte Objekt bleibt bis zum Ende des Laufs im kontrollierten
Ausführungskontext.

Eine spätere Evidenzablage ist ein eigener, manuell autorisierter
Dokumentationsprozess außerhalb von `ORT-001`. Dieser Prozess darf nur ein zuvor
validiertes Objekt unter einem neuen repository-relativen Pfad in `exports/`
ablegen und niemals vorhandene Nachweise überschreiben.

---

## 8. Determinismus

Der Metadata Reader muss Commit-Kurz-IDs newest-first und Dateimetadaten in
statischer Allowlist-Reihenfolge liefern. Result Builder und Output Sanitizer
dürfen keine Zeitstempel, Zufallswerte oder automatisch erzeugten Kennungen
ergänzen.

Bei identischer Eingabe und identischem internem Vorher-/Nachher-Snapshot muss
das Ergebnis feldgleich sein. Eine kanonische JSON-Serialisierung ist vor einem
byteidentischen Artefaktvergleich gesondert festzulegen.

Der Sanitizer gibt ausschließlich die zwei geschlossenen Schemata aus der
Spezifikation aus. Branchwerte werden ausschließlich als SHA-256 repräsentiert;
Pfade sind ausschließlich feste Allowlist-Konstanten. Commit- und Datei-Hashes,
Integer und statische Enumwerte sind die einzigen weiteren dynamischen Werte.
Unbekannte Felder werden nicht entfernt oder ignoriert, sondern lösen vor
Ausgabe das statische `SAFE_FAILURE`-Objekt aus.

---

## 9. Verbotene Komponenten und Aktionen

Nicht zulässig sind:

- n8n-Nodes und Workflow-Trigger,
- HTTP-, GitHub-, Netzwerk- oder Credential-Adapter,
- Git-Remote-Kommandos,
- Datei-, Repository-, Commit-, Push- oder Tag-Writer,
- automatische Log- oder Reportpersistenz,
- jede Shell sowie Kommandos oder Argumente außerhalb der geschlossenen
  Allowlist aus Abschnitt 9 der Spezifikation,
- Aufruf von WF-0011 oder einem anderen Workflow.

---

## 10. Freigabegates vor Implementierung

1. Spezifikation, Architektur, Flow und Testplan sind nach den vorliegenden
   Ergänzungen erneut widerspruchsfrei auditiert.
2. Die Implementierung beschränkt sich nachweisbar auf den einzigen Alias, die
   feste Datei-Allowlist und die geschlossene Kommando-Allowlist.
3. Report Builder, Sanitizer, Safe Failure Builder und Single Output Boundary
   setzen die geschlossenen Schemata ohne Zusatzfelder um.
4. Alle geplanten statischen, isolierten und lokalen Tests sind bestanden.
5. Die organisatorische Trennung der späteren Evidenzablage bleibt bestätigt.

Gate 1 entscheidet über die fachliche Implementierungsfreigabe. Die Gates 2 bis
5 entscheiden anschließend über die operative Freigabe; ohne sie darf
`ORT-001` nicht ausgeführt werden.

---

## 11. Aktueller Architekturstatus

```text
Version: v0.2.0
Status: draft
Umsetzungsstatus: not-started
Implementierte Komponenten: 0
ORT-001 ausgeführt: nein
```
