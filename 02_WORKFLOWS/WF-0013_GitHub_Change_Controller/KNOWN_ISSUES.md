# WF-0013 – Known Issues

Version: `v0.1.0`
Status: `draft`
Typ: `Controller`
Betriebsmodus: `prepare-only`

---

## 1. Zweck

Diese Datei dokumentiert bekannte Einschränkungen, offene Entscheidungen und technische Risiken von WF-0013.

Einträge werden nicht gelöscht. Erledigte Punkte erhalten den Status `resolved` und bleiben zur Nachvollziehbarkeit bestehen.

---

## 2. Statuswerte

- `open`
- `in-review`
- `resolved`
- `accepted`
- `deferred`

Prioritäten:

- `critical`
- `high`
- `medium`
- `low`

---

## 3. Offene Punkte

### KI-001 – Inhaltsgrößenlimit noch nicht festgelegt

**Status:** `open`
**Priorität:** `high`
**Bereich:** Validierung / Ressourcenbegrenzung

**Beschreibung:**

Für `proposed_content` wurde noch keine verbindliche maximale Größe definiert.

Ohne Größenlimit können sehr große Eingaben:

- unnötig Speicher beanspruchen
- n8n-Ausführungen verlangsamen
- Log- und Payload-Grenzen überschreiten
- nachgelagerte Workflows belasten

**Erforderliche Entscheidung:**

Vor der Freigabe von `v0.1.0` muss eine maximale Inhaltsgröße in Bytes festgelegt und in Validator, Spezifikation und Tests übernommen werden.

**Abnahmerelevanz:**

Blockiert T-024 und die Freigabe.

---

### KI-002 – Allowlist technisch noch nicht implementiert

**Status:** `open`
**Priorität:** `high`
**Bereich:** Target Guard

**Beschreibung:**

Die zulässige Kombination aus Owner und Repository ist dokumentiert, aber noch nicht technisch im Workflow umgesetzt.

**Risiko:**

Ein fehlerhaft konfigurierter Controller könnte Leseaufträge für nicht vorgesehene Repositories vorbereiten.

**Erforderliche Maßnahme:**

Eine explizite Allowlist als unveränderliche Workflow-Konfiguration implementieren. Owner und Repository müssen als Kombination geprüft werden.

**Abnahmerelevanz:**

Blockiert T-005, T-006 und S-004.

---

### KI-003 – Vertrag mit WF-0012 noch nicht technisch geprüft

**Status:** `open`
**Priorität:** `high`
**Bereich:** Reader-Integration

**Beschreibung:**

Die erwarteten Antwortfelder von WF-0012 sind dokumentiert, aber noch nicht gegen dessen tatsächlichen n8n-Export getestet.

Zu prüfen sind insbesondere:

- Erfolgsstatus
- normalisiertes Ziel
- SHA-Feld
- Inhaltsfeld
- Fehlerformat
- Verhalten bei leeren Ergebnissen

**Risiko:**

Abweichende Feldnamen oder Datentypen können zu falschen Ablehnungen oder unvollständigen Vergleichen führen.

**Erforderliche Maßnahme:**

Schemaabgleich und Integrationstest mit dem freigegebenen Stand von WF-0012 durchführen.

**Abnahmerelevanz:**

Blockiert T-010, T-011, T-012 und S-003.

---

### KI-004 – Writer-Payload noch nicht gegen WF-0011 abgeglichen

**Status:** `open`
**Priorität:** `high`
**Bereich:** Writer-Schnittstelle

**Beschreibung:**

Der vorbereitete Writer-Payload wurde konzeptionell definiert, aber noch nicht gegen das verbindliche Eingabeschema von WF-0011 geprüft.

Zu prüfen sind:

- Feldname für Branch oder Ref
- Feldname für SHA
- Inhaltsformat
- Commit-Nachricht
- erlaubte Zusatzfelder
- Pflichtfelder

**Risiko:**

Ein korrekt vorbereiteter Auftrag könnte später von WF-0011 abgelehnt oder falsch interpretiert werden.

**Erforderliche Maßnahme:**

Schemavergleich durchführen, ohne WF-0011 auszuführen.

**Abnahmerelevanz:**

Blockiert T-016 und die spätere kontrollierte Übergabe.

---

### KI-005 – Pfadnormalisierung noch nicht implementiert

**Status:** `open`
**Priorität:** `high`
**Bereich:** Pfadsicherheit

**Beschreibung:**

Die Regeln für relative Pfade, Traversal-Schutz, Backslashes, kodierte Sequenzen und Kontrollzeichen sind dokumentiert, aber noch nicht technisch umgesetzt.

**Risiko:**

Unterschiedliche Pfaddarstellungen könnten Schutzprüfungen umgehen oder zu uneindeutigen Zielen führen.

**Erforderliche Maßnahme:**

Eine deterministische Normalisierungs- und Validierungsfunktion implementieren. Dekodierung und Prüfung müssen in festgelegter Reihenfolge erfolgen.

**Abnahmerelevanz:**

Blockiert T-007 bis T-009 und S-004.

---

### KI-006 – Inhaltsvergleich ist derzeit nur als exakter Vergleich definiert

**Status:** `open`
**Priorität:** `medium`
**Bereich:** State Comparator

**Beschreibung:**

Der Vergleich lautet aktuell:

```text
current_content != proposed_content