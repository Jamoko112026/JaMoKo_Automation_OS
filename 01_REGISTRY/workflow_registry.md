# Workflow Registry

Dieses Register verwaltet alle Workflows des JaMoKo Automation OS.

Es dokumentiert den aktuellen Entwicklungsstand, die Versionierung, die technischen Abhängigkeiten sowie die Einordnung in Release- und Entwicklungsblöcke.

Die Vergabe neuer Workflow-IDs erfolgt ausschließlich über `id_registry.md`.

---

# Statuswerte

- `draft`
- `development`
- `testing`
- `released`
- `deprecated`
- `archived`

---

# Registrierte Workflows

| ID | Name | Typ | Status | Version | Workflow-Akte |
|----|------|-----|----------|---------|----------------|
| WF-0001 | Task Intake | Task | released | v1.0.0 | `02_WORKFLOWS/WF-0001_Task_Intake/README.md` |
| WF-0002 | Trello Board Setup | Setup | released | v1.0.0 | `02_WORKFLOWS/WF-0002_Trello_Board_Setup/README.md` |
| WF-0003 | Dashboard Initial Cards | Setup | released | v1.0.0 | `02_WORKFLOWS/WF-0003_Dashboard_Initial_Cards/README.md` |
| WF-0004 | Dashboard Sync | Synchronization | released | v1.0.0 | `02_WORKFLOWS/WF-0004_Dashboard_Sync/README.md` |
| WF-0005 | Dashboard Maintenance | Audit | released | v1.0.0 | `02_WORKFLOWS/WF-0005_Dashboard_Maintenance/README.md` |
| WF-0006 | Operations Synchronizer | Synchronization | released | v1.0.0 | `02_WORKFLOWS/WF-0006_Operations_Synchronizer/README.md` |
| WF-0008 | Object Loader | Loader | released | v0.2.0 | `02_WORKFLOWS/WF-0008_Object_Loader/README.md` |
| WF-0009 | Object Repair Engine | Processor | released | v0.1.0 | `02_WORKFLOWS/WF-0009_Object_Repair_Engine/README.md` |
| WF-0010 | Object Auditor | Validator | released | v0.1.0 | `02_WORKFLOWS/WF-0010_Object_Auditor/README.md` |
| WF-0011 | GitHub Writer | Writer | released | v0.1.0 | `02_WORKFLOWS/WF-0011_GitHub_Writer/README.md` |

---

# Reservierte Workflow-IDs

| ID | Geplanter Workflow | Status |
|----|--------------------|--------|
| WF-0007 | Project Sync | reserved |

---

# Technische Abhängigkeiten

| Workflow | Abhängigkeiten |
|----------|----------------|
| WF-0001 | Trello API |
| WF-0002 | Trello API |
| WF-0003 | WF-0002, Trello API |
| WF-0004 | WF-0002, Trello API |
| WF-0005 | WF-0002, Trello API |
| WF-0006 | Trello API, n8n |
| WF-0008 | GitHub API, n8n |
| WF-0009 | WF-0008, GitHub API, n8n |
| WF-0010 | WF-0008, WF-0009, n8n |
| WF-0011 | WF-0010, n8n |

---

# Release-Blöcke

## Dashboard Automation v1.0.0

Enthalten:

- WF-0002 – Trello Board Setup
- WF-0003 – Dashboard Initial Cards
- WF-0004 – Dashboard Sync
- WF-0005 – Dashboard Maintenance
- WF-0006 – Operations Synchronizer

Status:

`released`

---

## Object Quality Pipeline v1.0

Enthalten:

- WF-0008 – Object Loader
- WF-0009 – Object Repair Engine
- WF-0010 – Object Auditor

Status:

`released`

---

# Aktiver Entwicklungsblock

## Repository Automation v0.1.0

Enthalten:

- WF-0011 – GitHub Writer

Status:

`released`

Hinweis:

Version 0.1.0 ist eine deaktivierte Simulation ohne schreibenden Zugriff auf GitHub.

---

# Hinweise

- Jeder Workflow besitzt eine vollständige Workflow-Akte unter `02_WORKFLOWS/`.
- Veröffentlichte Versionen werden zusätzlich als Git-Tag versioniert.
- Änderungen an Workflow-IDs erfolgen ausschließlich über das `id_registry.md`.
- Architekturentscheidungen werden über die entsprechenden DEC-Dokumente dokumentiert.

---

# Letzte Aktualisierung

2026-08-02
