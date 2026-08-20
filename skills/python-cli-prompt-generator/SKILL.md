---
name: python-cli-prompt-generator
description: "From a short tool description (3-8 sentences), directly builds a finished, validated Python CLI tool: template-copy workflow, typed functions, tests (optional), Makefile, README."
---

# Skill: Python-CLI-Prompt-Generator

## Purpose
From a short tool description (3-8 sentences), this skill builds a finished, validated Python 3 CLI tool directly in the working directory. Based on the template below.

---

# INSTRUCTION (verbatim from here)

## Procedure
1. Extract: tool name, core purpose (1-2 sentences).
2. Copy the template and replace the `__TOOLNAME__` token in every file with the tool name.
3. Data models (`dataclasses`, `default_factory` instead of mutable defaults) ONLY if the tool carries state. Otherwise omit.
4. Before writing anything: go through all type names and modules the code will need, once, completely (including helper functions), then write the file in one pass — do not add imports one at a time as errors surface.
5. Define functions with typed signatures (incl. return type), including helper functions. Extract pure functions (no I/O) so they stay testable.
6. CLI `main(argv: list[str] | None = None) -> None` with `argparse`: arguments, help text.
7. Exception → message → `sys.exit(1)` table: list only exceptions the CLI actually raises (e.g. `FileNotFoundError`, `ValueError`). `KeyboardInterrupt` and `EOFError` (Ctrl-D at `input()`): blank line, `sys.exit(130)` — **build this in from the start** (see template), don't wrap it around an existing loop afterward.
8. For file I/O: define the format exactly.
9. Tests: exactly as many as the tool actually needs. `pytest-mini` (default): minimal, meaningful tests. Rule of thumb: 1 test per parser/error case, 1 test per expected CLI exception, plus 1 happy-path test each for parser and core logic. Complex tools: more thorough. Trivial tool: `pytest-none` (skip the test file, shorten validation accordingly).
10. Create a sample input file if the tool reads one.
11. When structurally reshaping existing code (e.g. changing control flow, wrapping a block around existing logic): rewrite the affected file completely, don't patch it piecemeal — piecemeal patches to indentation/control flow are error-prone.
12. Proceed to the validation and fix the issues.

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
"""Short description of the tool."""

from __future__ import annotations

import argparse
import sys


def main(argv: list[str] | None = None) -> None:
    """CLI entry point."""
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

### Makefile (recipe lines MUST use TAB characters — the code block below uses literal tabs, keep them when copying)
```
.PHONY: help
help: ## Show this help message
	@echo "Available commands:"
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "  \033[36m%-15s\033[0m %s\n", $$1, $$2}'

.DEFAULT_GOAL := help

.PHONY: validate
validate: ## Validate codebase — single shell line: make runs each recipe line in its own shell, so `&&` can NOT span lines
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

## Coding Style
- `from __future__ import annotations`
- Type hints everywhere. Comment only when explicitly requested (code AND other files).
- Generics as built-in types (`dict[str, str]`, `list[Question]`) — do NOT import `typing.Dict`/`typing.List`; `from __future__ import annotations` makes that unnecessary.
- No single-letter variable names (`l`, `O`, `I`) — use descriptive names (`line`, `item`, `idx`). Else, there will be Ruff errors.
- All imports in one block at the top. Else, there will be Ruff errors.
- Google-style docstrings: Args/Returns/Raises/Example
- dataclasses with `default_factory`
- `with` statements for file operations
- PEP 8/257/484, 4-space indentation, no tabs
- stdlib only, no external dependencies
- Colors as module constants; only emit when `sys.stdout.isatty()` (disable color for pipes/files)

## Validation
1. Immediately after each Python file: `python3 -m py_compile <file>`. Fix errors right away.
2. Preflight: `python3 --version`, `python3 -m ruff --version`, `python3 -m mypy --version`, `python3 -m pytest --version`. Missing modules: skip the corresponding `validate` steps and report them as `skipped` instead of aborting.
3. Run `make validate` yourself (note: `ruff check --fix` and `ruff format` rewrite files — that's intended, recompile afterward).
4. Report the result as compact status lines, no prose paragraphs. Example:
   ```
   py_compile: ok
   ruff_check: ok
   ruff_format: ok
   mypy: ok
   pytest: 5 passed
   demo: ok
   help: ok
   ```
5. Keep the response short. Don't print finished files into the chat.

## Test Setup
- `sys.path.insert(0, str(PROJECT_ROOT))` with `PROJECT_ROOT = Path(__file__).resolve().parent`
- `monkeypatch.setattr('sys.stdin', io.StringIO(...))` for file/stdin mocking; for `input()` use `monkeypatch.setattr('builtins.input', ...)` directly instead
- `capsys` for output assertions
- `tmp_path` fixtures for file tests
