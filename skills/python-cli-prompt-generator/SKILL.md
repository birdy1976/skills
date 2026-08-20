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
Du erzeugst aus einer Tool-Beschreibung EINEN vollständigen Arbeitsauftrag im Markdown-Format für einen Coding-Agenten und führst ihn anschliessend selbst aus – ohne Pause zwischen Erzeugen und Ausführen.

## Ausgabe (CRITICAL)
1. Speichere den Auftrag als `PROMPT.md` im Arbeitsverzeichnis.
2. Bestätige nur mit einem Satz, dass die Datei erstellt ist. Kein Codeblock im Chat, keine Erklärung.

## Vorgehen
1. Extrahiere: Hauptdatei-Name, Toolname, Kernzweck (1–2 Sätze).
2. Definiere Datenmodelle (`dataclasses`): typisiert, `default_factory` statt mutable Defaults.
3. Definiere **ALLE** Funktionen mit typisierter Signatur (inkl. Rückgabetyp) und Verhalten – auch Hilfsfunktionen. Jede später getestete Funktion muss hier stehen.
4. Definiere CLI (`argparse`): Argumente, Hilfetext, Tabelle Exception → Meldung → Exit-Code. **Hinweis:** Liste ausschliesslich Exceptions, die das CLI tatsächlich wirft (beispielsweise `FileNotFoundError`, `ValueError`).
5. Bei Datei-I/O: Format exakt festlegen (mit Beispiel).
6. Entwirf Tests (eine Klasse pro Bereich). Bei Zustandslogik: exakte Inputs und erwartete Werte angeben.
7. Zähle alle Tests zusammen zu `N`.
8. Wichtig: Fülle das Template unten aus. Nur `{...}` ersetzen, Struktur nicht ändern.
9. Testumfang je nach Option:
   - `pytest-none` → Test-Sektion und Pytest-Zeile in Validierung streichen (N = 0).
   - `pytest-mini` (Standard) → minimale, aussagekräftige Tests.
   - `pytest-full` → umfassende Tests.

## Vor Ausgabe prüfen (kurz)
- [ ] Summe der Tests pro Suite == `N` (oder N=0 bei `pytest-none`)
- [ ] Jede in Tests verwendete Funktion oben mit Signatur spezifiziert
- [ ] Alle Code-Fences geschlossen

---

## TEMPLATE für PROMPT.md

```markdown
# {Toolname}: {Hauptdatei} mit Tests

## Build / Test Pipeline
- Vor erstem Schreibzugriff: Arbeitsverzeichnis bestätigen (`pwd`), erst danach Dateien anlegen.
- Alle Dateien in einem Batch erstellen, wo möglich.
- Nach jeder Python-Datei sofort `python3 -m py_compile <datei>` ausführen. Fehler (auch Einrückung) sofort fixen.
- Validierung `make validate` selbst ausführen und Ergebnis als Status-Zeilen berichten. Nicht nachfragen.
- Fertige Dateien direkt speichern, nicht im Chat ausgeben. Antwort kurz halten.
- Code, Kommentare, Docstrings knapp: nur was die Regeln unten fordern.
- Validierungsergebnisse als kompakte Status-Zeilen berichten (Schritt: ok/fail), keine Fliesstext-Absätze. Beispiel:
  ```
  py_compile: ok
  ruff_check: ok
  ruff_format: ok
  mypy: ok
  pytest: {N} passed
  help: ok
  ```

## Aufgabe
{1–2 Sätze: Zweck, Nutzer, Kernverhalten}

## Projektstruktur
```
./
├── PROMPT.md
├── {Hauptdatei}
├── test_{Hauptdatei}
├── Makefile
├── README.md
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

### N+2. Coding Style
- `from __future__ import annotations`
- Type Hints überall
- Google-Docstrings: Args/Returns/Raises/Example
- dataclasses mit `default_factory`
- `with` für Dateioperationen
- PEP 8/257/484, 4 Leerzeichen Einrückung, keine Tabs
- Nur stdlib, keine externen Dependencies
- Sauberer Abbruch mit `Ctrl+C` (KeyboardInterrupt abfangen, ohne Traceback beenden)

{## Beispiel-Inputdatei (falls das Tool eine Datei liest, erstelle ein Beispiel)
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

## Makefile
`Makefile` mit:
```
.PHONY: validate
validate:  ## Validate codebase
	python3 -m py_compile *.py && \
	python3 -m ruff check --fix *.py && \
	python3 -m ruff format *.py && \
	python3 -m mypy *.py && \
	python3 -m pytest -q --tb=short && \
	python3 $(Hauptdatei) --help
```
Falls Makefile existiert: nur fehlende Sektionen ergänzen, nicht neu schreiben.

## README.md
Deutsche `README.md` mit:
- Name, kurze Beschreibung
- Nutzung: `python3 {Hauptdatei} {Beispiel-Argumente}`
- {Falls Beispiel-Inputdatei ein bestimmtes Format erwartet, hier beschreiben}
- Nur für Entwicklung: `make validate` etc.

Falls README existiert: nur fehlende Sektionen ergänzen, nicht neu schreiben.
```
