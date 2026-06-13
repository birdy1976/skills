# Skills — Agent Skills Repository

Public collection of OpenCode agent skills at `https://github.com/birdy1976/skills`.

## Structure

- Each skill is a directory under `skills/` with a single `SKILL.md` file.
- `SKILL.md` must have YAML frontmatter with `name` (kebab-case) and `description`.

```
skills/<skill-name>/
└── SKILL.md
```

## Conventions

- `SKILL.md` YAML header keys: `name`, `description` only.
- Skill content is pure markdown (any language).

## Adding a new skill

1. Create `skills/<your-skill-name>/SKILL.md`
2. Add YAML frontmatter:
   ```yaml
   ---
   name: your-skill-name
   description: "Short description of what the skill does."
   ---
   ```
3. Add new skill to the README.md.
4. Users install via: `npx skills add https://github.com/birdy1976/skills --skill <your-skill-name>`
5. Commit and push to add it to the public collection.

## Available skills

### python-cli-prompt-generator

Generates a `PROMPT.md` with a complete coding-agent work order from a short tool description.

**Install:** `npx skills add https://github.com/birdy1976/skills --skill python-cli-prompt-generator`

**Usage:** `/python-cli-prompt-generator [variant]`

| Variant | Behaviour |
|---|---|
| *(no arg)* | Minimal pytest section (default) |
| `pytest-mini` | Minimal pytest section |
| `pytest-none` | No pytest section or validation |
| `pytest-full` | Full pytest suite with validation |

The generated `PROMPT.md` is created in the current working directory.

## Tooling

None. No package.json, CI, tests, linters, or formatters.
