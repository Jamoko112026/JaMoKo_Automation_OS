# WF-0012 – GitHub Reader

## Flow v0.2.0

Version: `v0.2.0`
Status: `draft`
Umsetzungsstatus: `not-started`
Run: `ORT-001`

---

## 1. Flow-Grenze

Dieser Flow ist ein Plan. Es existiert kein ausführbarer v0.2.0-Runner und
`ORT-001` wurde nicht gestartet. Der veröffentlichte v0.1.1-n8n-Workflow bleibt
unverändert und gehört nicht zu diesem lokalen Ablauf.

---

## 2. Geplanter Gesamtablauf

```text
Invocation
  -> Input Gate
  -> Trusted Target Resolver
  -> Repository Guard
  -> Initial State Snapshot
  -> State Classifier
  -> File Scope Guard
  -> Local Metadata Reader
  -> Final State Snapshot
  -> Stability Guard
  -> Result Builder
  -> Output Sanitizer
  -> Safe Failure Builder bei Sanitizer- oder internem Ausgabefehler
  -> Single Output Boundary
```

Kein Schritt schreibt eine Datei oder persistiert einen Report.

---

## 3. Input Gate

Das Gate akzeptiert genau ein Objekt mit exakt:

```text
repository_alias = current-local-repository
read_profile = wf-0012-operational-minimal
```

Fehlende, zusätzliche, leere oder abweichende Felder führen unmittelbar zum
minimalen Ablehnungspfad. Absolute Pfade sind kein zulässiger Eingang.

---

## 4. Trusted Target Resolver

Der Resolver darf den Alias ausschließlich über eine vorab geprüfte statische
lokale Konfiguration auflösen. Die technische Form dieser Konfiguration ist
noch offen. Ein Ziel aus Eingabedaten, Umgebungsdaten unbekannter Herkunft oder
Netzwerkdaten ist unzulässig.

Schlägt die eindeutige Auflösung fehl, wird vor jeder Repository-Leseoperation
abgebrochen.

---

## 5. Repository Guard

Der Guard prüft ausschließlich lokal:

1. Ziel ist ein Git-Repository.
2. Repository ist kein Bare-Repository.
3. Ein symbolischer Branch ist eindeutig bestimmbar.
4. Der Zustand gehört zum freigegebenen ersten Betriebsumfang.

Ein Nicht-Git-Ordner wird wie in `LRT-003` sicher erkannt, aber nicht mit einem
veröffentlichten v0.1.1-Fehlercode versehen. Nicht nachgewiesene Zustände werden
geschlossen abgelehnt.

---

## 6. Initial State Snapshot und State Classifier

Der vollständige relevante Git-Zustand wird intern für den späteren
Stabilitätsvergleich erfasst. Öffentlich wird nur klassifiziert:

```text
clean -> accepted
dirty -> reported-neutral
```

Ein `dirty` Zustand darf weder bereinigt noch verändert werden. Namen oder
Inhalte geänderter Pfade dürfen nicht Teil des öffentlichen Ergebnisses sein.

---

## 7. File Scope Guard

Für jeden statisch freigegebenen Pfad wird vor dem Metadatenlesen geprüft:

- Pfad ist repository-relativ und exakt allowlisted,
- aufgelöstes Ziel bleibt innerhalb des Repositorys,
- Ziel ist eine reguläre Datei und kein Symlink,
- Pflichtdatei ist vorhanden und lesbar.

Ein Fehler beendet den Lauf ohne teilweise Datei- oder Repository-Metadaten.

---

## 8. Local Metadata Reader

Der Reader darf ausschließlich lesen:

- lokalen symbolischen Branch,
- `clean` oder `dirty`,
- höchstens drei lokale Commit-IDs als jeweils exakt zwölf Hex-Zeichen,
- Existenz, Typ, Byte-Größe und SHA-256 allowlistfähiger Dateien.

Dateibytes dürfen nur flüchtig in die SHA-256-Berechnung einer zuvor geprüften
Datei eingehen. Sie werden nicht dekodiert, gespeichert, geloggt oder
ausgegeben. Ebenso wenig ausgegeben werden Commit-Texte, Autoren,
E-Mail-Adressen, Remotes, URLs, Credentials, Reflogs oder Git-Notizen.

Commit-Kurz-IDs werden newest-first ausgegeben und müssen
`^[a-f0-9]{12}$` entsprechen. Dateimetadaten folgen der statischen
Allowlist-Reihenfolge; SHA-256-Werte müssen `^[a-f0-9]{64}$` entsprechen.

---

## 9. Final State Snapshot und Stability Guard

Nach dem Metadatenlesen wird der relevante Git-Zustand intern erneut erfasst.

```text
initial snapshot == final snapshot -> Fortsetzung
initial snapshot != final snapshot -> kontrollierte Ablehnung
```

Die Abweichung wird neutral als Instabilität behandelt. Weder Ursache noch
verursachender Prozess werden behauptet. Interne Statuspfade bleiben verborgen.

---

## 10. Result Builder

Der Result Builder erzeugt genau eine interne Variante:

| Variante | Bedingung |
|---|---|
| `completed / accepted` | unterstütztes Repository, `clean`, stabil und vollständig gelesen |
| `completed / reported-neutral` | unterstütztes Repository, `dirty`, stabil und vollständig gelesen |
| `rejected` | Vorbedingung, Scope-, Lese-, Stabilitäts- oder Sicherheitsgate fehlgeschlagen |

Eine Ablehnung enthält intern keine für die öffentliche Ausgabe freigegebenen
Teilmetadaten.

---

## 11. Output Sanitizer

Der Sanitizer arbeitet mit einer geschlossenen Feld-Allowlist aus
`SPECIFICATION_v0.2.0.md`. Er muss insbesondere entfernen oder blockieren:

- absolute Pfade und nicht allowlistfähige relative Pfade,
- Datei-Inhalte,
- vollständige Commit-IDs und Commit-Texte,
- Remotes, URLs, Credentials, Tokens und Secrets,
- rohe Git- und Systemfehler,
- Stack-Traces und Umgebungsdaten,
- interne Snapshots und temporäre Daten.

Kann ein Objekt nicht sicher sanitisiert werden, darf es nicht als
schemakonformer Erfolg oder neutrale Meldung ausgegeben werden. Stattdessen
muss ein separater Safe Failure Builder ein statisches minimales
Ablehnungsobjekt ohne Eingangs-, Repository-, Pfad- oder Fehlerdetails bilden.
Der endgültige statische Ablehnungsgrund ist vor Implementierung festzulegen.

---

## 12. Single Output Boundary

Der Flow darf pro Aufruf genau ein sanitisiertes Objekt liefern. Null, mehrere
oder gemischte Ergebnisobjekte sind unzulässig.

Auf jedem zulässigen Ausgang gelten:

```text
mode = local-read-only
writeRequested = false
writeExecuted = false
commitCreated = false
pushExecuted = false
```

---

## 13. Verbotene Abzweigungen

Es darf keine Verbindung geben zu:

- n8n oder einem anderen Workflow,
- HTTP, GitHub API oder Git-Remotes,
- Credential Stores,
- Datei- oder Report-Writern,
- Git-Mutationen,
- WF-0011,
- Commit-, Push- oder Tag-Funktionen.

---

## 14. Wiederholbarkeit

Bei gleicher Eingabe und unverändertem internem Snapshot müssen Result Builder
und Sanitizer dasselbe feldgleiche Objekt erzeugen. Der Flow fügt keine
Zeitstempel, Zufallswerte oder automatisch erzeugten Laufkennungen hinzu.

---

## 15. Spätere Evidenzablage

Eine spätere Ablage des bereits sanitisierten Endobjekts unter `exports/` ist
kein Flow-Schritt. Sie benötigt eine separate Autorisierung, einen neuen
repository-relativen Dateinamen und eigene JSON-, Sanitizer- und Diff-Prüfungen.

---

## 16. Abbruchpunkte

Jedes Gate arbeitet fail-closed. Der Ablauf endet vor dem nächsten Leseschritt,
wenn sein Eingang nicht eindeutig gültig ist. Nach einem Abbruch werden keine
Teilresultate weitergereicht.

Ein unerwarteter Laufzeitfehler darf weder als `completed` erscheinen noch rohe
technische Details ausgeben. Er muss über den statischen Safe-Failure-Pfad genau
eine minimale Ablehnung erzeugen. Deren endgültiger Grund ist vor Implementierung
noch festzulegen und zu testen.

---

## 17. Aktueller Flow-Status

```text
Version: v0.2.0
Status: draft
Umsetzungsstatus: not-started
Ausführbare Schritte: 0
ORT-001 ausgeführt: nein
```
