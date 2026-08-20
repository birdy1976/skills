---
name: python-cli-prompt-generator
description: "Erzeugt PROMPT.md: einen agenten-optimierten Arbeitsauftrag aus einer kurzen Tool-Beschreibung."
---

# Skill: Python-CLI-Prompt-Generator

## Zweck
Aus einer kurzen Tool-Beschreibung (3–8 Sätze) erzeugt dieser Skill einen vollständigen Arbeitsauftrag (`PROMPT.md`) für einen Coding-Agenten: Projektstruktur, Python-Regeln, Tests (optional), README, Validierungsschritte.

---

# ANWEISUNG AN DIE GENERIERENDE KI (ab hier verbatim)

## Rolle
Du erzeugst aus einer Tool-Beschreibung EINEN vollständigen Arbeitsauftrag im Markdown-Format für einen Coding-Agenten.

## Ausgabe (CRITICAL)
1. Speichere den Auftrag als `PROMPT.md` im Arbeitsverzeichnis.
2. Bestätige nur mit einem Satz, dass die Datei erstellt ist. Kein Codeblock im Chat, keine Erklärung.

## Vorgehen
1. Extrahiere: Hauptdatei-Name, Toolname, Kernzweck (1–2 Sätze).
2. Definiere Datenmodelle (dataclasses): typisiert, `default_factory` statt mutable Defaults.
3. Definiere ALLE Funktionen mit typisierter Signatur (inkl. Rückgabetyp) und Verhalten – auch Hilfsfunktionen. Jede später getestete Funktion muss hier stehen.
4. Definiere CLI (`argparse`): Argumente, Hilfetext, Tabelle Exception → Meldung → Exit-Code.
5. Bei Datei-I/O: Format exakt festlegen (mit Beispiel), ggf. Beispiel-Inputdatei verlangen.
6. Entwirf Tests (eine Klasse pro Bereich). Bei Zustandslogik: exakte Inputs und erwartete Werte angeben.
7. Zähle alle Tests zusammen zu N. N muss an beiden Stellen (Testsektion, Validierung) gleich sein.
8. Fülle das Template unten aus. Nur `{...}` ersetzen, Struktur nicht ändern.
9. Testumfang je nach Option:
   - `pytest-none`: Testsektion und Pytest-Zeile in Validierung streichen. N = 0.
   - `pytest-mini` (Standard): minimale, aussagekräftige Tests.
   - `pytest-full`: umfassende Tests.

## Vor Ausgabe prüfen
- [ ] Testsumme pro Suite = N (an beiden Stellen gleich, oder N=0)
- [ ] Jede in Tests verwendete Funktion oben spezifiziert
- [ ] Alle Signaturen vollständig typisiert
- [ ] Code-Fences alle geschlossen
- [ ] Keine Rationale/Begründungs-Abschnitte
- [ ] Format-/Sortier-/Rundungsregeln konkret, nicht "sinnvoll"
- [ ] Validierung: Agent führt selbst aus und berichtet Ergebnis
- [ ] Ausführungsregeln unverändert aus Template
- [ ] Alle Tools als `python3 -m <tool>` (nie bare `pytest`/`ruff`/`mypy`)
- [ ] Ausführungsregeln verlangen Status-Zeilen (ok/fail) statt Prosa für die Validierung

---

## TEMPLATE für PROMPT.md

```markdown
# {Toolname}: {Hauptdatei} mit Tests

## Ausführungsregeln
- Alle Dateien in einem Batch erstellen, wo möglich.
- Nach jeder Python-Datei sofort `python3 -m py_compile <datei>` ausführen. Fehler (auch Einrückung) sofort fixen.
- `python3 -m ruff check --fix {Hauptdatei} test_{Hauptdatei}` ausführen.
- `python3 -m ruff format {Hauptdatei} test_{Hauptdatei}` ausführen.
- Alle Validierungsschritte (unten) selbst ausführen und Ergebnisse berichten. Nicht nachfragen.
- Tools immer als `python3 -m <tool>` aufrufen (PATH-Probleme umgehen).
- Fertige Dateien direkt speichern, nicht im Chat ausgeben. Antwort kurz halten.
- Code, Kommentare, Docstrings knapp: nur was die Regeln unten fordern.

## Aufgabe
{1–2 Sätze: Zweck, Nutzer, Kernverhalten}

## Projektstruktur
```
./
├── README.md
├── PROMPT.md
├── {Hauptdatei}
├── test_{Hauptdatei}
{weitere Dateien}
```

## {Hauptdatei} – Anforderungen

### 1. Datenmodelle (dataclasses)
{Definitionen mit Type Hints, default_factory für mutable Defaults}

### 2–N. Funktionen
{Jede Funktion: typisierte Signatur inkl. Rückgabetyp, Verhalten, Sonderfälle, Exceptions}

### N+1. CLI `main(argv: list[str] | None = None) -> None`
- `argparse`: Argumente, Hilfetexte
- Tabelle: Exception → Meldung → `sys.exit(1)`

### N+2. Python-Regeln
- `from __future__ import annotations`
- Type Hints überall
- Google-Docstrings: Args/Returns/Raises/Example
- dataclasses mit `default_factory`
- `with` für Dateioperationen
- PEP 8/257/484, 4 Leerzeichen Einrückung, keine Tabs
- `python3` zum Ausführen verwenden
- Nur stdlib, keine externen Dependencies
- Sauberer Abbruch mit `Ctrl+C` (KeyboardInterrupt abfangen, ohne Traceback beenden)

{## Beispiel-Inputdatei (falls relevant)
- Format-Spezifikation
- Konkretes Beispiel}

## test_{Hauptdatei} – Tests (pytest, genau {N} Tests)

### Test{Bereich1} ({n1} Tests)
1. `test_...` – {kurz; bei Zustandslogik: exakte Inputs/Erwartungswerte}
...

### Test{BereichK} ({nk} Tests)
...

### Test-Setup
- `sys.path.insert(0, str(PROJECT_ROOT))`
- `monkeypatch.setattr('sys.stdin', io.StringIO(...))` für I/O-Mock
- `tmp_path`-Fixtures für Dateitests

## README.md
Deutsche `README.md` mit:
- Name, kurze Beschreibung
- Installation: keine (stdlib; pytest vorhanden)
- Nutzung: `python3 {Hauptdatei} {Beispiel-Argumente}`
- Test: `python3 -m pytest -q`
- {ggf. Datenformat kurz erklären}

Falls README existiert: nur fehlende Sektionen ergänzen, nicht neu schreiben.

## Validierung (selbst ausführen, Ergebnis als Status-Zeilen berichten)
1. `python3 -m py_compile {Hauptdatei}` → ohne Fehler
2. `python3 -m ruff check --fix {Hauptdatei} test_{Hauptdatei}` → ohne Fehler
3. `python3 -m ruff format {Hauptdatei} test_{Hauptdatei}` → Formatierung fixen
4. `python3 -m mypy --non-interactive {Hauptdatei}` → ohne Fehler
5. `python3 -m pytest -q --tb=short` → zeigt `"{N} passed"`, keine Warnings
6. `python3 {Hauptdatei} --help` → sinnvolle Usage-Meldung
```
