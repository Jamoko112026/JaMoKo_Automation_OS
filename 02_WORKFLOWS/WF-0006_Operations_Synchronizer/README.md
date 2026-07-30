# WF-0006 – Operations Synchronizer

## Status

development

## Version

0.1.0

## Typ

Synchronization Workflow

## Zweck

WF-0006 hält das JaMoKo Operations Dashboard in Trello aktuell.

Der Workflow sammelt relevante Aufgaben und Arbeitsobjekte aus den angebundenen Systemen, prüft deren vorhandenen Trello-Status und erstellt oder aktualisiert die benötigten Karten.

## Ziel

Alles, woran JaMoKo aktuell arbeitet, soll im Operations Dashboard sichtbar sein.

## Erste Ausbaustufe

Die erste Version synchronisiert zunächst eine zentrale Aufgabenquelle mit Trello.

Sie soll:

- bestehende Trello-Karten einlesen
- definierte Arbeitskarten einlesen
- Karten anhand einer stabilen ID vergleichen
- fehlende Karten erstellen
- bestehende Karten aktualisieren
- doppelte Karten verhindern
- einen Synchronisationsbericht erzeugen

## Spätere Datenquellen

- JaMoKo OS
- Automation OS
- Kundenobjekte
- Projekte
- persönliche Verwaltung
- GitHub
- Kalender
- E-Mail

## Zielsystem

Trello – JaMoKo Operations Dashboard

## Verantwortlicher Standard

- STD-0001 – Workflow Documentation Standard

## Abhängigkeiten

- WF-0004 – Dashboard Sync
- WF-0005 – Dashboard Maintenance
- Trello API
- n8n
