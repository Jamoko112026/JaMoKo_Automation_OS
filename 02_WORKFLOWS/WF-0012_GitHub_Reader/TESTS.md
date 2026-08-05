# WF-0012 – GitHub Reader

## Tests

Version: `v0.1.0`
Status: `passed`
Typ: `Reader`

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