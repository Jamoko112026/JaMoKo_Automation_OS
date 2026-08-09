# WF-0012 – GitHub Reader

## Specification v0.2.0

Version: `v0.2.0`
Status: `draft`
Umsetzungsstatus: `not-started`
Betriebsentwurf: `local-read-only`

---

## 1. Dokumentgrenze

Dieses Dokument plant den späteren Operational Read-only Run `ORT-001`.
Es beschreibt weder eine vorhandene Implementierung noch einen bereits
durchgeführten operativen Lauf.

Der veröffentlichte WF-0012-Vertrag bleibt unverändert `v0.1.1/released`.
Insbesondere werden dessen GitHub-Reader-Eingabe, Ausgabe, Fehlercodes und
Workflow-Export durch diesen Entwurf nicht ersetzt oder erweitert.

Die Nachweise `LRT-001`, `LRT-002` und `LRT-003` belegen ausschließlich die dort
dokumentierten lokalen Testbedingungen. Sie sind Voraussetzungen für die
Planung, aber keine Ausführung oder Freigabe von `ORT-001`.

---

## 2. Zweck von ORT-001

`ORT-001` soll später genau einen kontrollierten, lokalen und wiederholbaren
Read-only-Lauf gegen ein statisch freigegebenes Repository ausführen. Der Lauf
soll einen begrenzten Zustands-Snapshot bilden, ohne den Zielzustand zu ändern.

Der Lauf darf ausschließlich:

- erkennen, ob das Ziel ein Git-Repository ist,
- den symbolischen lokalen Branch lesen,
- den Working-Tree-Zustand als `clean` oder `dirty` bestimmen,
- höchstens drei auf exakt zwölf Hex-Zeichen verkürzte lokale Commit-IDs lesen,
- Metadaten statisch freigegebener Dateien lesen,
- genau ein sanitisiertes Ergebnisobjekt bilden.

`ORT-001` ist kein GitHub-API-Lauf, kein n8n-Lauf und kein Writer.

---

## 3. Zulässige Eingabe

Der spätere Runner darf genau ein Eingabeobjekt mit genau diesen Feldern
akzeptieren:

```json
{
  "repository_alias": "current-local-repository",
  "read_profile": "wf-0012-operational-minimal"
}
```

Für den ersten Entwurf sind ausschließlich die gezeigten Werte zulässig. Null
oder mehrere Eingaben, fehlende oder zusätzliche Felder, Nicht-Strings und
leere Strings führen zu `INPUT_INVALID`. Formal gültige, aber von den beiden
Konstanten abweichende Strings führen zu `TARGET_NOT_ALLOWED`. Eingabefelder
werden weder getrimmt noch normalisiert. Unbekannte oder zusätzliche
Eingabefelder sind unzulässig und werden nicht ignoriert.

Ein absoluter, relativer oder frei wählbarer Repository-Pfad ist kein
Eingabefeld. Der einzige Alias wird intern und unveränderlich auf das beim
Prozessstart vorhandene Arbeitsverzeichnis abgebildet. Es gibt genau einen Alias
und genau ein Ziel; null oder mehrere Auflösungen sind unzulässig.

Für die lokale Zielidentität gelten verbindlich:

1. Das von `getcwd` gelieferte Arbeitsverzeichnis wird über die Dateisystem-API
   ohne Shell auf seinen kanonischen absoluten Pfad aufgelöst. Weicht der
   `getcwd`-Wert von dieser kanonischen Form ab, insbesondere durch auflösbare
   Symlink-Komponenten, folgt `TARGET_RESOLUTION_FAILED`. Da kein Pfad
   Eingabefeld ist, wird kein vom Aufrufer gelieferter Symlink-Pfad akzeptiert.
2. `<kanonische-root>/.git` muss per `lstat` ein reales Verzeichnis sein. Fehlt
   `.git`, folgt `NOT_A_GIT_REPOSITORY`; Datei, Symlink oder anderer Dateityp
   führt zu `REPOSITORY_STATE_UNSUPPORTED`.
   `<kanonische-root>/.git/objects` muss ebenfalls per `lstat` ein reales
   Verzeichnis sein. Die Datei `.git/objects/info/alternates` darf nicht
   existieren. Abweichungen führen zu `REPOSITORY_STATE_UNSUPPORTED`; damit
   sind externe oder mehrdeutige Object Stores ausgeschlossen.
3. Das intern gelesene Git-Toplevel muss nach kanonischer Auflösung exakt mit
   der kanonischen Root übereinstimmen. Jede Abweichung führt zu
   `TARGET_RESOLUTION_FAILED`.
4. Als statischer Identitätsmarker muss
   `02_WORKFLOWS/WF-0012_GitHub_Reader/exports/WF-0012_GitHub_Reader_v0.1.1.json`
   eine reguläre, nicht verlinkte Datei mit SHA-256
   `6a46efc2d85d949e743eb062400d8caea793bcaadd53e2e21b7751e9d97601bb`
   sein. Abweichung oder fehlender Marker führt zu `TARGET_NOT_ALLOWED`.
5. Kein intern verwendeter absoluter Pfad darf Result Builder, Sanitizer, Logs
   oder Report erreichen.

Jeder allowlistfähige relative Dateipfad wird wortgleich an die kanonische Root
angehängt, per `lstat` geprüft und anschließend kanonisch aufgelöst. Die
kanonische Datei muss mit dem erwarteten Pfad `<root>/<Allowlistwert>` exakt
übereinstimmen und innerhalb der Root liegen. Symlinks in der Datei oder einem
Pfadsegment, `..`, alternative Schreibweisen und Mehrdeutigkeiten führen zu
`FILE_SCOPE_INVALID`.

---

## 4. Begrenzter Datenumfang

Das Profil `wf-0012-operational-minimal` darf nur folgende Daten lesen:

| Datenart | Zulässiger Umfang |
|---|---|
| Repository-Erkennung | Boolean: Git-Repository erkannt oder nicht erkannt |
| Branch | genau ein symbolischer lokaler Branch; öffentlich nur als SHA-256 der geprüften Branchbytes |
| Working Tree | ausschließlich `clean` oder `dirty`; keine Pfadliste im Report |
| Lokale Commits | höchstens drei lokale Commit-IDs, jeweils exakt die ersten zwölf Hex-Zeichen; keine Autoren, E-Mail-Adressen oder Commit-Texte |
| Dateimetadaten | repository-relativer Allowlist-Pfad, Existenz, regulärer Dateityp, Byte-Größe von `0` bis `1048576` und SHA-256 |

Commit-Kurz-IDs müssen `^[a-f0-9]{12}$` entsprechen und in absteigender lokaler
Commit-Reihenfolge ausgegeben werden. Datei-SHA-256-Werte müssen
`^[a-f0-9]{64}$` entsprechen. Die Dateiliste folgt immer der Reihenfolge der
statischen Allowlist.

Nach Entfernung genau eines abschließenden LF aus der Ausgabe des erlaubten
Branch-Kommandos müssen die Branchbytes 1 bis 255 Byte lang sein und dürfen kein
NUL, CR oder weiteres LF enthalten. Der Rohwert wird nie ausgegeben. Der Report
enthält ausschließlich seinen SHA-256 als 64-stelligen lowercase Hex-String.

Die statische Datei-Allowlist des ersten Profils umfasst ausschließlich:

- `02_WORKFLOWS/WF-0012_GitHub_Reader/README.md`
- `02_WORKFLOWS/WF-0012_GitHub_Reader/SPECIFICATION.md`
- `02_WORKFLOWS/WF-0012_GitHub_Reader/ARCHITECTURE.md`
- `02_WORKFLOWS/WF-0012_GitHub_Reader/FLOW.md`
- `02_WORKFLOWS/WF-0012_GitHub_Reader/TESTS.md`
- `02_WORKFLOWS/WF-0012_GitHub_Reader/CHANGELOG.md`
- `02_WORKFLOWS/WF-0012_GitHub_Reader/KNOWN_ISSUES.md`
- `02_WORKFLOWS/WF-0012_GitHub_Reader/exports/WF-0012_GitHub_Reader_v0.1.1.json`

Die Bytes einer allowlistfähigen regulären Datei dürfen ausschließlich als
flüchtiger Eingabestrom für ihre SHA-256-Berechnung gelesen werden. Sie dürfen
nicht dekodiert, als Nutzdaten gespeichert, geloggt oder ausgegeben werden.
Datei-Inhalte als Nutzdaten, absolute Pfade, Remote-Konfigurationen, URLs,
Credential-Daten, Tokens, Secrets, Git-Notizen, Reflogs und sonstige Git-Objekte
liegen außerhalb des zulässigen Datenumfangs.

Symlinks und andere nicht reguläre Dateitypen dürfen nicht als erlaubte Dateien
verarbeitet werden. Ein SHA-256 darf nur für eine nach Allowlist, Repository-
Grenze, Größe und Dateityp vollständig geprüfte Datei berechnet werden. Alle
acht Allowlist-Dateien sind Pflichtdateien; deshalb enthält ein
`completed`-Report exakt acht Dateiobjekte.

---

## 5. Zustände und geschlossene Ablehnungsgründe

Die folgende Liste ist vollständig. Andere öffentliche `reason`-Werte sind
unzulässig und dürfen weder dynamisch erzeugt noch aus technischen Fehlertexten
abgeleitet werden:

```text
INPUT_INVALID
TARGET_NOT_ALLOWED
TARGET_RESOLUTION_FAILED
NOT_A_GIT_REPOSITORY
REPOSITORY_STATE_UNSUPPORTED
FILE_SCOPE_INVALID
LOCAL_READ_FAILED
STATE_CHANGED_DURING_RUN
SAFE_FAILURE
```

Die verbindliche Zuordnung lautet:

| Gate oder Zustand | Report | `outcome` oder `reason` |
|---|---|---|
| Exakt eine gültige Eingabe, erlaubtes und stabiles Repository, Working Tree `clean`, alle acht Dateien gültig gelesen | `status = completed` | `outcome = accepted` |
| Wie zuvor, Working Tree `dirty` | `status = completed` | `outcome = reported-neutral` |
| Null oder mehrere Eingaben; Eingabe nicht Objekt; Pflichtfeld fehlt, leer oder kein String; zusätzliches Feld | `status = rejected` | `reason = INPUT_INVALID` |
| Formal gültige Strings, aber Alias oder Profil weicht vom einzigen erlaubten Wert ab; Identitätsmarker fehlt oder weicht ab | `status = rejected` | `reason = TARGET_NOT_ALLOWED` |
| Arbeitsverzeichnis nicht kanonisch, enthält Symlink-Komponenten, Alias löst auf null oder mehrere Ziele auf oder Git-Toplevel weicht ab | `status = rejected` | `reason = TARGET_RESOLUTION_FAILED` |
| `<root>/.git` fehlt | `status = rejected` | `reason = NOT_A_GIT_REPOSITORY` |
| `.git` oder `.git/objects` ist kein reales Verzeichnis; alternativer Object Store vorhanden; Bare-Repository, Detached HEAD, ungeborener Branch, ungültige Branchbytes, Submodule oder verknüpfter Worktree | `status = rejected` | `reason = REPOSITORY_STATE_UNSUPPORTED` |
| Andere Allowlist-Pflichtdatei als der bereits im Identity Marker Gate geprüfte Marker fehlt, ist zu groß, unlesbar, kein regulärer Dateityp, ein Symlink, außerhalb der Root oder kanonisch abweichend | `status = rejected` | `reason = FILE_SCOPE_INVALID` |
| Erlaubte lokale Git- oder Dateisystem-Leseoperation endet mit nicht separat zugeordnetem Exitcode, Timeout, Ausgabeüberschreitung oder ungültigem Format | `status = rejected` | `reason = LOCAL_READ_FAILED` |
| Relevanter Vorher- und Nachher-Snapshot weichen ab | `status = rejected` | `reason = STATE_CHANGED_DURING_RUN` |
| Unerwartete interne Ausnahme; Result Builder, Schema-Prüfung, Sanitizer oder Single Output Boundary kann kein gültiges Objekt garantieren | `status = rejected` | `reason = SAFE_FAILURE` |

Die Gates werden ausschließlich in der in `FLOW_v0.2.0.md` festgelegten
Reihenfolge ausgewertet. Bei mehreren gleichzeitig vorhandenen fachlichen
Fehlern gilt der Grund des zuerst erreichten fehlgeschlagenen Gates. Eine
unerwartete interne Ausnahme führt unabhängig vom aktuellen Gate immer zu
`SAFE_FAILURE`.

Ein `dirty` Working Tree ist weder ein sauberer Zustand noch ein technischer
Fehler. Er wird neutral gemeldet. Der Lauf darf ihn nicht bereinigen,
verändern, stagen oder anderweitig behandeln.

---

## 6. Geschlossener Report-Vertrag

Es existieren genau zwei disjunkte Objektschemata: `completed` und `rejected`.
Alle aufgeführten Felder sind Pflichtfelder; optionale Felder gibt es nicht.
Unbekannte oder zusätzliche Felder werden niemals ignoriert. In einer Eingabe
führen sie zu `INPUT_INVALID`; in einem intern gebildeten Reportkandidaten zu
der statischen `SAFE_FAILURE`-Ausgabe. Ein Verbraucher muss ein abweichendes
Objekt als vertragswidrig ablehnen.

### 6.1 Top-Level-Schema bei `completed`

Die Felder erscheinen in dieser festen Reihenfolge:

| Feld | Typ | Kardinalität | Zulässiger Wert |
|---|---|---|---|
| `run` | String | genau 1 | exakt `ORT-001` |
| `specification_version` | String | genau 1 | exakt `v0.2.0` |
| `specification_status` | String | genau 1 | exakt `draft` |
| `status` | String | genau 1 | exakt `completed` |
| `outcome` | String | genau 1 | `accepted` oder `reported-neutral` |
| `mode` | String | genau 1 | exakt `local-read-only` |
| `repository` | Objekt | genau 1 | exakt das Schema aus 6.2 |
| `recent_commits` | Array | genau 1 | 1 bis 3 Objekte nach 6.3, newest-first |
| `files` | Array | genau 1 | exakt 8 Objekte nach 6.4 in Allowlist-Reihenfolge |
| `writeRequested` | Boolean | genau 1 | exakt `false` |
| `writeExecuted` | Boolean | genau 1 | exakt `false` |
| `commitCreated` | Boolean | genau 1 | exakt `false` |
| `pushExecuted` | Boolean | genau 1 | exakt `false` |

### 6.2 Repository-Objekt

| Feld | Typ | Kardinalität | Zulässiger Wert |
|---|---|---|---|
| `alias` | String | genau 1 | exakt `current-local-repository` |
| `git_repository` | Boolean | genau 1 | exakt `true` |
| `branch` | Objekt | genau 1 | exakt `representation` und `sha256` |
| `working_tree` | String | genau 1 | `clean` oder `dirty` |

Das Branch-Objekt enthält exakt:

| Feld | Typ | Kardinalität | Zulässiger Wert |
|---|---|---|---|
| `representation` | String | genau 1 | exakt `sha256` |
| `sha256` | String | genau 1 | exakt `^[a-f0-9]{64}$` |

Der rohe Branchname ist in jedem öffentlichen Ergebnis unzulässig. Bei
`working_tree = clean` muss `outcome = accepted` gelten; bei
`working_tree = dirty` muss `outcome = reported-neutral` gelten. Jede andere
Kombination führt vor Ausgabe zu `SAFE_FAILURE`.

### 6.3 Commit-Objekte

Jedes Objekt enthält ausschließlich `short_sha` als String mit exakt
`^[a-f0-9]{12}$`. Das Array enthält mindestens den verifizierten `HEAD` und
höchstens drei Einträge in newest-first-Reihenfolge. Null Einträge oder weitere
Felder führen zu `SAFE_FAILURE`.

### 6.4 Dateiobjekte

Jedes der exakt acht Objekte enthält in dieser Reihenfolge:

| Feld | Typ | Kardinalität | Zulässiger Wert |
|---|---|---|---|
| `path` | String | genau 1 | exakt der jeweilige Wert der statischen Datei-Allowlist |
| `exists` | Boolean | genau 1 | exakt `true` |
| `type` | String | genau 1 | exakt `regular-file` |
| `size_bytes` | Integer | genau 1 | `0` bis `1048576` einschließlich |
| `sha256` | String | genau 1 | exakt `^[a-f0-9]{64}$` |

Fehlt eine Pflichtdatei, entsteht kein `completed`-Report.

### 6.5 Top-Level-Schema bei `rejected`

Die Felder erscheinen in dieser festen Reihenfolge:

| Feld | Typ | Kardinalität | Zulässiger Wert |
|---|---|---|---|
| `run` | String | genau 1 | exakt `ORT-001` |
| `specification_version` | String | genau 1 | exakt `v0.2.0` |
| `specification_status` | String | genau 1 | exakt `draft` |
| `status` | String | genau 1 | exakt `rejected` |
| `mode` | String | genau 1 | exakt `local-read-only` |
| `reason` | String | genau 1 | exakt einer der neun Werte aus Abschnitt 5 |
| `writeRequested` | Boolean | genau 1 | exakt `false` |
| `writeExecuted` | Boolean | genau 1 | exakt `false` |
| `commitCreated` | Boolean | genau 1 | exakt `false` |
| `pushExecuted` | Boolean | genau 1 | exakt `false` |

`outcome`, `repository`, `recent_commits`, `files` und jede andere Metadaten-
oder Fehlerdetailstruktur sind bei `rejected` unzulässig. `SAFE_FAILURE` wird
als vollständig statisches Objekt ausschließlich aus den Konstanten dieses
Schemas erzeugt und übernimmt keinerlei dynamische Daten.

### 6.6 Serialisierung

Falls das Objekt nach der getrennten Autorisierung aus Abschnitt 7 als JSON
serialisiert wird, gelten UTF-8 ohne BOM, zwei Leerzeichen Einrückung, LF als
Zeilenende, genau ein finales LF und die in 6.1 bis 6.5 festgelegte
Feldreihenfolge. Zahlen werden dezimal ohne Exponent geschrieben. Da alle
zulässigen freien Repräsentationen ASCII-Konstanten oder lowercase Hexwerte
sind, ist keine inhaltliche Unicode-Normalisierung erforderlich oder erlaubt.

Die LRT-Bezeichnung `not-a-git-repository` bleibt ein lokaler Nachweiswert und
wird nicht rückwirkend zu einem Fehlercode von v0.1.1. Der v0.2.0-Grund lautet
eigenständig `NOT_A_GIT_REPOSITORY`.

---

## 7. Speicherregeln

`ORT-001` selbst darf keine Reportdatei erzeugen oder überschreiben. Sein
Read-only-Lauf endet mit genau einem sanitisierten Objekt im kontrollierten
Ausführungskontext.

Eine Ablage unter `exports/` wäre eine separate, nachgelagerte und ausdrücklich
zu autorisierende Dokumentationshandlung. Sie gehört nicht zum operativen Lauf,
muss einen neuen repository-relativen Dateinamen verwenden und darf keine
bestehende Datei überschreiben. Automatische Logs oder unbereinigte
Zwischenspeicherung sind unzulässig.

Vor einer solchen Ablage muss nachgewiesen sein, dass das Objekt dem finalen
Schema entspricht und keine absoluten Pfade, Datei-Inhalte, Remotes,
Credentials, Tokens, Secrets oder rohe Fehler enthält.

---

## 8. Harte Verbote

Für `ORT-001` sind ausnahmslos verboten:

- n8n-Ausführung oder Aufruf eines anderen Workflows,
- HTTP- oder sonstiger Netzwerkzugriff,
- GitHub API und Git-Remote-Zugriff,
- Lesen oder Verwenden von Credentials,
- mutierende Datei- oder Verzeichnisoperationen,
- `git add`, Commit, Push, Tag, Fetch, Pull, Merge, Checkout oder Reset,
- Aufruf oder Vorbereitung von WF-0011,
- Ausgabe von absoluten Pfaden, Datei-Inhalten oder sensitiven Rohdaten,
- automatische Speicherung des Reports.

---

## 9. Technischer Read-only-Ausführungsvertrag

### 9.1 Prozessgrenze

Git wird ausschließlich als direkter Kindprozess mit einem Argument-Array
gestartet. Eine Shell, `eval`, Pipes, Umleitungen, Globs, Command Substitution,
Aliases und benutzerdefinierte Git-Aliases sind unzulässig. `<ROOT>` bezeichnet
in der folgenden Liste ausschließlich die intern kanonisch geprüfte Root und
wird als genau ein Argument übergeben.

Jeder Git-Aufruf verwendet exakt diesen unveränderlichen Präfix:

```text
git --no-pager --no-optional-locks
  --no-replace-objects
  --no-lazy-fetch
  -c core.fsmonitor=false
  -c core.untrackedCache=false
  -c color.ui=false
  -c core.quotePath=true
  -C <ROOT>
```

Zeilenumbrüche dienen hier nur der Lesbarkeit und erzeugen keine Shell-Syntax.
Der Kindprozess erhält eine ansonsten leere Umgebung mit exakt:

```text
LC_ALL=C
LANG=C
GIT_OPTIONAL_LOCKS=0
GIT_NO_REPLACE_OBJECTS=1
GIT_TERMINAL_PROMPT=0
GIT_CONFIG_NOSYSTEM=1
GIT_CONFIG_GLOBAL=<system-null-device>
```

Es werden keine weiteren Variablen geerbt; insbesondere fehlen `HOME`, `PATH`,
alle Proxy-, SSH-, Askpass- und Credential-Variablen. Das Git-Executable wird
ohne PATH-Suche aus einer bei Installation geprüften lokalen
Executable-Referenz gestartet; diese Referenz ist weder Eingabe noch
Reportfeld. Vor Freigabe einer Implementierung muss dieselbe Referenz die
Option `--no-lazy-fetch` nachweislich unterstützen; andernfalls darf ORT-001
nicht gestartet werden. `<system-null-device>` wird intern einmal über die Laufzeitplattform
auf deren Nullgerät aufgelöst und nie ausgegeben.

Jeder Prozess besitzt ein Zeitlimit von 5 Sekunden, ein stdout-Limit von
1048576 Byte und ein stderr-Limit von 65536 Byte. stderr wird nur begrenzt
verworfen und nie geloggt, geparst oder ausgegeben. Timeout, Limitüberschreitung
oder Startfehler führt zu `LOCAL_READ_FAILED`.

### 9.2 Geschlossene Git-Kommando-Allowlist

Nach dem gemeinsamen Präfix ist ausschließlich eines dieser Argument-Suffixe
zulässig:

| ID | Exaktes Argument-Suffix | Zweck |
|---|---|---|
| `GIT-01` | `rev-parse --is-inside-work-tree` | Worktree bestätigen |
| `GIT-02` | `rev-parse --is-bare-repository` | Bare-Zustand ausschließen |
| `GIT-03` | `rev-parse --show-toplevel` | interne Root-Übereinstimmung prüfen |
| `GIT-04` | `symbolic-ref --quiet --short HEAD` | symbolischen Branch intern lesen |
| `GIT-05` | `rev-parse --verify HEAD^{commit}` | vorhandenen HEAD-Commit bestätigen |
| `GIT-06` | `ls-files --stage -z` | Index und Submodule intern prüfen |
| `GIT-07` | `status --porcelain=v1 -z --untracked-files=all --ignore-submodules=none` | vollständigen lokalen Working-Tree-Status intern lesen |
| `GIT-08` | `log -n 3 --format=%H` | höchstens drei lokale Commit-IDs lesen |

Andere Git-Kommandos, Optionen, Konfigurationswerte oder Argumente sind
unzulässig. Insbesondere sind Remote-Namen, URLs und Revisionen aus der Eingabe
kein Argument.

### 9.3 Exitcode- und Formatzuordnung

| Operation | Erwartung | Abweichung |
|---|---|---|
| Vorprüfung `<ROOT>/.git` | reales Verzeichnis | fehlt: `NOT_A_GIT_REPOSITORY`; anderer Typ: `REPOSITORY_STATE_UNSUPPORTED` |
| `GIT-01` | Exit 0, stdout exakt `true` plus LF | jeder andere Exit oder Wert: `LOCAL_READ_FAILED` |
| `GIT-02` | Exit 0, stdout exakt `false` plus LF | `true` plus LF: `REPOSITORY_STATE_UNSUPPORTED`; sonst `LOCAL_READ_FAILED` |
| `GIT-03` | Exit 0, genau ein Pfad plus LF; kanonisch exakt `<ROOT>` | Pfadabweichung: `TARGET_RESOLUTION_FAILED`; sonst `LOCAL_READ_FAILED` |
| `GIT-04` | Exit 0, genau ein LF-terminierter Branchwert nach Abschnitt 4 | Exit 1 oder ungültige Branchbytes: `REPOSITORY_STATE_UNSUPPORTED`; anderer Exit: `LOCAL_READ_FAILED` |
| `GIT-05` | Exit 0, genau eine lowercase Hex-ID mit 40 oder 64 Zeichen plus LF | Exit 128: `REPOSITORY_STATE_UNSUPPORTED`; sonst `LOCAL_READ_FAILED` |
| `GIT-06` | Exit 0, gültige NUL-separierte Stage-Einträge | Mode `160000`: `REPOSITORY_STATE_UNSUPPORTED`; ungültiges Format oder anderer Exit: `LOCAL_READ_FAILED` |
| `GIT-07` | Exit 0, gültige Porcelain-v1-`-z`-Bytes | anderer Exit oder ungültiges Format: `LOCAL_READ_FAILED` |
| `GIT-08` | Exit 0, 1 bis 3 lowercase Hex-IDs mit jeweils 40 oder 64 Zeichen | anderer Exit, Anzahl oder Format: `LOCAL_READ_FAILED` |
| `lstat`, `realpath`, read-only `open`, `fstat`, SHA-256 | erwarteter Dateityp, Pfad und Wert | Markerabweichung: `TARGET_NOT_ALLOWED`; sonstige Scope-Abweichung: `FILE_SCOPE_INVALID`; technischer Lesefehler: `LOCAL_READ_FAILED` |

`GIT-06` wird als Folge NUL-terminierter Records mit exakt
`<mode> SP <object-id> SP <stage> TAB <path> NUL` validiert. `mode` besitzt
sechs Oktalziffern, `object-id` 40 oder 64 lowercase Hex-Zeichen, `stage` genau
eine Ziffer von `0` bis `3`, und `path` ist intern nicht leer. Nur Mode `160000`
hat die fachliche Sonderwirkung `REPOSITORY_STATE_UNSUPPORTED`; Pfade verlassen
nie die interne Grenze.

Die stdout-Bytes von `GIT-07` werden nicht in Pfadwerte zerlegt. Wegen direktem
Aufruf des fest konfigurierten Git-Executables, Exit 0, Porcelain v1 und `-z`
ist jede Ausgabe bis zum Größenlimit ein gültiger interner Snapshot: null Byte
Ausgabe bedeutet `clean`, jede nicht leere Ausgabe `dirty`. Dadurch werden
weder Pfade interpretiert noch in den Report übernommen.

Nur die ausdrücklich genannten Exitcodes besitzen eine fachliche Zuordnung.
Signale und alle übrigen Exitcodes führen zu `LOCAL_READ_FAILED`. Eine
unerwartete Ausnahme außerhalb einer kontrolliert fehlgeschlagenen erlaubten
Operation führt zu `SAFE_FAILURE`.

### 9.4 Erlaubte Dateisystemoperationen

Ohne externes Kommando sind ausschließlich `getcwd`, `realpath`, `lstat`,
read-only `open` mit No-Follow-Schutz, `fstat`, sequentielles Lesen zur
SHA-256-Berechnung und `close` zulässig. Erzeugen, Löschen, Umbenennen,
Schreiben, Rechteänderungen und Zeitstempeländerungen durch den Runner sind
unzulässig.

### 9.5 Relevanter Git-Zustand

Der interne Snapshot ist ein geordnetes Tupel aus exakt:

1. kanonischer Root und kanonischem Git-Toplevel,
2. `lstat`-Typ, Device und Inode der realen Verzeichnisse `.git` und
   `.git/objects` sowie die bestätigte Abwesenheit von
   `.git/objects/info/alternates`,
3. Bare-Wert aus `GIT-02`,
4. geprüften rohen Branchbytes aus `GIT-04`,
5. vollständiger HEAD-ID aus `GIT-05`,
6. vollständigen Rohbytes aus `GIT-06`,
7. vollständigen Rohbytes aus `GIT-07`,
8. vollständiger geordneter Commit-ID-Liste aus `GIT-08`,
9. pro Allowlist-Datei in Allowlist-Reihenfolge: relativer Pfad, Device, Inode,
   Modus, Byte-Größe, Nanosekunden-Mtime und SHA-256.

Unmittelbar vor und nach der Metadatenbildung wird dieses gesamte Tupel neu
erfasst. Nur bytegenaue Gleichheit aller Elemente ist stabil. Jede Abweichung
führt zu `STATE_CHANGED_DURING_RUN`; die internen Snapshotwerte werden nicht
ausgegeben. `working_tree = clean` gilt ausschließlich bei leerer Ausgabe von
`GIT-07`, andernfalls gilt `dirty`.

---

## 10. Wiederholbarkeit

Bei gleicher freigegebener Eingabe und bytegenau unverändertem internem
Repository-Snapshot muss `ORT-001` ein semantisch identisches Ergebnisobjekt
bilden. Automatisch erzeugte Zeitstempel, Zufallswerte, Laufkennungen oder eine
vom Dateisystem abhängige Listenreihenfolge sind im Ergebnis unzulässig.

Die Commit-Liste ist newest-first auf höchstens drei Einträge begrenzt. Die
Dateiliste folgt der dokumentierten Allowlist. Eine spätere Serialisierung muss
eine festgelegte Feldreihenfolge verwenden, wenn ein byteidentischer
Artefaktvergleich gefordert wird.

Das Sanitizing ist eine geschlossene Validierung und keine heuristische
Textbereinigung:

- Der rohe Branchwert wird deterministisch als SHA-256 über exakt die nach
  Abschnitt 4 geprüften Branchbytes dargestellt; ausgegeben werden nur
  `representation = sha256` und 64 lowercase Hex-Zeichen.
- Pfade werden nicht maskiert oder normalisiert. Ausschließlich die acht
  wortgleichen Allowlist-Konstanten dürfen in einem `completed`-Report stehen.
- Commit-IDs werden validiert und deterministisch auf die ersten zwölf
  lowercase Hex-Zeichen verkürzt.
- Datei-Hashes werden als 64 lowercase Hex-Zeichen und Größen als Integer
  ausgegeben.
- Ablehnungen bestehen nur aus statischen Schema- und Reason-Konstanten. Rohe
  Eingaben oder Fehlermeldungen werden nicht maskiert, sondern vollständig
  verworfen.
- Für unbekannte Felder oder nicht allowlistfähige dynamische Strings gibt es
  keine Ersatzdarstellung; der Reportkandidat wird verworfen und durch das
  statische `SAFE_FAILURE`-Objekt ersetzt.

Damit können Tokens, Remotes, absolute Pfade, Datei-Inhalte und andere freie
Texte keinen zulässigen Ausgabepfad erreichen. Gleiche geprüfte Branchbytes,
Commit-IDs und Dateimetadaten erzeugen stets dieselbe Repräsentation.

---

## 11. Vorbedingungen

`ORT-001` darf erst ausgeführt werden, wenn mindestens nachgewiesen ist:

1. Ein separater lokaler Runner ist implementiert und unabhängig geprüft.
2. Aliasauflösung, Befehls-Allowlist und Datei-Allowlist sind geschlossen und
   unveränderbar für den Lauf geladen.
3. Der Runner kann keine Netzwerk-, Remote-, Credential- oder Writer-Funktion
   aufrufen.
4. Ein zentraler Sanitizer und eine Single-Output-Grenze sind implementiert.
5. Der vollständige Working-Tree-Zustand kann intern vor und nach dem Lauf
   verglichen werden, ohne Pfade öffentlich auszugeben.
6. Alle geplanten v0.2.0-Struktur-, Negativ-, Stabilitäts-, Sanitizer- und
   Seiteneffektfreiheitstests sind bestanden und nachvollziehbar belegt.
7. Die zulässigen Ablehnungsgründe und das endgültige Report-Schema sind
   dokumentarisch freigegeben.
8. Die Speicherung von Laufnachweisen ist technisch und organisatorisch vom
   Read-only-Lauf getrennt.

Solange eine Vorbedingung fehlt, bleibt `ORT-001` `not-started`.

---

## 12. Abbruchbedingungen

Der Lauf muss ohne Teil-Erfolg abbrechen, wenn:

- die Eingabe nicht exakt dem freigegebenen Profil entspricht,
- das Ziel nicht eindeutig lokal aufgelöst werden kann,
- das Ziel kein unterstütztes Git-Repository ist,
- ein nicht freigegebener Repository-Zustand erkannt wird,
- ein erlaubter Pfad die Repository-Grenze verlässt oder kein regulärer
  Dateipfad ist,
- eine erforderliche lokale Leseoperation fehlschlägt,
- sich der intern verglichene Repository-Zustand während des Laufs ändert,
- der Sanitizer oder die Single-Output-Grenze fehlschlägt,
- eine verbotene Aktion erforderlich würde.

Bei Abbruch dürfen keine teilweise gelesenen Repository-, Commit- oder
Dateimetadaten ausgegeben werden.

---

## 13. Erfolgskriterien

Ein späterer `ORT-001`-Lauf ist nur erfolgreich, wenn:

- alle Vorbedingungen erfüllt waren,
- genau ein freigegebenes lokales Repository gelesen wurde,
- nur der festgelegte Datenumfang verarbeitet wurde,
- der Vorher-/Nachher-Zustand intern identisch ist,
- genau ein schemakonformes sanitisiertes Objekt entstand,
- alle vier Schreibschutzwerte `false` sind,
- keine verbotene Aktion stattfand,
- keine automatische Reportdatei erzeugt wurde.

---

## 14. Aktueller Planungsstatus

```text
Version: v0.2.0
Status: draft
Umsetzungsstatus: not-started
ORT-001 ausgeführt: nein
```

Der nächste zulässige Schritt ist ein Vertragsaudit dieses Entwurfs. Erst nach
separater Freigabe dürfen Implementierung und Tests geplant werden.
