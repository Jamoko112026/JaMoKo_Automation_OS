# Workflow Registry

Dieses Register verwaltet alle Workflows des JaMoKo Automation OS.

Die Vergabe neuer Workflow-IDs erfolgt über `id_registry.md`.

---

## Statuswerte

- `draft`
- `development`
- `testing`
- `released`
- `deprecated`
- `archived`

---

## Workflows

| ID | Name | Typ | Status | Version | Workflow-Akte |
|----|------|-----|--------|---------|----------------|
| WF-0001 | Task Intake | Task | released | v1.0.0 | `02_WORKFLOWS/WF-0001_Task_Intake/README.md` |
| WF-0002 | Trello Board Setup | Setup | released | v1.0.0 | `02_WORKFLOWS/WF-0002_Trello_Board_Setup/README.md` |
| WF-0003 | Dashboard Initial Cards | Setup | released | v1.0.0 | `02_WORKFLOWS/WF-0003_Dashboard_Initial_Cards/README.md` |
| WF-0004 | Dashboard Sync | Synchronization | released | v1.0.0 | `02_WORKFLOWS/WF-0004_Dashboard_Sync/README.md` |
| WF-0005 | Dashboard Maintenance | Audit | released | v1.0.0 | `02_WORKFLOWS/WF-0005_Dashboard_Maintenance/README.md` |
| WF-0006 | Operations Synchronizer | Synchronization | released | v1.0.0 | `02_WORKFLOWS/WF-0006_Operations_Synchronizer/README.md` |
| WF-0008 | Object Loader | Loader | development | v0.2.0 | `02_WORKFLOWS/WF-0008_Object_Loader/README.md` |
| WF-0009 | Object Repair Engine | Processor | released | v0.1.0 | `02_WORKFLOWS/WF-0009_Object_Repair_Engine/README.md` |
| WF-0010 | Object Auditor | Validator | draft | v0.1.0 | `02_WORKFLOWS/WF-0010_Object_Auditor/README.md` |

---

## Reservierte IDs

| ID | Geplanter Workflow | Status |
|----|--------------------|--------|
| WF-0007 | Project Sync | reserved |
| WF-0011 | GitHub Writer | reserved |

---

## Abhängigkeiten

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

---

## Aktueller Release-Block

### Dashboard Automation v1.0.0

Enthalten:

- WF-0002 – Trello Board Setup
- WF-0003 – Dashboard Initial Cards
- WF-0004 – Dashboard Sync
- WF-0005 – Dashboard Maintenance
- WF-0006 – Operations Synchronizer

Status:

`released`

---

## Aktueller Entwicklungsblock

### Object Quality Pipeline

Enthalten:

- WF-0008 – Object Loader
- WF-0009 – Object Repair Engine
- WF-0010 – Object Auditor
- WF-0011 – GitHub Writer

Status:

`development`

---

## Letzte Aktualisierung

2026-08-02