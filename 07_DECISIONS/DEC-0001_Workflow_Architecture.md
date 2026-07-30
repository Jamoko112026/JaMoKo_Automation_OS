# DEC-0001 – Workflow Architecture

## Status

accepted

## Datum

2026-07-30

---

## Entscheidung

Alle produktiven Automationen werden als eigenständige Workflows mit einer eindeutigen Workflow-ID geführt.

Jeder Workflow besitzt:

- Workflow-Akte
- Version
- Changelog
- Dokumentation
- Tests
- Exportdatei

---

## Begründung

Dadurch werden Workflows:

- nachvollziehbar
- wartbar
- reproduzierbar
- versionierbar
- unabhängig voneinander entwickelbar

---

## Konsequenzen

Neue Workflows erhalten:

- neue Workflow-ID
- Eintrag im Workflow Registry
- Dokumentation nach STD-0001

---

## Betroffene Standards

- STD-0001