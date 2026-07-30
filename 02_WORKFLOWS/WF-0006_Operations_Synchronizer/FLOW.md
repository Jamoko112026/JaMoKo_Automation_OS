# WF-0006 – Operations Synchronizer

## Version

0.1.0

## Status

development

---

## Zweck

WF-0006 synchronisiert operative Arbeitsobjekte mit dem JaMoKo Operations Dashboard in Trello.

Der Workflow erkennt:

- neue Objekte
- geänderte Objekte
- unveränderte Objekte

Anschließend erstellt oder aktualisiert er die zugehörigen Trello-Karten.

---

## Ablauf

```text
Manual Trigger
      │
      ▼
01 Load Configuration
      │
      ├─────────────────────────────┐
      ▼                             ▼
02 Get Existing Trello Cards   03 Load Operations Items
      │                             │
      └──────────────┬──────────────┘
                     ▼
                04 Merge Inputs
                     │
                     ▼
               05 Normalize Data
                     │
                     ▼
              06 Compare by Object ID
                     │
          ┌──────────┼───────────┐
          ▼          ▼           ▼
       missing     changed    unchanged
          │          │           │
          ▼          ▼           ▼
07 Resolve List  09 Resolve List  No Action
          │          │
          ▼          ▼
08 Create Card  10 Update Card
          │          │
          └──────────┴───────────┐
                                 ▼
                         11 Create Sync Report
                                 │
                                 ▼
                                End