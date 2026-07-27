# WF-0001 – Workflow

## Zweck

Empfängt eine Aufgabe über einen Webhook und erstellt daraus automatisch eine Trello-Karte.

## Version

0.1.0

## Trigger

Webhook (POST)

## Ablauf

```text
Webhook
    │
    ▼
Edit Fields
    │
    ▼
Create Trello Card
```

## Eingabe

Siehe `contract.md`

## Ausgabe

Neue Trello-Karte

## Verwendete Services

- n8n
- Trello