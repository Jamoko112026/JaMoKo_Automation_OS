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
  -> Identity Marker Gate
  -> Initial State Snapshot
  -> State Classifier
  -> File Scope Guard
  -> Local Metadata Reader
  -> Final State Snapshot
  -> Stability Guard
  -> Result Builder
  -> Output Sanitizer
  -> Single Output Boundary

known gate rejection -> Static Rejection Builder -> Output Sanitizer
unexpected/schema/sanitizer failure -> Safe Failure Builder
Safe Failure Builder -> Single Output Boundary
```

Kein Schritt schreibt eine Datei oder persistiert einen Report.

---

## 3. Input Gate

Das Gate akzeptiert genau ein Objekt mit exakt:

```text
repository_alias = current-local-repository
read_profile = wf-0012-operational-minimal
```

Null oder mehrere Eingaben, fehlende oder zusätzliche Felder, Nicht-Strings und
leere Strings führen zu `INPUT_INVALID`. Formal gültige Strings mit
abweichendem Alias oder Profil führen zu `TARGET_NOT_ALLOWED`. Absolute oder
relative Pfade sind kein zulässiger Eingang. Werte werden nicht normalisiert.

---

## 4. Trusted Target Resolver

Der einzige Alias wird intern auf das von `getcwd` gelieferte Arbeitsverzeichnis
beim Prozessstart abgebildet. Der Resolver prüft ohne Shell dessen kanonische
Form und verbietet eine Abweichung durch Symlink-Auflösung. Null oder mehrere
Ziele sowie jede Abweichung führen zu `TARGET_RESOLUTION_FAILED`. Die intern
benötigte absolute Root wird nie ausgegeben.

---

## 5. Repository Guard

Der Guard prüft ausschließlich lokal und über die geschlossene Git-
Kommando-Allowlist:

1. Ziel ist ein Git-Repository.
2. `.git/objects` ist ein reales Verzeichnis und
   `.git/objects/info/alternates` fehlt.
3. Repository ist kein Bare-Repository.
4. Ein symbolischer Branch ist eindeutig bestimmbar.
5. HEAD bezeichnet einen vorhandenen Commit.
6. Index enthält keinen Submoduleintrag mit Mode `160000`.
7. Branchausgabe erfüllt die Bytegrenze der Spezifikation.

Das kanonische Git-Toplevel muss exakt mit der Root übereinstimmen; andernfalls
folgt `TARGET_RESOLUTION_FAILED`.

Ein Nicht-Git-Ordner wird wie in `LRT-003` sicher erkannt, aber nicht mit einem
veröffentlichten v0.1.1-Fehlercode versehen. Nicht nachgewiesene Zustände werden
mit `REPOSITORY_STATE_UNSUPPORTED` abgelehnt. Der eigenständige v0.2.0-Grund für
ein fehlendes `.git`-Verzeichnis lautet `NOT_A_GIT_REPOSITORY`.

---

## 5.1 Identity Marker Gate

Erst nach erfolgreichem Repository Guard wird der feste v0.1.1-Export als
Identitätsmarker geprüft. Pfad und erwarteter SHA-256 stehen unveränderlich in
der Spezifikation. Fehlender oder abweichender Marker führt zu
`TARGET_NOT_ALLOWED`. Dadurch behält ein leerer Nicht-Git-Ordner eindeutig den
Grund `NOT_A_GIT_REPOSITORY`.

---

## 6. Initial State Snapshot und State Classifier

Das vollständige geordnete Snapshot-Tupel aus Abschnitt 9.5 der Spezifikation
wird intern für den späteren Stabilitätsvergleich erfasst. Öffentlich wird aus
den NUL-separierten Porcelain-v1-Bytes nur klassifiziert:

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
- Pflichtdatei ist vorhanden, lesbar und höchstens 1048576 Byte groß,
- kanonischer Zielpfad ist exakt Root plus wortgleicher Allowlistwert.

Ein Scope-Fehler endet mit `FILE_SCOPE_INVALID` ohne teilweise Datei- oder
Repository-Metadaten. Ein technischer Fehler einer ansonsten gültigen lokalen
Leseoperation endet mit `LOCAL_READ_FAILED`.

---

## 8. Local Metadata Reader

Der Reader darf ausschließlich über die acht Git-Argument-Suffixe und die
Dateisystem-Leseoperationen aus Abschnitt 9 der Spezifikation lesen:

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

Git wird direkt ohne Shell, Expansion, Pipe oder Umleitung aufgerufen. Präfix,
Umgebung, Argumente, Zeit- und Ausgabelimits sowie Exitcode-Zuordnung sind
geschlossen in der Spezifikation festgelegt. Jeder nicht dort erlaubte Aufruf
ist ein Vertragsverstoß und darf nicht implementiert werden.

---

## 9. Final State Snapshot und Stability Guard

Nach dem Metadatenlesen wird der relevante Git-Zustand intern erneut erfasst.

```text
initial snapshot == final snapshot -> Fortsetzung
initial snapshot != final snapshot -> STATE_CHANGED_DURING_RUN
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
| `rejected` | statischer Grund gemäß vollständiger Matrix in Abschnitt 5 der Spezifikation |

Eine Ablehnung enthält intern keine für die öffentliche Ausgabe freigegebenen
Teilmetadaten.

---

## 11. Output Sanitizer

Der Sanitizer validiert gegen genau eines der beiden geschlossenen Schemata aus
`SPECIFICATION_v0.2.0.md`. Er ignoriert und entfernt keine unbekannten Felder;
ein abweichender Kandidat führt zum statischen `SAFE_FAILURE`-Objekt. Er muss
insbesondere blockieren:

- absolute Pfade und nicht allowlistfähige relative Pfade,
- Datei-Inhalte,
- vollständige Commit-IDs und Commit-Texte,
- Remotes, URLs, Credentials, Tokens und Secrets,
- rohe Git- und Systemfehler,
- Stack-Traces und Umgebungsdaten,
- interne Snapshots und temporäre Daten.

Der Branchname wird nie ausgegeben, sondern ausschließlich deterministisch als
SHA-256 der geprüften Branchbytes repräsentiert. Relative Pfade sind nur als
wortgleiche Allowlist-Konstanten zulässig. Ablehnungen enthalten ausschließlich
statische Enumwerte und niemals maskierte Rohdaten.

Kann ein Objekt nicht sicher sanitisiert werden, darf es nicht als
schemakonformer Erfolg oder neutrale Meldung ausgegeben werden. Stattdessen
muss ein separater Safe Failure Builder das exakte `rejected`-Schema mit
`reason = SAFE_FAILURE` ohne Eingangs-, Repository-, Pfad- oder Fehlerdetails
bilden.

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
eine minimale Ablehnung mit `reason = SAFE_FAILURE` erzeugen.

---

## 17. Aktueller Flow-Status

```text
Version: v0.2.0
Status: draft
Umsetzungsstatus: not-started
Ausführbare Schritte: 0
ORT-001 ausgeführt: nein
```
