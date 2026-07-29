# Workflow

## Ablauf

Manual Trigger

↓

01 – Get Existing Cards

↓

02 – Get Board Lists

↓

03 – Define Dashboard Cards

↓

04 – Compare Cards

↓

05 – Filter Missing Cards

↓

06 – Match Target List ID

↓

07 – Create Missing Card

## Node-Beschreibung

### 01 – Get Existing Cards

Liest alle offenen Karten des Zielboards über die Trello REST API.

### 02 – Get Board Lists

Liest die Listen des Zielboards über die Trello REST API.

### 03 – Define Dashboard Cards

Definiert den gewünschten Soll-Zustand des Dashboards.

### 04 – Compare Cards

Vergleicht die definierten Karten mit den vorhandenen Karten und erzeugt einen Status:

- `exists`
- `missing`
- `wrong_list`
- `duplicate`

### 05 – Filter Missing Cards

Lässt ausschließlich Karten mit dem Status `missing` weiterlaufen.

### 06 – Match Target List ID

Ordnet den definierten Ziel-Listennamen der tatsächlichen Trello-Listen-ID zu.

### 07 – Create Missing Card

Erstellt ausschließlich fehlende Karten in der richtigen Trello-Liste.

## Architekturprinzip

Der Workflow arbeitet zustandsorientiert.

Er führt nicht pauschal Schreiboperationen aus, sondern vergleicht zuerst Soll- und Ist-Zustand.

Nur eine tatsächlich fehlende Karte erzeugt eine Schreibaktion.
