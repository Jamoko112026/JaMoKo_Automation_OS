# ID Registry

Dieses Register verwaltet alle vergebenen Workflow-IDs des JaMoKo Automation OS.

Workflow-IDs werden fortlaufend vergeben und sind dauerhaft eindeutig. Eine einmal vergebene ID wird niemals erneut verwendet.

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
| WF-0008 | Workflow | Object Loader | released |
| WF-0009 | Workflow | Object Repair Engine | released |
| WF-0010 | Workflow | Object Auditor | released |

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

## Vergaberegeln

- Jede Workflow-ID wird genau einmal vergeben.
- Vergebene IDs werden niemals erneut verwendet.
- Reservierte IDs dürfen ausschließlich durch eine dokumentierte Entscheidung geändert oder freigegeben werden.
- Jeder neue Workflow wird gleichzeitig im `id_registry.md` und im `workflow_registry.md` eingetragen.
- Jeder veröffentlichte Workflow erhält eine vollständige Workflow-Akte unter `02_WORKFLOWS/`.
- Veröffentlichte Workflow-Versionen werden zusätzlich über Git-Tags versioniert.

---

## Versionierte Releases

| Workflow | Aktuelle Version |
|----------|------------------|
| WF-0008 – Object Loader | v0.2.0 |
| WF-0009 – Object Repair Engine | v0.1.0 |
| WF-0010 – Object Auditor | v0.1.0 |

---

## Letzte Aktualisierung

2026-08-02