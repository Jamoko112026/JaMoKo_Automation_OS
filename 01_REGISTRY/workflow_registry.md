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

---

## Reservierte IDs

| ID | Geplanter Workflow | Status |
|----|--------------------|--------|
| WF-0006 | Customer Sync | reserved |
| WF-0007 | Project Sync | reserved |
| WF-0008 | Object Loader | reserved |
| WF-0009 | GitHub Sync | reserved |
| WF-0010 | Daily Review | reserved |

---

## Abhängigkeiten

| Workflow | Abhängigkeiten |
|----------|----------------|
| WF-0001 | Trello API |
| WF-0002 | Trello API |
| WF-0003 | WF-0002, Trello API |
| WF-0004 | WF-0002, Trello API |
| WF-0005 | WF-0002, Trello API |

---

## Aktueller Release-Block

### Dashboard Automation v1.0.0

Enthalten:

- WF-0002 – Trello Board Setup
- WF-0003 – Dashboard Initial Cards
- WF-0004 – Dashboard Sync
- WF-0005 – Dashboard Maintenance

Status:

`released`

---

## Letzte Aktualisierung

2026-07-30
