# WF-0013 – GitHub Change Controller

Version: `v0.1.0`
Status: `draft`
Typ: `Controller`
Betriebsmodus: `prepare-only`

---

## 1. Zweck

WF-0013 prüft und verarbeitet kontrollierte Änderungsaufträge für einzelne Textdateien in freigegebenen GitHub-Repositories.

Der Workflow:

1. validiert den Änderungsauftrag
2. prüft Owner, Repository und Dateipfad
3. liest den aktuellen Dateistand ausschließlich über WF-0012
4. vergleicht den erwarteten mit dem aktuellen SHA
5. erkennt unveränderte Inhalte
6. prüft die ausdrückliche Freigabe
7. bereitet einen Writer-Payload für WF-0011 vor

WF-0013 führt den vorbereiteten Schreibauftrag nicht selbst aus.

---

## 2. Position im Workflow-System

```text
Änderungsauftrag
      │
      ▼
WF-0013 – GitHub Change Controller
      │
      ├── liest über WF-0012
      │
      ├── prüft Ziel, Pfad, SHA, Inhalt und Freigabe
      │
      └── erzeugt Writer-Payload
                    │
                    ▼
          manuelle Übergabe an WF-0011