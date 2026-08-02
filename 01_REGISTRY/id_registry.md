# ID Registry

Dieses Register verwaltet alle vergebenen Workflow-IDs im JaMoKo Automation OS.

Die Vergabe erfolgt fortlaufend. Bereits vergebene IDs werden niemals erneut verwendet.

---

## Vergebene IDs

| ID | Typ | Name | Status |
|----|-----|------|--------|
| WF-0001 | Workflow | Task Intake | released |
| WF-0002 | Workflow | Trello Board Setup | released |
| WF-0003 | Workflow | Dashboard Initial Cards | released |
| WF-0004 | Workflow | Dashboard Sync | released |
| WF-0005 | Workflow | Dashboard Maintenance | released |
| WF-0006 | Workflow | Operations Synchronizer | released |
| WF-0008 | Workflow | Object Loader | development |
| WF-0009 | Workflow | Object Repair Engine | released |
| WF-0010 | Workflow | Object Auditor | draft |

---

## Reservierte IDs

| ID | Geplanter Workflow | Status |
|----|--------------------|--------|
| WF-0007 | Project Sync | reserved |
| WF-0011 | GitHub Writer | reserved |

---

## Nächste freie ID

```text
WF-0012
```

---

## Regeln

- Jede Workflow-ID wird genau einmal vergeben.
- Vergebene IDs werden niemals erneut verwendet.
- Reservierte IDs dürfen nur durch eine dokumentierte Entscheidung geändert werden.
- Neue Workflows werden gleichzeitig im `workflow_registry.md` und im `id_registry.md` eingetragen.
- Jeder veröffentlichte Workflow erhält eine Workflow-Akte unter `02_WORKFLOWS/`.

---

## Letzte Aktualisierung

2026-08-02