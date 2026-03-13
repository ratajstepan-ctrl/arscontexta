# platforms/ -- Distribution View

This directory maps the distribution layout for each supported agent platform. It does not contain the generation logic itself -- that lives in `generators/`. What `platforms/` provides is a reference view of what each platform produces and how the shared components relate to platform-specific ones.

## Relationship to generators/

The `generators/` directory is the working structure:

- `generators/claude-md.md` -- CLAUDE.md / SYSTEM_PROMPT.md generation template
- `generators/features/` -- 14 composable feature blocks

`platforms/` organizes these same components from a distribution perspective: what does a Claude Code user get? What does a Hermes-agent user get? What is shared across platforms?

## Structure

```
platforms/
├── shared/
│   ├── features/     --> generators/features/ (14 canonical feature blocks)
│   └── templates/    --> reference/templates/ (10 note type templates)
├── claude-code/
│   ├── generator.md  --> generators/claude-md.md (CLAUDE.md generation)
│   └── hooks/        Hook templates for Claude Code platform
├── hermes-agent/
│   ├── README.md     Platform overview for Hermes3 via Ollama
│   └── generator.md  SYSTEM_PROMPT.md generation guide
└── README.md         This file
```

## How platforms/ is used

During plugin packaging (Section 18 of the PRD), the build process references `platforms/` to assemble the distribution:

- **Claude Code plugin** reads `platforms/shared/` and `platforms/claude-code/` to bundle feature blocks, templates, generation logic, and hook templates alongside the `skills/`, `reference/`, and `thinking/` directories.
- **Hermes-agent** reads `platforms/shared/` and `platforms/hermes-agent/` to produce `SYSTEM_PROMPT.md`, plain-markdown skill prompts in `prompts/`, and conventional vault structure.

The `generators/` directory remains the canonical source. Files here are not duplicated -- README files document the relationship and provide platform-specific context that the generator files themselves don't carry.

## What each platform produces

| Output | Claude Code | Hermes-agent |
|--------|-------------|-------------|
| Context file | CLAUDE.md | SYSTEM_PROMPT.md |
| Hooks | .claude/hooks/ (bash scripts) | None (manual instructions in SYSTEM_PROMPT.md) |
| Settings | .claude/settings.json | ops/config.yaml |
| Skills | .claude/skills/ (SKILL.md format) | prompts/ (plain markdown) |
| Identity | Embedded in CLAUDE.md | Embedded in SYSTEM_PROMPT.md + self/ |
| Memory bootstrap | Embedded in CLAUDE.md | self/ directory loaded manually |
| Platform marker | .claude/ directory | .hermes-agent/ directory |
