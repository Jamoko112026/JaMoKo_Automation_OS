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
| Trusted Target Resolver | freigegebenen Alias aus statischer lokaler Konfiguration auflösen | freie oder ausgegebene absolute Pfade akzeptieren |
| Repository Guard | Git-Repository und unterstützten symbolischen Branch prüfen | Remote-Kommandos ausführen |
| State Snapshot Reader | internen Vorher-Zustand und `clean`/`dirty` lesen | Zustand verändern oder Pfadlisten ausgeben |
| File Scope Guard | feste Datei-Allowlist, Repository-Grenze und regulären Dateityp prüfen | Symlinks oder freie Pfade zulassen |
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
  -> trusted static target configuration
  -> local repository read boundary
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
| kein Git-Repository | reject | minimale `rejected`-Ausgabe |
| nicht unterstützter oder instabiler Zustand | reject | minimale `rejected`-Ausgabe |
| Sanitizer- oder interner Ausgabefehler | fail closed | statische minimale Ablehnung ohne dynamische Daten |

Die abschließenden öffentlichen Ablehnungsgründe einschließlich des statischen
Safe-Failure-Grunds sind noch nicht freigegeben. Es werden keine
v0.1.1-Fehlercodes für lokale Zustände umgedeutet.

---

## 6. Stabilitätsgrenze

Der Runner muss den vollständigen relevanten Git-Zustand intern unmittelbar vor
und nach den erlaubten Leseoperationen erfassen. Beide Snapshots müssen
bytegenau identisch sein. Die internen Pfade dürfen den Sanitizer nicht
passieren.

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

---

## 9. Verbotene Komponenten und Aktionen

Nicht zulässig sind:

- n8n-Nodes und Workflow-Trigger,
- HTTP-, GitHub-, Netzwerk- oder Credential-Adapter,
- Git-Remote-Kommandos,
- Datei-, Repository-, Commit-, Push- oder Tag-Writer,
- automatische Log- oder Reportpersistenz,
- Shell-Kommandos außerhalb einer abschließend geprüften Lese-Allowlist,
- Aufruf von WF-0011 oder einem anderen Workflow.

---

## 10. Freigabegates vor Implementierung

1. Spezifikation, Architektur, Flow und Testplan sind widerspruchsfrei geprüft.
2. Alias- und Datei-Allowlist sind abschließend freigegeben.
3. Zulässige lokale Lesekommandos und ihre Fehlergrenzen sind festgelegt.
4. Report-Schema und Ablehnungsgründe sind abschließend festgelegt.
5. Sanitizer-, Stabilitäts- und Single-Output-Konzept sind testbar beschrieben.
6. Die organisatorische Trennung der späteren Evidenzablage ist bestätigt.

Ohne diese Gates darf keine Implementierung begonnen und `ORT-001` nicht
ausgeführt werden.

---

## 11. Aktueller Architekturstatus

```text
Version: v0.2.0
Status: draft
Umsetzungsstatus: not-started
Implementierte Komponenten: 0
ORT-001 ausgeführt: nein
```
