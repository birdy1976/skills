---
name: python-cli-prompt-generator
description: "From a short tool description (3-8 sentences), directly builds a finished, validated Python CLI tool: template-copy workflow, typed functions, tests (optional), Makefile, README."
---

# Skill: Python-CLI-Prompt-Generator

## Purpose
From a short tool description (3-8 sentences), this skill builds a finished, validated Python 3 CLI tool directly in the working directory. Based on the template below.

---

# INSTRUCTION (verbatim from here)

## Coding Style
- `from __future__ import annotations`
- Type hints everywhere. IMPORTANT: NO docstrings/comments EVER in ANY of the files.
- Generics as built-in types (`dict[str, str]`, `list[Question]`) — do NOT import `typing.Dict`/`typing.List`; `from __future__ import annotations` makes that unnecessary.
- No single-letter variable names (`l`, `O`, `I`) — use descriptive names (`line`, `item`, `idx`). Else, there will be Ruff errors.
- All imports in one block at the top. Else, there will be Ruff errors.
- dataclasses with `default_factory`
- `with` statements for file operations
- PEP 8/257/484, 4-space indentation, no tabs
- stdlib only, no external dependencies
- Colors as module constants; only emit when `sys.stdout.isatty()` (disable color for pipes/files)

## Validation
1. Run `make validate` yourself (note: `ruff check --fix` and `ruff format` rewrite files — that's intended, recompile afterward).
2. Report the result as compact status lines, no prose paragraphs. Example:
   ```
   py_compile: ok
   ruff_check: ok
   ruff_format: ok
   mypy: ok
   pytest: 5 passed
   ```
3. Keep the response short. Don't print finished files into the chat.

## Test Setup
- Use `sys.path.insert(0, str(PROJECT_ROOT))` with `PROJECT_ROOT = Path(__file__).resolve().parent`.
- Mock file I/O with `monkeypatch.setattr('sys.stdin', io.StringIO(...))`; for `input()` use `monkeypatch.setattr('builtins.input', ...)`.
- Capture output with `capsys`, and write at least one test per public API or error case.

## Project Structure
```
./
├── {__TOOLNAME__}.py
├── test_{__TOOLNAME__}.py
├── Makefile
├── README.md
{additional files}
```

## Template

### {__TOOLNAME__}.py
```
from __future__ import annotations

import argparse
import sys


def main(argv: list[str] | None = None) -> None:
    parser = argparse.ArgumentParser(description="Short description.")
    parser.add_argument("file", help="...")
    args = parser.parse_args(argv)

    try:
        print("TODO: logic here")
    except (KeyboardInterrupt, EOFError):
        print()
        sys.exit(130)


if __name__ == "__main__":
    main()
```
Any loop containing `input()` belongs inside this `try/except` block from the start — don't wrap it around later.

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
- Recipe lines MUST use TAB characters — the code block below uses literal tabs, keep them when copying.
- If the tool does not accept an extra text file argument, remove the smoke test from the `validate` recipe.
```
.PHONY: help
help: ## Show this help message
	@echo "Available commands:"
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "  \033[36m%-15s\033[0m %s\n", $$1, $$2}'

.DEFAULT_GOAL := help

.PHONY: validate
validate: ## Validate codebase
	python3 -m py_compile *.py && python3 -m ruff check --fix *.py && python3 -m ruff format *.py && python3 -m mypy *.py && python3 -m pytest -q --tb=short && python3 {__TOOLNAME__}.py --help && (python3 {__TOOLNAME__}.py {__TOOLNAME__}.txt </dev/null || true)
```

### README.md
```
# {__TOOLNAME__}

Short description.

## Usage including --help

```
python3 {__TOOLNAME__}.py <arguments>
```

## Development

Development only: `make validate`.
```

## Procedure
1. Extract: tool name, core purpose (1-2 sentences).
2. Copy the template and replace the `__TOOLNAME__` token in every file with the tool name.
3. Data models (`dataclasses`, `default_factory` instead of mutable defaults) ONLY if the tool carries state. Otherwise omit.
4. Before writing anything: go through all type names and modules the code will need, once, completely (including helper functions), then write the file in one pass — do not add imports one at a time as errors surface.
5. Define functions with typed signatures (incl. return type), including helper functions. Extract pure functions (no I/O) so they stay testable.
6. CLI `main(argv: list[str] | None = None) -> None` with `argparse`: arguments, help text.
7. Exception → message → `sys.exit(1)` table: list only exceptions the CLI actually raises (e.g. `FileNotFoundError`, `ValueError`). `KeyboardInterrupt` and `EOFError` (Ctrl-D at `input()`): blank line, `sys.exit(130)` — **build this in from the start** (see template), don't wrap it around an existing loop afterward.
8. For file I/O: define the format exactly.
9. Tests: write at least one meaningful test per public API or error case. A minimal set includes a parser test, an exception test, and a happy‑path test for core logic.
10. Create a sample input file if the tool reads one.
11. When structurally reshaping existing code (e.g. changing control flow, wrapping a block around existing logic): rewrite the affected file completely, don't patch it piecemeal — piecemeal patches to indentation/control flow are error-prone.
12. Proceed to the validation and fix the issues.
