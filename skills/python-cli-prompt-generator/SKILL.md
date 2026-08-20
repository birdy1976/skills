---
name: python-cli-prompt-generator
description: "Erzeugt aus einer kurzen Tool-Beschreibung (3–8 Sätze) direkt ein fertiges, validiertes Python-CLI-Tool: Template-Kopierworkflow, typisierte Funktionen, Tests (optional), Makefile, README."
---

# Skill: Python-CLI-Prompt-Generator

## Zweck
Aus einer kurzen Tool-Beschreibung (3–8 Sätze) erstellt dieser Skill ein fertiges, validiertes Python-3-CLI-Tool direkt im Arbeitsverzeichnis. Basis ist das Template.

---

# ANWEISUNG (ab hier verbatim)

## Vorgehen
1. Extrahiere: Toolname, Kernzweck (1–2 Sätze).
2. Template übernehmen und Token `__TOOLNAME__` in allen Dateien durch den Toolnamen ersetzen.
3. Datenmodelle (`dataclasses`, `default_factory` statt mutable Defaults) NUR wenn das Tool Zustand trägt. Sonst weglassen.
4. Funktionen mit typisierter Signatur (inkl. Rückgabetyp) definieren, auch Hilfsfunktionen. Reine Funktionen (ohne I/O) auslagern, damit sie testbar sind.
5. CLI `main(argv: list[str] | None = None) -> None` mit `argparse`: Argumente, Hilfetexte.
6. Exception → Meldung → `sys.exit(1)`-Tabelle: ausschliesslich Exceptions, die das CLI tatsächlich wirft (z. B. `FileNotFoundError`, `ValueError`). `KeyboardInterrupt` und `EOFError` (Ctrl-D bei `input()`): Leerzeile, `sys.exit(130)`.
7. Bei Datei-I/O: Format exakt festlegen.
8. Tests: genau so viele, wie das Tool wirklich braucht. `pytest-mini` (Standard): minimale, aussagekräftige Tests. Faustregel: je 1 Test pro Parser-/Fehlerfall, je 1 Test pro erwarteter CLI-Exception, plus je 1 Happy-Path-Test für Parser und Kernlogik. Komplexe Tools: umfassender. Triviales Tool: `pytest-none` (Testdatei weglassen, Validierung entsprechend kürzen).
9. Beispiel-Inputdatei anlegen, wenn das Tool eine Datei liest.

## Projektstruktur
```
./
├── {__TOOLNAME__}.py
├── test_{__TOOLNAME__}.py
├── Makefile
├── README.md
{weitere Dateien}
```

## Template

### {__TOOLNAME__}.py
```
"""Kurze Beschreibung des Tools."""

from __future__ import annotations

import argparse
import sys


def main(argv: list[str] | None = None) -> None:
    """CLI-Einstiegspunkt."""
    parser = argparse.ArgumentParser(description="Kurze Beschreibung.")
    parser.add_argument("datei", help="...")
    args = parser.parse_args(argv)

    try:
        print("TODO: Logik hier")
    except KeyboardInterrupt:
        print()
        sys.exit(130)


if __name__ == "__main__":
    main()
```

### test_{__TOOLNAME__}.py
```
from __future__ import annotations

import io
import sys
from pathlib import Path

import pytest

PROJECT_ROOT = Path(__file__).resolve().parent
sys.path.insert(0, str(PROJECT_ROOT))

import {__TOOLNAME__}  # noqa: E402


def test_placeholder() -> None:
    assert True
```

### Makefile
```
.PHONY: help
help:  ## Show this help message
        @echo "Available commands:"
        @grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "  \033[36m%-15s\033[0m %s\n", $$1, $$2}'

.DEFAULT_GOAL := help

.PHONY: validate
validate:  ## Validate codebase
        python3 -m py_compile *.py && \
        python3 -m ruff check --fix *.py && \
        python3 -m ruff format *.py && \
        python3 -m mypy *.py && \
        python3 -m pytest -q --tb=short && \
        python3 {__TOOLNAME__}.py --help
```

### README.md
```
# {__TOOLNAME__}

Kurze Beschreibung.

## Nutzung inklusive --help

```
python3 {__TOOLNAME__}.py <argumente>
```

## Entwicklung

Nur für Entwicklung: `make validate`.
```

## Coding Style
- `from __future__ import annotations`
- Type Hints überall
- Google-Docstrings: Args/Returns/Raises/Example
- dataclasses mit `default_factory`
- `with` für Dateioperationen
- PEP 8/257/484, 4 Leerzeichen Einrückung, keine Tabs
- Nur stdlib, keine externen Dependencies
- Farben als Modulkonstanten; nur ausgeben, wenn `sys.stdout.isatty()` (Farbe abschalten bei Pipes/Dateien)

## Validierung
1. Nach jeder Python-Datei sofort `python3 -m py_compile <datei>`. Fehler sofort fixen.
2. Preflight: `python3 --version`, `python3 -m ruff --version`, `python3 -m mypy --version`, `python3 -m pytest --version`. Fehlende Module: die zugehörigen `validate`-Schritte überspringen und als `skipped` melden statt abbrechen.
3. `make validate` selbst ausführen (Hinweis: `ruff check --fix` und `ruff format` schreiben Dateien um — das ist gewollt, danach erneut kompilieren).
4. Ergebnis als kompakte Status-Zeilen berichten, keine Fliesstext-Absätze. Beispiel:
   ```
   py_compile: ok
   ruff_check: ok
   ruff_format: ok
   mypy: ok
   pytest: 5 passed
   help: ok
   ```
5. Antwort kurz halten. Fertige Dateien nicht im Chat ausgeben.

## Test-Setup
- `sys.path.insert(0, str(PROJECT_ROOT))` mit `PROJECT_ROOT = Path(__file__).resolve().parent`
- `monkeypatch.setattr('sys.stdin', io.StringIO(...))` für Datei-/Stdin-Mock; bei `input()` stattdessen direkt `monkeypatch.setattr('builtins.input', ...)`
- `capsys` für Output-Prüfung
- `tmp_path`-Fixtures für Dateitests
