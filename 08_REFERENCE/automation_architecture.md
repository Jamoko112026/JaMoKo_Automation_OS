# JaMoKo Automation Architecture

## Version

v1.0

---

# Überblick

Das JaMoKo Automation OS verbindet Geschäftsprozesse,
Dokumentation und Automatisierung.

```
                Benutzer
                    │
                    ▼
              Trello Dashboard
                    │
                    ▼
              n8n Workflows
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
   JaMoKo OS              Externe Dienste
        │                       │
        ▼                       ▼
   Kundenobjekte          Trello API
   Projekte               GitHub API
   Standards              E-Mail
   Entscheidungen         CRM
        │
        ▼
   Dokumentation
```

---

# Ebenen

## Ebene 1

Benutzer

- startet Prozesse
- pflegt Inhalte

---

## Ebene 2

Trello

Visualisierung

---

## Ebene 3

n8n

Automatisierung

---

## Ebene 4

JaMoKo Automation OS

Dokumentation

Versionierung

Workflowbibliothek

Standards

---

## Ebene 5

JaMoKo OS

Objekte

Beziehungen

Entscheidungen

Historie

---

# Architekturprinzipien

- Dokumentation vor Automatisierung
- Standards vor Individualisierung
- Wiederholbarkeit
- Nachvollziehbarkeit
- Versionierung
- Community First

---

# Workflow-Lebenszyklus

Idee

↓

Entwicklung

↓

Test

↓

Released

↓

Produktiv

↓

Wartung

↓

Archiv

---

# Aktueller Stand

Workflowbibliothek:

- WF-0001
- WF-0002
- WF-0003
- WF-0004
- WF-0005

Standards:

- STD-0001

Version:

Automation OS v0.1