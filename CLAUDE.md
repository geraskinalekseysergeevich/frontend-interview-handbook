# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Pure Markdown knowledge base for frontend developer interview preparation. No build system, no tests, no package manager.

## Structure

```
profile-react.md          # Skills matrix: rows = skills, cols = seniority levels (Junior → Senior)
skills/
  frontend/               # Browser/React-specific skills
  shared/                 # Cross-stack skills (JS, TS, async, VCS, monitoring, etc.)
  js/                     # JS-specific skills (testing)
```

## Skill file conventions

Each file in `skills/` follows this structure:

```markdown
# Topic — Subtopic

## Уровни навыка
### Уровень 1
* ...bullet points of competencies...

### Уровень 2
* ...

## Как прокачать
### Уровень 1
* links
```

Levels are integers 1–4. Higher = deeper expertise. Level descriptions must be additive — each level builds on the previous.

## Skills matrix (`profile-react.md`)

Markdown table linking skill files to required levels per seniority grade:

| Skill (linked) | Junior | Junior+ | Middle | Middle+ | Senior | Comments |
|---|---|---|---|---|---|---|

- Cell value is the expected level integer (or `0` = not required).
- Rows are grouped by category (Language, Browser APIs, Application development, Infra, Other).
- Links use relative paths: `[Name](./skills/path/file.md)`.

## Editing rules

- Adding a new skill: create the file in the appropriate `skills/` subdirectory, then add a row to the matrix in `profile-react.md`.
- Skill levels must be monotonically non-decreasing across Junior → Senior columns.
- Language of content: Russian. Technical terms stay in their original form.
