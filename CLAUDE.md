# cogni-copilot

Claude Code plugin that ports Claude Code plugins to Microsoft Copilot Studio.

## Project Structure

```
.claude-plugin/plugin.json          # Plugin manifest
agents/plugin-analyzer.md           # Subagent for analyzing source plugins
commands/port-plugin.md             # /port-plugin slash command (entry point)
skills/copilot-studio-mapping/
  SKILL.md                          # Translation rules and component mapping
  references/
    connector-templates.md          # OpenAPI 2.0 connector examples
    gui-instructions-template.md    # GUI walkthrough template
    topic-templates.md              # AdaptiveDialog YAML templates (14 node types)
    workflow-templates.md           # Power Automate flow JSON templates
```

## Conventions

- YAML files use 2-space indentation
- Frontmatter on all markdown components (agents, commands, skills)
- Custom connectors MUST use OpenAPI 2.0 (not 3.0) — Copilot Studio constraint
- Topic node IDs should be descriptive: `sendActivity_greeting`, `question_getInput`
- Agent names max 42 characters, no angle brackets
