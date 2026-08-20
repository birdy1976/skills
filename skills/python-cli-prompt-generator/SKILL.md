---
name: python-cli-prompt-generator
description: "Zero-Prose Automated TDD Python CLI Builder for local OSS models."
---

# Skill: Python-CLI-Prompt-Generator

## CRITICAL RULES FOR MODELS (NEVER VIOLATE)
1. NO PROSE. Zero conversational filler. Respond ONLY in code blocks or the requested verification status lines. Don't print finished files into the chat: add them to the working directory.
2. NO COMMENTS. Absolutely NO `# comments` and NO `"""docstrings"""` in ANY generated file. Code must be self-explanatory. Leaving a comment causes a linting/syntax failure.
3. PREVENT BROKEN CODE: Never patch existing functions piecemeal. If a file fails validation, rewrite the entire file or the entire function from scratch to prevent indentation/control-flow errors.
4. TEST HIERARCHY: If `make validate` fails, verify whether the test mock matches the user's requirement. Do NOT alter the core CLI architecture to satisfy a broken test; fix the test or the parser first.

---

# INSTRUCTION

## 1. Project Architecture & Setup
Extract the tool name from the prompt and replace `{__TOOLNAME__}` globally. Generate these exact files in the working directory:

```
./
├── {__TOOLNAME__}.py
├── test_{__TOOLNAME__}.py
├── {__TOOLNAME__}.txt (Optional: only if the tool requires a sample input file)
├── Makefile
└── README.md
```

### File Templates

#### {__TOOLNAME__}.py
```python
from __future__ import annotations

import argparse
import sys

def main(argv: list[str] | None = None) -> None:
    parser = argparse.ArgumentParser(description="{__DESCRIPTION__}")

    args = parser.parse_args(argv)

    try:
        pass
    except (KeyboardInterrupt, EOFError):
        print()
        sys.exit(130)

if __name__ == "__main__":
    main()
```
*Constraint*: Every loop containing `input()` or interactive I/O MUST execute inside the `try/except` block from the start.

#### test_{__TOOLNAME__}.py
```python
from __future__ import annotations

import io
import sys
from pathlib import Path
import pytest

PROJECT_ROOT = Path(__file__).resolve().parent
sys.path.insert(0, str(PROJECT_ROOT))

import {__TOOLNAME__}
```

#### Makefile
*Constraint*: Lines inside recipes MUST start with a literal TAB character.
*Optional smoke test*: Add `&& (python3 {__TOOLNAME__}.py {__TOOLNAME__}.txt </dev/null || true)` to the `validate` recipe, if there is a `{__TOOLNAME__}.txt`.
```makefile
.PHONY: help
help: ## Show this help message
	@echo "Available commands:"
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "  \033[36m%-15s\033[0m %s\n", $$1, $$2}'

.DEFAULT_GOAL := help

.PHONY: validate
validate: ## Validate codebase
	python3 -m py_compile *.py && python3 -m ruff check --fix *.py && python3 -m ruff format *.py && python3 -m mypy *.py && python3 -m pytest -q --tb=short && python3 {__TOOLNAME__}.py --help
```

### README.md
```markdown
# {__TOOLNAME__}

{__DESCRIPTION__}

## Usage including --help

```
python3 {__TOOLNAME__}.py <arguments>
```

## Development

Development only: `make validate`.
```

---

## 2. Coding Rules
- Imports: Single block at the top. No inline imports.
- Variable Names: No single-letter names except inside list comprehensions. Use descriptive names like `line`, `item`, `idx`.
- Typings: Use `list[Type]`, `dict[str, str]`. Do NOT import from the old `typing` module; use built-in generics.
- Dependencies: Python Standard Library ONLY. No external third-party packages.
- Terminal Colors: Only emit ANSI color constants if `sys.stdout.isatty()` is True (disable for pipes/files).

---

## 3. Strict Execution Workflow (Step-by-Step)

You must act as an automated loop. Move to the next step immediately without asking for permission or writing conversational prose.

### Step 1: Analyze & Create Test Data
1. Extract the target tool name from the user prompt.
2. If the tool operates on an input file format, create a minimal, valid sample file named `{__TOOLNAME__}.txt` with realistic test data.

### Step 2: Generate Scaffold & Tests (test_{__TOOLNAME__}.py)
Write the testing suite *before* finalizing the logic (TDD approach). Write at least 3 distinct tests:
1. `test_parser`: Validates CLI arguments and flags.
2. `test_happy_path`: Mocks standard inputs/files and validates successful core logic output via `capsys`.
3. `test_error_handling`: Validates resilience against expected edge cases (e.g., empty files, malformed inputs).

### Step 3: Implement Core Logic & Fix Issues
1. Immediately run `make validate` and fix the issues.
2. If there are linting/typing errors, rewrite the failing structural block or file completely. Do NOT patch lines piecemeal.
3. Loop dynamically until all validation steps report `ok` or `passed` and output the exact block format below:

```
py_compile: [status]
ruff_check: [status]
ruff_format: [status]
mypy: [status]
pytest: [status]
```
