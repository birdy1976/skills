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
9. Tests: `pytest-mini` (default) — use this concrete checklist instead of "as many as needed":
   - 1 happy-path test for the parser (fixture string → expected objects)
   - 1 test per parser error branch (missing ANSWER line, no options, missing prompt, out-of-range value, empty input)
   - 1 happy-path test for the core logic (full run, all-correct inputs)
   - 1 test per expected CLI exception (`FileNotFoundError`/`ValueError` → exit 1; EOF/`KeyboardInterrupt` → exit 130, asserted via `pytest.raises(SystemExit)` + `excinfo.value.code`)
   - 1 test per pure helper with branches (validation, color gating on/off, shuffle, formatting)
   Complex tools: more thorough (round trips, edge cases). Trivial tool: `pytest-none` (skip the test file, shorten validation accordingly).
10. Create a sample input file if the tool reads one — convention: name it `{__TOOLNAME__}.txt` so the `demo` target and the smoke run at the end of `validate` work out of the box.
11. When structurally reshaping existing code (e.g. changing control flow, wrapping a block around existing logic): rewrite the affected file completely, don't patch it piecemeal — piecemeal patches to indentation/control flow are error-prone.
12. Proceed to the validation and fix the issues.

## Project Structure
```
./
├── {__TOOLNAME__}.py
├── test_{__TOOLNAME__}.py
├── {__TOOLNAME__}.txt   (sample input, only if the tool reads a file)
├── Makefile
├── ruff.toml
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
        # TODO: logic here — any loop containing input() stays inside this try/except
        with open(args.file, encoding="utf-8") as handle:
            text = handle.read()
        # TODO: parse `text`, then run the interactive loop
    except FileNotFoundError as error:
        print(f"{__TOOLNAME__}.py: error: {error}", file=sys.stderr)
        sys.exit(1)
    except ValueError as error:
        print(f"{__TOOLNAME__}.py: error: {error}", file=sys.stderr)
        sys.exit(1)
    except (KeyboardInterrupt, EOFError):
        print()
        sys.exit(130)


if __name__ == "__main__":
    main()
```
Exception → message → `sys.exit(1)` scaffold: only exceptions the CLI actually raises (`FileNotFoundError`, `ValueError`); add more only if the logic introduces them. Any loop containing `input()` belongs inside this `try/except` block from the start — don't wrap it around later.

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

.PHONY: demo
demo: ## Smoke-run the tool against the sample file (EOF input ends it gracefully with exit 130)
	python3 {__TOOLNAME__}.py {__TOOLNAME__}.txt </dev/null || true
```

### ruff.toml
```
line-length = 100

[lint]
select = ["E", "F", "I"]
```
Ruff picks this up automatically (it does NOT change the "stdlib only" rule — it is a config file, not a dependency).

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
- Type hints everywhere
- Generics as built-in types (`dict[str, str]`, `list[Question]`) — do NOT import `typing.Dict`/`typing.List`; `from __future__ import annotations` makes that unnecessary.
- No single-letter variable names (`l`, `O`, `I`) — use descriptive names (`line`, `item`, `idx`). Else, there will be Ruff errors.
- All imports in one block at the top. Else, there will be Ruff errors.
- Google-style docstrings: Args/Returns/Raises/Example
- dataclasses with `default_factory`
- `with` statements for file operations
- PEP 8/257/484, 4-space indentation, no tabs
- stdlib only, no external dependencies
- Colors as module constants; only emit when `sys.stdout.isatty()` (disable color for pipes/files). Implement as a pure `style(text, code, enabled)` helper that takes an `enabled` bool — callers pass `sys.stdout.isatty()`, tests exercise both branches.
- Tools that shuffle/randomize (quiz, sampler, reorderer): expose a `--seed` flag (int, default None) and pass an explicit `random.Random(seed)` into the logic — tests and demos then use a fixed seed for reproducible output.

## Validation
1. Immediately after each Python file: `python3 -m py_compile <file>`. Fix errors right away.
2. Preflight: `python3 --version`, `python3 -m ruff --version`, `python3 -m mypy --version`, `python3 -m pytest --version`. Missing modules: skip the corresponding `validate` steps and report them as `skipped` instead of aborting.
3. Run `make validate` yourself (note: `ruff check --fix` and `ruff format` rewrite files — that's intended, recompile afterward). `validate` ends with a functional smoke run of the tool against the sample file (no stdin → EOF → exit 130), so runtime errors surface — not just import/parse errors.
4. Report the result as compact status lines, no prose paragraphs. Example:
   ```
   py_compile: ok
   ruff_check: ok
   ruff_format: ok
   mypy: ok
   pytest: 20 passed
   demo: ok
   help: ok
   ```
5. Keep the response short. Don't print finished files into the chat.

## Test Setup
- `sys.path.insert(0, str(PROJECT_ROOT))` with `PROJECT_ROOT = Path(__file__).resolve().parent`
- `monkeypatch.setattr('sys.stdin', io.StringIO(...))` for file/stdin mocking; for `input()` use `monkeypatch.setattr('builtins.input', ...)` directly instead
- `capsys` for output assertions
- `tmp_path` fixtures for file tests
- Exit codes: assert with `pytest.raises(SystemExit) as excinfo:` and check `excinfo.value.code` (1 for file/parse errors, 130 for EOF/`KeyboardInterrupt`)
- Color gating: monkeypatch `sys.stdout.isatty` (or call the `style(text, code, enabled)` helper directly) and assert the same text renders with and without ANSI codes
