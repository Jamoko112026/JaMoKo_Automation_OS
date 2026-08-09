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

Für den ersten Entwurf sind ausschließlich die gezeigten Werte zulässig.
Weitere Felder, mehrere Eingabeobjekte, leere Werte und andere Werte werden
abgelehnt.

Ein absoluter oder frei wählbarer Repository-Pfad ist kein Eingabefeld. Der
Alias muss später über eine lokal gepflegte, statische und vor Ausführung
geprüfte Konfiguration auf genau ein Ziel aufgelöst werden. Diese Auflösung ist
noch nicht implementiert.

---

## 4. Begrenzter Datenumfang

Das Profil `wf-0012-operational-minimal` darf nur folgende Daten lesen:

| Datenart | Zulässiger Umfang |
|---|---|
| Repository-Erkennung | Boolean: Git-Repository erkannt oder nicht erkannt |
| Branch | genau ein symbolischer lokaler Branchname |
| Working Tree | ausschließlich `clean` oder `dirty`; keine Pfadliste im Report |
| Lokale Commits | höchstens drei lokale Commit-IDs, jeweils exakt die ersten zwölf Hex-Zeichen; keine Autoren, E-Mail-Adressen oder Commit-Texte |
| Dateimetadaten | repository-relativer Pfad, Existenz, regulärer Dateityp, Byte-Größe und SHA-256 |

Commit-Kurz-IDs müssen `^[a-f0-9]{12}$` entsprechen und in absteigender lokaler
Commit-Reihenfolge ausgegeben werden. Datei-SHA-256-Werte müssen
`^[a-f0-9]{64}$` entsprechen. Die Dateiliste folgt immer der Reihenfolge der
statischen Allowlist.

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
Grenze und Dateityp vollständig geprüfte Datei berechnet werden.

---

## 5. Repository-Zustände

| Zustand | Behandlung |
|---|---|
| Git-Repository, symbolischer Branch, Working Tree `clean`, alle Pflichtmetadaten lesbar | akzeptieren und als `completed` reporten |
| Git-Repository, symbolischer Branch, Working Tree `dirty`, alle Pflichtmetadaten lesbar | neutral als `completed` mit `outcome = reported-neutral` reporten |
| Kein Git-Repository | kontrolliert als `rejected` ablehnen |
| Alias oder Profil nicht freigegeben | kontrolliert als `rejected` ablehnen |
| Detached HEAD, ungeborener Branch oder nicht eindeutig bestimmbarer Branch | bis zu einer eigenen Spezifikation kontrolliert als `rejected` ablehnen |
| Allowlist-Verletzung, Symlink, fehlende Pflichtdatei oder Pfad außerhalb des Repositorys | kontrolliert als `rejected` ablehnen |
| Lese-, Sanitizer- oder Stabilitätsprüfung schlägt fehl | kontrolliert als `rejected` ablehnen |

Ein `dirty` Working Tree ist weder Erfolg im Sinne eines sauberen Zustands noch
ein technischer Fehler. Der Zustand wird neutral gemeldet. Der Lauf darf ihn
nicht bereinigen, verändern, stagen oder anderweitig behandeln.

Weitere Repository-Zustände wie Submodule, verknüpfte Worktrees oder Bare-
Repositories sind für diesen Entwurf nicht freigegeben und müssen bis zu einem
eigenen Nachweis abgelehnt werden.

---

## 6. Entwurf des sanitisierten Reports

Ein akzeptierter sauberer oder neutral gemeldeter unsauberer Zustand soll genau
ein Objekt dieser Form ergeben:

```json
{
  "run": "ORT-001",
  "specification_version": "v0.2.0",
  "specification_status": "draft",
  "status": "completed",
  "outcome": "accepted",
  "mode": "local-read-only",
  "repository": {
    "alias": "current-local-repository",
    "git_repository": true,
    "branch": "symbolic-branch",
    "working_tree": "clean"
  },
  "recent_commits": [
    { "short_sha": "abcdef012345" }
  ],
  "files": [
    {
      "path": "repository-relative-allowlisted-path",
      "exists": true,
      "type": "regular-file",
      "size_bytes": 1,
      "sha256": "64-hex-characters"
    }
  ],
  "writeRequested": false,
  "writeExecuted": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

Im tatsächlichen Schema muss `outcome` bei `working_tree = clean` exakt
`accepted` und bei `working_tree = dirty` exakt `reported-neutral` sein.

Eine Ablehnung soll genau ein minimales Objekt ohne Repository-, Commit- oder
Dateimetadaten ergeben:

```json
{
  "run": "ORT-001",
  "specification_version": "v0.2.0",
  "specification_status": "draft",
  "status": "rejected",
  "mode": "local-read-only",
  "reason": "sanitized-reason",
  "writeRequested": false,
  "writeExecuted": false,
  "commitCreated": false,
  "pushExecuted": false
}
```

Die zulässigen `reason`-Werte sind vor einer Implementierung abschließend
festzulegen. Die LRT-Bezeichnung `not-a-git-repository` ist kein veröffentlichter
WF-0012-v0.1.1-Fehlercode und wird durch diesen Entwurf nicht rückwirkend zu
einem solchen.

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

## 9. Wiederholbarkeit

Bei gleicher freigegebener Eingabe und bytegenau unverändertem internem
Repository-Snapshot muss `ORT-001` ein semantisch identisches Ergebnisobjekt
bilden. Automatisch erzeugte Zeitstempel, Zufallswerte, Laufkennungen oder eine
vom Dateisystem abhängige Listenreihenfolge sind im Ergebnis unzulässig.

Die Commit-Liste ist newest-first auf höchstens drei Einträge begrenzt. Die
Dateiliste folgt der dokumentierten Allowlist. Eine spätere Serialisierung muss
eine festgelegte Feldreihenfolge verwenden, wenn ein byteidentischer
Artefaktvergleich gefordert wird.

---

## 10. Vorbedingungen

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

## 11. Abbruchbedingungen

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

## 12. Erfolgskriterien

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

## 13. Aktueller Planungsstatus

```text
Version: v0.2.0
Status: draft
Umsetzungsstatus: not-started
ORT-001 ausgeführt: nein
```

Der nächste zulässige Schritt ist ein Vertragsaudit dieses Entwurfs. Erst nach
separater Freigabe dürfen Implementierung und Tests geplant werden.
