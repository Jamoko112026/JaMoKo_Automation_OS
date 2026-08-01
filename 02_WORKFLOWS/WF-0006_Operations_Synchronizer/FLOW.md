# WF-0006 – Flow

## Version

1.0.0

## Status

stable

## Zweck

Diese Datei beschreibt den tatsächlichen Ablauf von WF-0006 – Operations Synchronizer.

## Ablauf

```text
Manual Trigger
      │
      ▼
01 – Load Configuration
      │
      ▼
02 – Load Operations Items
      │
      ├──────────────────────────────────┐
      ▼                                  ▼
03 – Build Operations Index     03 – Get Existing Trello Cards
                                         │
                                         ▼
                                05 – Build Trello Index
      │                                  │
      └────────────────┬─────────────────┘
                       ▼
                     Merge
                       │
                       ▼
              07 – Compare Objects
                 ┌─────┼─────┐
                 ▼     ▼     ▼
              create update none/error
                 │     │
                 │     ▼
                 │ 15 – Filter Update
                 │     │
                 │     ▼
                 │ 16 – Update Trello Card
                 │     │
                 │     ▼
                 │ 18 – Log Update Result
                 │
                 ▼
          08 – Filter Create
                 │
                 ▼
        09 – Resolve Target List
                 │
                 ▼
       12 – Merge Target and Lists
                 │
                 ▼
       13 – Resolve Target List ID
                 │
                 ▼
         14 – Create Trello Card
                 │
                 ▼
          17 – Log Create Result

              19 – Summary