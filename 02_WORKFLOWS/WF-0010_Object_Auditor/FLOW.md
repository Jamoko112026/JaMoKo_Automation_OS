# WF-0010 – Flow

## Version

0.1.0

## Status

draft

---

## Übersicht

WF-0010 prüft den Objektbestand des JaMoKo OS auf strukturelle Fehler, Inkonsistenzen und Abweichungen von definierten Standards.

Der Workflow erzeugt ausschließlich einen Audit Report.

WF-0010 verändert keine Objektakten, Register oder sonstigen Quelldaten.

---

## Ablauf

```text
01 – Manual Trigger
      │
      ▼
02 – Load Audit Configuration
      │
      ▼
03 – Load Object Data
      │
      ▼
04 – Load Registry Data
      │
      ▼
05 – Build Audit Context
      │
      ▼
06 – Audit Object IDs
      │
      ▼
07 – Audit Status Values
      │
      ▼
08 – Audit Registry Consistency
      │
      ▼
09 – Audit Relationships
      │
      ▼
10 – Merge Audit Findings
      │
      ▼
11 – Calculate Audit Status
      │
      ▼
12 – Audit Summary