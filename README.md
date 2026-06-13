# Skills
Public repository for agent skills 

## Verfügbare Skills

### Python CLI Prompt Generator

**Beschreibung:**
Der `python-cli-prompt-generator` Skill generiert einen vollständigen, agenten-optimierten Arbeitsauftrag (`PROMPT.md`) aus einer kurzen Tool-Beschreibung. Dieser Prompt enthält eine definierte Projektstruktur, Python-Expert-Regeln, sowie detaillierte Ausführungs- und Validierungsregeln für einen Coding-Agenten. Die Generierung der Pytest-Tests kann über Optionen gesteuert werden.

**Installation:**
Um diesen Skill zu installieren, führen Sie den folgenden Befehl in Ihrem Terminal aus:

```bash
npx skills add https://github.com/birdy1976/skills --skill python-cli-prompt-generator
```

**Verwendung:**
Es stehen folgende Optionen für die Generierung von Pytest-Tests zur Verfügung:

1.  **Standard-Aufruf (minimale Pytests):**
    `/python-cli-prompt-generator`
    *Dieser Aufruf erzeugt einen PROMPT.md mit minimalen, aber aussagekräftigen Pytests (Standardverhalten).*

2.  **Ohne Pytests:**
    `/python-cli-prompt-generator pytest-none`
    *Dieser Aufruf erzeugt einen PROMPT.md, der keine Pytest-Sektion und keine zugehörige Validierung enthält.*

3.  **Minimale Pytests:**
    `/python-cli-prompt-generator pytest-mini`
    *Dieser Aufruf erzeugt einen PROMPT.md mit minimalen Pytests.*

4.  **Umfassende Pytests (aktuelle Implementation):**
    `/python-cli-prompt-generator pytest-full`
    *Dieser Aufruf erzeugt einen PROMPT.md mit einer umfassenden Pytest-Implementierung, ähnlich dem vorherigen Standard.*

Nach dem Aufruf des Skills wird die KI den generierten `PROMPT.md` im aktuellen Arbeitsverzeichnis erstellen und den Inhalt zusätzlich im Chat ausgeben.
