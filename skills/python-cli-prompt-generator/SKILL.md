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
5. DEBUG CEILING: If validation fails more than 3 times consecutively, strip all advanced logic, fall back to the simplest working procedural implementation, and re-validate.

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
import builtins
from pathlib import Path
import pytest

PROJECT_ROOT = Path(__file__).resolve().parent
sys.path.insert(0, str(PROJECT_ROOT))

import {__TOOLNAME__}

class InputMock:
    def __init__(self, answers: list[str]) -> None:
        self._answers = list(answers)
    def __call__(self, prompt: str = "") -> str:
        return self._answers.pop(0) if self._answers else ""
```

#### Makefile
*Constraint*: Lines inside recipes MUST start with a literal TAB character. Write RAW makefile code only. NEVER wrap the template inside quotes (`"""`) or markdown blocks.
```makefile
.PHONY: validate
validate:
	python3 -m py_compile *.py && python3 -m ruff check --fix *.py && python3 -m ruff format *.py && python3 -m mypy *.py && python3 -m pytest -q --tb=short && python3 {__TOOLNAME__}.py --help && (python3 {__TOOLNAME__}.py {__TOOLNAME__}.txt </dev/null || true)
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
- Terminal Colors: Define colors as module constants. If `sys.stdout.isatty()` is False, dynamically re-assign all color constants to empty strings `""` at startup to keep print statements clean and branchless.
- Interactive Safety: To prevent infinite loops in automated test environments, any interactive loop reading `input()` MUST break and exit with `sys.exit(1)` immediately if `input()` returns an empty string `""` multiple times consecutively or if end-of-file (EOF) conditions occur during non-interactive pipes.

---

## 3. Strict Execution Workflow (Step-by-Step)

You must act as an automated loop. Move to the next step immediately without asking for permission or writing conversational prose.

### Step 1: Analyze & Create Test Data
1. Extract the target tool name from the user prompt.
2. If the tool operates on an input file format, create a minimal, valid sample file named `{__TOOLNAME__}.txt` with realistic test data.

### Step 2: Generate Scaffold & Tests (test_{__TOOLNAME__}.py)
Write the testing suite *before* finalizing the logic (TDD approach). Write at least 3 distinct tests:
1. `test_parser`: Validates CLI arguments and flags.
2. `test_happy_path`: Mocks standard inputs and validates successful core logic output via `capsys`.
   - RANDOM DEACTIVATION: You MUST patch the shuffle function directly: `monkeypatch.setattr(learn.random, "shuffle", lambda x: None)`.
   - TEST DATA ALIGNMENT: Read the true `ANSWER:` tokens from the active tool's input asset file to construct the mock answers. Never guess or hardcode static sequences blindly.
   - EOF PROTECTION: To prevent premature loop crashes (`SystemExit: 130`) during test execution, the `InputMock.__call__` method MUST raise a clean `SystemExit(0)` or return a specific terminal exit string instead of running dry or throwing unexpected exceptions when the answer array is exhausted.
3. `test_error_handling`: Validates resilience against expected edge cases (e.g., empty files, malformed inputs).

### Step 3: Execute Validation Tool
1. ACTION: You MUST now call the Agent's integrated terminal execution tool to run `make validate` right now. Do NOT output prose. Do NOT wait for user input.
2. Output the exact block format below based on the tool's execution result. Do not explain the output.

```
py_compile: [status]
ruff_check: [status]
ruff_format: [status]
mypy: [status]
pytest: [status]
```

### Step 4: Implement Core Logic & Fix Issues
1. Write the production code inside `{__TOOLNAME__}.py`.
2. For interactive or stateful loops, use immutable data copies or clean list replacements rather than direct index mutation to prevent tracking bugs.
2. MANDATORY: Trigger the terminal/task tool again to execute `make validate`.
3. If errors occur, rewrite the failing function or file completely via a single code block. Do NOT patch lines piecemeal. Do NOT add comments.
4. Loop step 4 dynamically until all validation steps report `ok` or `passed`. Max 3 loops.
